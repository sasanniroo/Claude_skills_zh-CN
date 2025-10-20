# Python MCP 服务器实现指南

## 概览

本文总结使用 MCP Python SDK（FastMCP）构建 MCP 服务器时的最佳实践与示例，包括服务器初始化、工具注册、借助 Pydantic 进行输入校验、错误处理以及完整示例。

---

## 快速参考

### 核心导入
```python
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field, field_validator, ConfigDict
from typing import Optional, List, Dict, Any
from enum import Enum
import httpx
```

### 服务器初始化
```python
mcp = FastMCP("service_mcp")
```

### 工具注册模式
```python
@mcp.tool(name="tool_name", annotations={...})
async def tool_function(params: InputModel) -> str:
    # 实现
    ...
```

---

## MCP Python SDK 与 FastMCP

官方 MCP Python SDK 提供 FastMCP 高层框架，具备：
- 自动从函数签名与文档字符串生成描述、输入模式
- 与 Pydantic 集成，实现输入校验
- 使用 `@mcp.tool` 装饰器注册工具

完整 SDK 文档可通过 WebFetch 获取：  
`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`

## 服务器命名规范

Python MCP 服务器命名需遵循 `{service}_mcp`（小写、下划线），例如 `github_mcp`。名称应：
- 通用，不限定某个具体功能
- 能体现集成的服务或 API
- 易于从任务描述中推断
- 不包含版本号或日期

## 工具实现

### 工具命名

工具名使用 snake_case，且需包含服务前缀以避免冲突，例如：
- `slack_send_message`
- `github_create_issue`

### 使用 FastMCP 定义工具

借助 `@mcp.tool` 装饰器与 Pydantic 模型：

```python
from pydantic import BaseModel, Field, ConfigDict
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("example_mcp")

class ServiceToolInput(BaseModel):
    '''工具输入模型。'''
    model_config = ConfigDict(
        str_strip_whitespace=True,
        validate_assignment=True,
        extra='forbid'
    )

    param1: str = Field(..., description="示例：'user123' 或 'project-abc'", min_length=1, max_length=100)
    param2: Optional[int] = Field(default=None, description="可选整数，含约束", ge=0, le=1000)
    tags: Optional[List[str]] = Field(default_factory=list, description="标签列表", max_items=10)

@mcp.tool(
    name="service_tool_name",
    annotations={
        "title": "可读的工具名称",
        "readOnlyHint": True,
        "destructiveHint": False,
        "idempotentHint": True,
        "openWorldHint": False
    }
)
async def service_tool_name(params: ServiceToolInput) -> str:
    '''工具描述将作为 description。'''
    ...
```

## Pydantic v2 要点

- 使用 `model_config`（替代旧版 `Config`）
- 使用 `field_validator`（替代 `validator`）
- 使用 `model_dump()`（替代 `dict()`）
- 校验器必须加 `@classmethod`
- 所有字段必须显式标注类型

```python
class CreateUserInput(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, validate_assignment=True)

    name: str = Field(..., min_length=1, max_length=100, description="姓名")
    email: str = Field(..., pattern=r'^[\w\.-]+@[\w\.-]+\.\w+$', description="邮箱")
    age: int = Field(..., ge=0, le=150, description="年龄")

    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if not v.strip():
            raise ValueError("Email cannot be empty")
        return v.lower()
```

## 响应格式选项

支持 Markdown（默认）与 JSON：

```python
class ResponseFormat(str, Enum):
    MARKDOWN = "markdown"
    JSON = "json"

class UserSearchInput(BaseModel):
    query: str = Field(..., description="查询条件")
    response_format: ResponseFormat = Field(
        default=ResponseFormat.MARKDOWN,
        description="返回格式：'markdown' 或 'json'"
    )
```

- **Markdown**：使用标题、列表；时间转为可读格式；显示名与 ID 并列；省略冗余元数据；按逻辑分组
- **JSON**：返回完整结构化数据，字段统一

## 分页实现

```python
class ListInput(BaseModel):
    limit: Optional[int] = Field(default=20, ge=1, le=100, description="最大返回数量")
    offset: Optional[int] = Field(default=0, ge=0, description="跳过结果数")

async def list_items(params: ListInput) -> str:
    data = await api_request(limit=params.limit, offset=params.offset)
    response = {
        "total": data["total"],
        "count": len(data["items"]),
        "offset": params.offset,
        "items": data["items"],
        "has_more": data["total"] > params.offset + len(data["items"]),
        "next_offset": params.offset + len(data["items"]) if data["total"] > params.offset + len(data["items"]) else None
    }
    return json.dumps(response, indent=2)
```

## 字数限制与截断

