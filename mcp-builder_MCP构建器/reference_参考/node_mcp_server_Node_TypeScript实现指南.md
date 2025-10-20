# Node/TypeScript MCP 服务器实现指南

## 概览

本文提供基于 MCP TypeScript SDK 开发 MCP 服务器的最佳实践与示例，涵盖项目结构、服务器初始化、工具注册、使用 Zod 的输入校验、错误处理以及完整示例。

---

## 快速参考

### 核心导入
```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import axios, { AxiosError } from "axios";
```

### 服务器初始化
```typescript
const server = new McpServer({
  name: "service-mcp-server",
  version: "1.0.0"
});
```

### 工具注册模式
```typescript
server.registerTool("tool_name", {...config}, async (params) => {
  // 实现
});
```

---

## MCP TypeScript SDK

官方 SDK 提供：
- `McpServer`：用于初始化服务器
- `registerTool`：注册工具
- 与 Zod 的运行时输入校验集成
- 类型安全的工具处理函数

更多详情请参阅 SDK 文档。

## 服务器命名规范

Node/TypeScript MCP 服务器需遵循 `{service}-mcp-server` 格式（小写连字符），例如：
- `github-mcp-server`
- `jira-mcp-server`

命名应：
- 通用且不局限具体功能
- 能体现整合的服务 / API
- 便于从任务描述推断
- 不包含版本号或日期

## 项目结构

推荐结构：
```
{service}-mcp-server/
├── package.json
├── tsconfig.json
├── README.md
├── src/
│   ├── index.ts          # 入口，初始化 McpServer
│   ├── types.ts          # 类型定义
│   ├── tools/            # 工具实现（按领域拆分文件）
│   ├── services/         # API 客户端与通用工具
│   ├── schemas/          # Zod 校验
│   └── constants.ts      # 常量（API_URL、CHARACTER_LIMIT 等）
└── dist/                 # 构建输出（入口：dist/index.js）
```

## 工具实现

### 工具命名

使用 snake_case（如 `search_users`），并确保加上服务前缀避免冲突：
- `slack_send_message` 而非 `send_message`
- `github_create_issue` 而非 `create_issue`

### 工具结构

使用 `registerTool` 注册，要求：
- 使用 Zod 进行运行时输入校验
- 必须显式提供 `title`、`description`、`inputSchema`、`annotations`
- `inputSchema` 需传入 Zod 对象（不是 JSON Schema）
- 明确声明参数与返回值类型

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({
  name: "example-mcp",
  version: "1.0.0"
});

const UserSearchInputSchema = z.object({
  query: z.string()
    .min(2, "Query must be at least 2 characters")
    .max(200, "Query must not exceed 200 characters")
    .describe("Search string to match against names/emails"),
  limit: z.number()
    .int()
    .min(1)
    .max(100)
    .default(20)
    .describe("Maximum results to return"),
  offset: z.number()
    .int()
    .min(0)
    .default(0)
    .describe("Number of results to skip for pagination"),
  response_format: z.nativeEnum(ResponseFormat)
    .default(ResponseFormat.MARKDOWN)
    .describe("Output format: 'markdown' for human-readable or 'json' for machine-readable")
}).strict();

type UserSearchInput = z.infer<typeof UserSearchInputSchema>;

server.registerTool(
  "example_search_users",
  {
    title: "Search Example Users",
    description: `...`,
    inputSchema: UserSearchInputSchema,
    annotations: {
      readOnlyHint: true,
      destructiveHint: false,
      idempotentHint: true,
      openWorldHint: true
    }
  },
  async (params: UserSearchInput) => {
    // 实现细节...
  }
);
```

## 使用 Zod 进行输入校验

```typescript
const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
}).strict();

enum ResponseFormat {
  MARKDOWN = "markdown",
  JSON = "json"
}

const PaginationSchema = z.object({
  limit: z.number().int().min(1).max(100).default(20),
  offset: z.number().int().min(0).default(0)
});
```

## 响应格式选项

支持 Markdown（默认）与 JSON：

- **Markdown**：使用标题、列表，时间转为可读格式，显示名 + ID，省略冗余元数据
- **JSON**：返回完整结构化数据，字段命名一致

## 分页实现

```typescript
const ListSchema = z.object({
  limit: z.number().int().min(1).max(100).default(20),
  offset: z.number().int().min(0).default(0)
});

async function listItems(params: z.infer<typeof ListSchema>) {
  const data = await apiRequest(params.limit, params.offset);
  const response = {
    total: data.total,
    count: data.items.length,
    offset: params.offset,
    items: data.items,
    has_more: data.total > params.offset + data.items.length,
    next_offset: data.total > params.offset + data.items.length
      ? params.offset + data.items.length
      : undefined
  };
  return JSON.stringify(response, null, 2);
}
```

## 字数上限与截断

```typescript
export const CHARACTER_LIMIT = 25000;

