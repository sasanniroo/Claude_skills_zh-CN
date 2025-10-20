# MCP 服务器开发最佳实践与指南

## 概览

本文汇总构建 Model Context Protocol（MCP）服务器时的关键最佳实践与规范，包括命名、工具设计、响应格式、分页、错误处理、安全与合规要求等。

---

## 快速参考

### 服务器命名
- **Python**：`{service}_mcp`（如 `slack_mcp`）
- **Node/TypeScript**：`{service}-mcp-server`（如 `slack-mcp-server`）

### 工具命名
- 使用带服务前缀的 snake_case
- 格式：`{service}_{action}_{resource}`
- 例：`slack_send_message`、`github_create_issue`

### 响应格式
- 同时支持 JSON 与 Markdown
- JSON 便于程序处理；Markdown 便于阅读

### 分页
- 必须遵守 `limit` 参数
- 返回 `has_more`、`next_offset`、`total_count`
- 默认 20-50 条

### 字符数限制
- 设置 `CHARACTER_LIMIT` 常量（通常 25,000）
- 优雅截断并附说明
- 告知如何筛选缩小范围

---

## 目录
1. 服务器命名约定
2. 工具命名与设计
3. 响应格式规范
4. 分页最佳实践
5. 字符限制与截断
6. 工具开发最佳实践
7. 传输层最佳实践
8. 测试要求
9. OAuth 与安全最佳实践
10. 资源管理最佳实践
11. Prompt 管理最佳实践
12. 错误处理标准
13. 文档编写要求
14. 合规与监控
15. 总结
16. 工具（Tools）概述与最佳实践

---

## 1. 服务器命名约定

遵循以下标准化命名：

**Python**：`{service}_mcp`（小写、下划线）
- 例：`slack_mcp`、`github_mcp`

**Node/TypeScript**：`{service}-mcp-server`（小写、连字符）
- 例：`slack-mcp-server`、`jira-mcp-server`

命名要求：
- 通用且不依赖具体功能
- 能体现集成的服务 / API
- 易于从任务描述推断
- 不使用版本号或日期

---

## 2. 工具命名与设计

### 工具命名原则

1. 使用 snake_case：`search_users`、`create_project`
2. 必须包含服务前缀：`slack_send_message` 而不是 `send_message`
3. 以动词开头（get、list、search、create…）
4. 具体明确，避免与其他服务器冲突
5. 在同一服务器内保持一致风格

### 工具设计指南

- 描述需准确、无歧义
- 功能必须与描述匹配
- 避免与其他 MCP 服务器混淆
- 添加工具注解（readOnlyHint、destructiveHint、idempotentHint、openWorldHint）
- 工具逻辑聚焦、原子化

---

## 3. 响应格式规范

所有返回数据的工具应支持多种格式：

### JSON（`response_format="json"`）
- 面向机器处理，字段结构完整
- 保持字段命名与类型一致
- 适合后续计算与自动化处理

### Markdown（`response_format="markdown"`，常用默认）
- 面向人类阅读
- 使用标题、列表等提升可读性
- 将时间戳转为可读格式（如 “2024-01-15 10:30:00 UTC”）
- 显示名与 ID 结合（如 “@john.doe (U123456)”）
- 省略冗余元数据
- 按逻辑分组信息

---

## 4. 分页最佳实践

用于列出资源的工具须：
- **遵循 `limit` 参数**：有上限就不要一次取全部
- **实现分页**：基于 `offset` 或游标
- **返回分页信息**：含 `has_more`、`next_offset`/`next_cursor`、`total_count`
- **不可一次加载全部结果**（尤其大数据量）
- **合理默认值**：通常 20-50
- **响应中明确分页信息**，便于 LLM 继续请求

示例结构：
```json
{
  "total": 150,
  "count": 20,
  "offset": 0,
  "items": [...],
  "has_more": true,
  "next_offset": 20
}
```

---

## 5. 字符限制与截断

避免输出过长：
- 在模块中定义 `CHARACTER_LIMIT`（常用 25,000）
- 返回前检测结果长度
- 超出时优雅截断并标注
- 提供缩小结果的建议
- 在响应中说明截断范围与取更多数据的方法

示例：
```python
CHARACTER_LIMIT = 25000

if len(result) > CHARACTER_LIMIT:
    truncated_data = data[:max(1, len(data) // 2)]
    response["truncated"] = True
    response["truncation_message"] = (
        f"Response truncated from {len(data)} to {len(truncated_data)} items. "
        f"Use 'offset' parameter or add filters to see more results."
    )
```

---

## 6. 工具开发最佳实践

### 通用指南
1. 工具名描述性强、动词开头
2. 使用详细 JSON Schema 校验参数
3. 在工具说明中加入示例
4. 完善错误处理
5. 长操作提供进度提示
6. 工具操作尽量原子化
7. 文档化返回结构
8. 设置合理超时时间
9. 对资源密集型操作考虑限流
10. 记录工具调用日志