```python
CHARACTER_LIMIT = 25000

if len(result) > CHARACTER_LIMIT:
    truncated_data = data[:max(1, len(data) // 2)]
    response["data"] = truncated_data
    response["truncated"] = True
    response["truncation_message"] = (
        f"Response truncated from {len(data)} to {len(truncated_data)} items. "
        f"Use 'offset' parameter or add filters to see more results."
    )
    result = json.dumps(response, indent=2)
```

## 错误处理

```python
def _handle_api_error(e: Exception) -> str:
    if isinstance(e, httpx.HTTPStatusError):
        if e.response.status_code == 404:
            return "Error: Resource not found. Please check the ID is correct."
        elif e.response.status_code == 403:
            return "Error: Permission denied."
        elif e.response.status_code == 429:
            return "Error: Rate limit exceeded."
        return f"Error: API request failed with status {e.response.status_code}"
    elif isinstance(e, httpx.TimeoutException):
        return "Error: Request timed out. Please try again."
    return f"Error: Unexpected error occurred: {type(e).__name__}"
```

## 复用工具函数

```python
async def _make_api_request(endpoint: str, method: str = "GET", **kwargs) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.request(
            method,
            f"{API_BASE_URL}/{endpoint}",
            timeout=30.0,
            **kwargs
        )
        response.raise_for_status()
        return response.json()
```

## Async/Await 最佳实践

```python
# 推荐
async def fetch_data(resource_id: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"{API_URL}/resource/{resource_id}")
        response.raise_for_status()
        return response.json()
```

## 类型注解

在代码中全面使用类型提示：

```python
from typing import Optional, List, Dict, Any

async def get_user(user_id: str) -> Dict[str, Any]:
    data = await fetch_user(user_id)
    return {"id": data["id"], "name": data["name"]}
```

## 工具文档字符串

每个工具都需提供详尽 docstring，明确输入输出类型与示例。详见原文示例。

## 完整示例

文档中提供了完整的 `example_mcp` 服务器样例，实现了用户查询等功能，并示范了常量、枚举、Pydantic 模型、错误处理与 `mcp.run()` 的使用。

---

## FastMCP 进阶特性

### 上下文注入

工具可通过 `Context` 获取日志、进度、资源访问等能力：

```python
from mcp.server.fastmcp import Context

@mcp.tool()
async def advanced_search(query: str, ctx: Context) -> str:
    await ctx.report_progress(0.25, "Starting search...")
    await ctx.log_info("Processing query", {"query": query})
    ...

@mcp.tool()
async def interactive_tool(resource_id: str, ctx: Context) -> str:
    api_key = await ctx.elicit("Please provide your API key:", input_type="password")
    ...
```

### 资源注册

使用 `@mcp.resource` 暴露静态或模板化数据：

```python
@mcp.resource("file://documents/{name}")
async def get_document(name: str) -> str:
    with open(f"./docs/{name}", "r") as f:
        return f.read()
```

### 结构化返回

工具可返回 `TypedDict`、Pydantic 模型、数据类等，FastMCP 会自动序列化：

```python
class UserData(TypedDict):
    id: str
    name: str
    email: str

@mcp.tool()
async def get_user_typed(user_id: str) -> UserData:
    return {"id": user_id, "name": "John Doe", "email": "john@example.com"}
```

### 生命周期管理

通过 `lifespan` 管理长连接或缓存：

```python
@asynccontextmanager
async def app_lifespan():
    db = await connect_to_database()
    yield {"db": db}
    await db.close()

mcp = FastMCP("example_mcp", lifespan=app_lifespan)
```

### 传输方式

FastMCP 支持多种 Transport：

```python
if __name__ == "__main__":
    mcp.run()                        # 默认 stdio
    # mcp.run(transport="streamable_http", port=8000)
    # mcp.run(transport="sse", port=8000)
```

---

## 代码最佳实践

### 复用与组合
- 提取公共逻辑，避免复制粘贴
- 封装共享 API 客户端、错误处理、格式化逻辑
- 聚合业务逻辑为可组合函数

### Python 特有建议
1. 全面使用类型注解
2. 输入校验交给 Pydantic，避免手写
3. 合理组织导入（标准库、第三方、本地）
4. 错误处理使用具体异常类型（如 `httpx.HTTPStatusError`）
5. 对需要清理的资源使用 `async with`
6. 常量使用模块级大写形式

---

## 质量检查清单

请根据原文提供的清单逐项确认，包括：
- **设计层面**：工具是否解决完整工作流、命名是否自然、响应是否节省上下文、错误是否有指导性等
- **实现层面**：关键工具是否完成、命名注解是否准确、输入校验是否完善、docstring 是否详尽、返回结构是否一致等
- **进阶特性**：是否适当使用 Context、资源、生命周期、结构化输出、不同传输方式等
- **代码质量**：分页、字符截断、过滤选项、统一错误处理、常量与异步模式是否正确
- **测试**：服务器可启动；示例调用正常；错误场景处理得当

完成上述检查后，再提交或分发你的 Python MCP 服务器。