if (result.length > CHARACTER_LIMIT) {
  const truncatedData = data.slice(0, Math.max(1, data.length / 2));
  response.data = truncatedData;
  response.truncated = true;
  response.truncation_message =
    `Response truncated from ${data.length} to ${truncatedData.length} items. ` +
    `Use 'offset' parameter or add filters to see more results.`;
  result = JSON.stringify(response, null, 2);
}
```

## 错误处理

```typescript
function handleApiError(error: unknown): string {
  if (error instanceof AxiosError) {
    if (error.response) {
      switch (error.response.status) {
        case 404:
          return "Error: Resource not found. Please check the ID is correct.";
        case 403:
          return "Error: Permission denied. You don't have access to this resource.";
        case 429:
          return "Error: Rate limit exceeded. Please wait before making more requests.";
        default:
          return `Error: API request failed with status ${error.response.status}`;
      }
    } else if (error.code === "ECONNABORTED") {
      return "Error: Request timed out. Please try again.";
    }
  }
  return `Error: Unexpected error occurred: ${error instanceof Error ? error.message : String(error)}`;
}
```

## 复用工具函数

```typescript
async function makeApiRequest<T>(
  endpoint: string,
  method: "GET" | "POST" | "PUT" | "DELETE" = "GET",
  data?: any,
  params?: any
): Promise<T> {
  const response = await axios({
    method,
    url: `${API_BASE_URL}/${endpoint}`,
    data,
    params,
    timeout: 30000,
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    }
  });
  return response.data;
}
```

## Async/Await 最佳实践

```typescript
// 推荐
async function fetchData(resourceId: string): Promise<ResourceData> {
  const response = await axios.get(`${API_URL}/resource/${resourceId}`);
  return response.data;
}

// 避免 Promise 链式调用
```

## TypeScript 最佳实践

1. 在 `tsconfig.json` 中启用 strict 模式
2. 定义明确的接口
3. 不使用 `any`（可用 `unknown` 或具体类型）
4. 使用 Zod 执行运行时校验
5. 为复杂类型编写类型守卫
6. 使用 try-catch 并进行类型判断
7. 善用可选链与空值合并

```typescript
interface UserResponse {
  id: string;
  name: string;
  email: string;
  team?: string;
  active: boolean;
}

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  team: z.string().optional(),
  active: z.boolean()
});

type User = z.infer<typeof UserSchema>;

async function getUser(id: string): Promise<User> {
  const data = await apiCall(`/users/${id}`);
  return UserSchema.parse(data);
}
```

## package.json & tsconfig.json 示例

```json
{
  "name": "{service}-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/index.js",
  "scripts": {
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "clean": "rm -rf dist"
  },
  "engines": {
    "node": ">=18"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.6.1",
    "axios": "^1.7.9",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@types/node": "^22.10.0",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2"
  }
}
```

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 完整示例

（示例与原文一致，略）

---

## 进阶功能

### 资源注册

```typescript
server.registerResource(
  {
    uri: "file://documents/{name}",
    name: "Document Resource",
    description: "Access documents by name",
    mimeType: "text/plain"
  },
  async (uri: string) => {
    const match = uri.match(/^file:\/\/documents\/(.+)$/);
    if (!match) throw new Error("Invalid URI format");
    const documentName = match[1];
    const content = await loadDocument(documentName);
    return {
      contents: [{
        uri,
        mimeType: "text/plain",
        text: content
      }]
    };
  }
);

server.registerResourceList(async () => {
  const documents = await getAvailableDocuments();
  return {
    resources: documents.map(doc => ({
      uri: `file://documents/${doc.name}`,
      name: doc.name,
      mimeType: "text/plain",
      description: doc.description
    }))
  };
});
``】

**资源 vs 工具**：
- 资源：URI 模板、数据访问、相对静态
- 工具：复杂逻辑、校验、可能有副作用

### 传输方式

```typescript
const stdioTransport = new StdioServerTransport();
await server.connect(stdioTransport);

const sseTransport = new SSEServerTransport("/message", response);
await server.connect(sseTransport);
```

- **Stdio**：命令行、子进程、本地开发
- **HTTP**：Web 服务、远程访问、多客户端
- **SSE**：实时更新、推送、监控看板

### 通知

```typescript
server.notification({ method: "notifications/tools/list_changed" });
server.notification({ method: "notifications/resources/list_changed" });
```

仅在能力确实发生变化时发送通知。

---

## 代码最佳实践

### 复用与组合

1. 提取公共逻辑，避免重复
2. 封装共享 API 客户端
3. 集中处理错误
4. 把业务逻辑拆成可组合函数
5. 提取公共 Markdown / JSON 格式化逻辑

### 避免重复

- **禁止**复制粘贴类似代码
- 重复逻辑应抽取函数
- 常见操作（分页、过滤、字段选择、格式化）应共用
- 认证 / 授权逻辑须集中管理

---

## 构建与运行

```bash
npm run build
npm start
npm run dev
```

务必确保 `npm run build` 成功。

---

## 质量检查清单

（建议保留原文中的检查项，按分类列出。）

- **设计层面**：工具应完成完整工作流、名称反映任务划分、响应格式适配上下文等
- **实现层面**：重点工具已实现，注解设置正确，Zod 校验使用 `.strict()` 等
- **TypeScript 质量**：严格模式、无 `any`、显式返回类型、错误类型守卫等
- **进阶特性**：适当注册资源、使用正确传输方式、必要时发通知
- **项目配置**：`package.json`、`tsconfig.json` 设置正确，构建产物正常
- **代码质量**：实现分页、字符限制、合理过滤、网络操作有超时处理、公用逻辑抽取
- **测试构建**：执行 `npm run build`、`node dist/index.js`、验证示例调用

---