### 安全事项

**输入校验**
- 严格校验参数
- 清理文件路径、防止目录穿越
- 校验 URL 等外部 ID
- 控制参数尺寸、范围
- 防止命令注入

**访问控制**
- 实现必要的认证与授权
- 审计工具调用
- 限制请求频率
- 监测滥用行为

**错误处理**
- 不向客户端暴露内部错误
- 在服务器侧记录安全相关错误
- 妥善处理超时
- 错误后清理资源
- 校验返回值

### 工具注解
- 设置 readOnlyHint、destructiveHint 等注解
- 注解仅为提示，不具备安全保障
- 客户端不应仅凭注解做安全关键决策

---

## 7. 传输层最佳实践

MCP 支持多种传输方式：

### stdio
- **适合**：命令行工具、本地集成、子进程
- **特点**：标准输入输出通信、无需网络配置
- **适用场景**：
  - 本地开发工具
  - 桌面应用（如 Claude Desktop）
  - CLI 工具
  - 单用户单会话

### HTTP
- **适合**：Web 服务、远程访问、多客户端
- **特点**：HTTP 请求-响应，可部署为网络服务
- **适用场景**：
  - 多客户端同时访问
  - 云端部署
  - Web 应用集成
  - 需负载均衡或扩缩容

### SSE（Server-Sent Events）
- **适合**：实时更新、推送、流式数据
- **特点**：基于 HTTP 的单向服务器推送，长连接
- **适用场景**：
  - 需要实时数据更新
  - 实现推送通知
  - 日志 / 监控流式输出
  - 长耗时任务的渐进式结果

### 选择对比

| 维度         | Stdio | HTTP  | SSE   |
|--------------|-------|-------|-------|
| 部署         | 本地  | 远程  | 远程  |
| 客户端数量   | 1     | 多    | 多    |
| 通信模式     | 双向  | 请求-响应 | 服务端推送 |
| 实现复杂度   | 低    | 中    | 中-高 |
| 实时支持     | 否    | 否    | 是    |

### 通用传输指南
1. 正确管理连接生命周期
2. 完善处理通信错误
3. 设置合适超时
4. 管理连接状态
5. 断开连接时清理资源

### 传输安全
- 防范 DNS 重绑定攻击
- 实现认证机制
- 校验消息格式
- 优雅处理畸形消息

### stdio 特别注意
- 不要向 stdout 打日志，以免破坏协议；改用 stderr
- 合理管理标准流

---

## 8. 测试要求

制定完整测试策略：
- **功能测试**：验证合法/非法输入
- **集成测试**：检查与外部系统的交互
- **安全测试**：认证、输入清洗、限流等
- **性能测试**：负载、超时
- **错误处理**：错误返回、资源清理

---

## 9. OAuth 与安全最佳实践

### 身份认证与授权

**OAuth 2.1**
- 使用受信任机构签发的证书
- 在处理请求前验证 access token
- 仅接受针对本服务器的 token
- 拒绝缺失受众字段的 token
- 不要透传来自 MCP 客户端的 token

**API Key 管理**
- 保存在环境变量中，不写入代码
- 启动时验证 Key
- 认证失败提供清晰错误提示
- 通过安全渠道传递敏感凭据

### 输入校验与安全

**务必校验输入**：
- 规范化文件路径，防止目录穿越
- 校验 URL、外部 ID
- 控制参数大小范围
- 防止命令注入
- 使用 Pydantic/Zod 等进行 schema 校验

**错误处理安全性**：
- 不向客户端暴露内部错误
- 服务器端记录安全相关错误
- 错误提示要有用但不过度泄露
- 出错后及时清理资源

### 隐私与数据保护

**数据收集原则**
- 仅收集实现功能所必需的数据
- 不采集无关对话信息
- 非必要情况下不收集个人敏感信息
- 明确告知访问了哪些数据

**数据传输**
- 未经说明，不向组织外服务器发送数据
- 所有网络通信使用 HTTPS
- 验证外部服务证书

---

## 10. 资源管理最佳实践

1. 仅建议必要资源
2. 使用明确的资源名称
3. 正确处理资源边界
4. 尊重客户端对资源的控制
5. 使用模型可控的原语（工具）暴露数据

---

## 11. Prompt 管理最佳实践

- 客户端应向用户展示拟提交的 prompt
- 用户可修改或拒绝 prompt
- 展示模型生成的响应
- 用户可修改或拒绝响应
- 使用采样时关注成本

---

## 12. 错误处理标准

- 使用标准 JSON-RPC 错误码
- 工具错误应在结果对象中呈现（而非协议层）
- 错误信息具体且有帮助
- 不泄露内部实现细节
- 出错时正确清理资源

---

## 13. 文档编写要求

- 清晰记录所有工具与功能
- 每个关键功能提供至少 3 个可用示例
- 记录安全注意事项
- 标注所需权限和访问级别
- 说明速率限制与性能特征

---

## 14. 合规与监控

- 实施日志记录，用于调试与监控
- 跟踪工具使用模式
- 监控潜在滥用行为
- 对安全相关操作保留审计记录
- 做好持续合规检查准备

---

## 15. 总结

上述实践旨在帮助开发者构建安全、高效、合规的 MCP 服务器，确保其能顺利纳入 MCP 生态并为用户提供可靠体验。

---

# 工具（Tools）

> 让 LLM 通过服务器执行操作

工具是 MCP 的关键原语，使服务器可以向客户端公开可执行功能，供 LLM 调用，从而与外部系统交互、执行计算乃至在现实世界中采取行动。

<Note>
工具被设计为 **由模型控制**，即服务器向客户端暴露工具时，期望 AI 模型能自动调用（通常会有人工审批环节）。
</Note>

## 概览

工具允许服务器向客户端公开可调用的函数。核心特性：
- **可发现性**：客户端通过 `tools/list` 获取工具清单
- **可调用性**：客户端通过 `tools/call` 调用工具，服务器执行并返回结果
- **灵活性**：工具可以是简单计算，也可以封装复杂 API

工具与 [资源](/docs/concepts/resources) 相似，都有唯一标识与说明，但工具代表动态操作，可改变状态或调用外部系统。

## 工具定义结构

```typescript
{
  name: string;          // 工具唯一名称
  description?: string;  // 人类易读描述
  inputSchema: {         // JSON Schema，用于参数校验
    type: "object",
    properties: { ... }
  },
  annotations?: {        // 可选提示
    title?: string;
    readOnlyHint?: boolean;
    destructiveHint?: boolean;
    idempotentHint?: boolean;
    openWorldHint?: boolean;
  }
}
```

## 实现示例

<Tabs>
  <Tab title="TypeScript">
    ```typescript
    const server = new Server({
      name: "example-server",
      version: "1.0.0"
    }, {
      capabilities: {
        tools: {}
      }
    });

    server.setRequestHandler(ListToolsRequestSchema, async () => {
      return {
        tools: [{
          name: "calculate_sum",
          description: "Add two numbers together",
          inputSchema: {
            type: "object",
            properties: {
              a: { type: "number" },
              b: { type: "number" }
            },
            required: ["a", "b"]
          }
        }]
      };
    });

    server.setRequestHandler(CallToolRequestSchema, async (request) => {
      if (request.params.name === "calculate_sum") {
        const { a, b } = request.params.arguments;
        return {
          content: [
            {
              type: "text",
              text: String(a + b)
            }
          ]
        };
      }
      throw new Error("Tool not found");
    });
    ```
  </Tab>

  <Tab title="Python">
    ```python
    app = Server("example-server")

    @app.list_tools()
    async def list_tools() -> list[types.Tool]:
        return [
            types.Tool(
                name="calculate_sum",
                description="Add two numbers together",
                inputSchema={
                    "type": "object",
                    "properties": {
                        "a": {"type": "number"},
                        "b": {"type": "number"}
                    },
                    "required": ["a", "b"]
                }
            )
        ]

    @app.call_tool()
    async def call_tool(
        name: str,
        arguments: dict
    ) -> list[types.TextContent | types.ImageContent | types.EmbeddedResource]:
        if name == "calculate_sum":
            a = arguments["a"]
            b = arguments["b"]
            result = a + b
            return [types.TextContent(type="text", text=str(result))]
        raise ValueError(f"Tool not found: {name}")
    ```
  </Tab>
</Tabs>

## 工具类型示例

### 系统操作
```typescript
{
  name: "execute_command",
  description: "Run a shell command",
  inputSchema: {
    type: "object",
    properties: {
      command: { type: "string" },
      args: { type: "array", items: { type: "string" } }
    }
  }
}
```

### API 集成
```typescript
{
  name: "github_create_issue",
  description: "Create a GitHub issue",
  inputSchema: {
    type: "object",
    properties: {
      title: { type: "string" },
      body: { type: "string" },
      labels: { type: "array", items: { type: "string" } }
    }
  }
}
```

### 数据处理
```typescript
{
  name: "analyze_csv",
  description: "Analyze a CSV file",
  inputSchema: {
    type: "object",
    properties: {
      filepath: { type: "string" },
      operations: {
        type: "array",
        items: { enum: ["sum", "average", "count"] }
      }
    }
  }
}
```

## 工具实现最佳实践

1. 提供清晰名称与描述
2. 使用详细 JSON Schema
3. 在描述中附示例
4. 完善错误处理
5. 长操作提供进度
6. 原子化实现
7. 文档化返回结构
8. 设置超时
9. 考虑限流
10. 记录日志

### 工具名称冲突

当多个 MCP 服务器存在同名工具时，客户端可通过以下方式区分：
- `web1___search_web`、`web2___search_web`（基于用户配置的唯一名称）
- `jrwxs___search_web`、`6cq52___search_web`（随机前缀）
- `web1.example.com:search_web`（使用服务器 URI 前缀）

初始化流程返回的服务器名称不保证唯一，不适合用于区分。

## 安全注意事项

### 输入校验
- 校验参数
- 清理文件路径与命令
- 校验 URL / 外部 ID
- 控制参数范围
- 防止命令注入

### 访问控制
- 必要时实现认证
- 进行授权检查
- 审计工具调用
- 限流并监控滥用

### 错误处理
- 不向客户端暴露内部错误
- 服务器记录安全相关日志
- 妥善处理超时
- 出错后清理资源
- 校验返回值

## 工具发现与更新

- 客户端随时可列出工具
- 服务器可通过 `notifications/tools/list_changed` 通知更新
- 运行时可动态增删工具
- 更新工具定义需谨慎

## 错误处理示例

工具应在结果对象中报告错误，而非协议层。示例：

<Tabs>
  <Tab title="TypeScript">
    ```typescript
    try {
      const result = performOperation();
      return {
        content: [
          { type: "text", text: `Operation successful: ${result}` }
        ]
      };
    } catch (error) {
      return {
        isError: true,
        content: [
          { type: "text", text: `Error: ${error.message}` }
        ]
      };
    }
    ```
  </Tab>
  <Tab title="Python">
    ```python
    try:
        result = perform_operation()
        return types.CallToolResult(
            content=[types.TextContent(type="text", text=f"Operation successful: {result}")]
        )
    except Exception as error:
        return types.CallToolResult(
            isError=True,
            content=[types.TextContent(type="text", text=f"Error: {str(error)}")]
        )
    ```
  </Tab>
</Tabs>

这样 LLM 可以意识到出错并采取行动。

## 工具注解

注解为客户端提供工具行为提示，帮助优化体验，但不能作为安全依据。

### 作用
1. 向 UI 提供额外信息
2. 帮助分类与展示
3. 提示潜在副作用
4. 辅助设计直观的审批界面

### 可用注解

| 注解              | 类型    | 默认值 | 说明                                            |
|-------------------|---------|--------|-------------------------------------------------|
| `title`           | string  | -      | 人类可读标题                                    |
| `readOnlyHint`    | boolean | false  | 若为 true，表示工具不会修改环境                 |
| `destructiveHint` | boolean | true   | 若为 true，工具可能执行破坏性更新              |
| `idempotentHint`  | boolean | false  | 若为 true，同参数重复调用不会产生额外影响       |
| `openWorldHint`   | boolean | true   | 若为 true，工具可能与开放外部世界交互          |

### 示例
```typescript
// 只读搜索工具
{ ... annotations: { title: "Web Search", readOnlyHint: true, openWorldHint: true } }

// 删除文件工具
{ ... annotations: { title: "Delete File", readOnlyHint: false, destructiveHint: true, idempotentHint: true, openWorldHint: false } }

// 创建数据库记录
{ ... annotations: { title: "Create Database Record", readOnlyHint: false, destructiveHint: false, idempotentHint: false, openWorldHint: false } }
```

### 在实现中使用注解

<Tabs>
  <Tab title="TypeScript">
    ```typescript
    server.setRequestHandler(ListToolsRequestSchema, async () => ({
      tools: [{
        name: "calculate_sum",
        description: "Add two numbers together",
        inputSchema: { ... },
        annotations: {
          title: "Calculate Sum",
          readOnlyHint: true,
          openWorldHint: false
        }
      }]
    }));
    ```
  </Tab>
  <Tab title="Python">
    ```python
    from mcp.server.fastmcp import FastMCP

    mcp = FastMCP("example-server")

    @mcp.tool(
        annotations={
            "title": "Calculate Sum",
            "readOnlyHint": True,
            "openWorldHint": False
        }
    )
    async def calculate_sum(a: float, b: float) -> str:
        return str(a + b)
    ```
  </Tab>
</Tabs>

### 注解最佳实践
1. 精确标明副作用
2. 提供易懂标题
3. 正确声明幂等性
4. 指明是否面向开放环境
5. 牢记注解仅为提示

## 工具测试

完整测试应覆盖：
- 功能测试：合法/非法输入
- 集成测试：真实或模拟依赖
- 安全测试：认证、授权、输入清洗、限流
- 性能测试：负载、超时
- 错误处理：确保通过 MCP 协议报告错误并清理资源
