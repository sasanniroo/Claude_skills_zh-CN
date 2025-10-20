---
name: mcp-builder
description: 构建高质量 MCP（Model Context Protocol）服务器的指南，帮助 LLM 通过精心设计的工具与外部服务交互。适用于在 Python（FastMCP）或 Node/TypeScript（MCP SDK）中构建整合外部 API 或服务的 MCP 服务器。
license: 完整条款见 LICENSE.txt
---

# MCP 服务器开发指南

## 概览

当你要创建高质量的 MCP 服务器，使 LLM 能够高效地与外部服务交互时，请使用本技能。MCP 服务器通过提供工具，让 LLM 可以访问外部服务或 API。衡量 MCP 服务器质量的标准，是它让 LLM 借助这些工具完成真实世界任务的能力。

---

# 开发流程

## 🚀 总体工作流

打造高质量 MCP 服务器通常分为四个阶段：

### 第一阶段：深入调研与规划

#### 1.1 理解以智能体为中心的设计原则

在开始实现之前，先了解如何为 AI 智能体设计工具，熟悉以下原则：

**围绕工作流设计，而非简单包裹 API 端点：**
- 不要仅仅把现有 API 端点原样暴露——应打造高价值的工作流工具
- 将相关操作整合（例如 `schedule_event` 同时检查空闲并创建事件）
- 专注于能够完成完整任务的工具，而非单一 API 调用
- 思考智能体在真实场景中需要完成哪些工作流

**优化有限的上下文窗口：**
- 智能体的上下文窗口有限——让每个 token 都物尽其用
- 返回信息聚焦高价值信号，而非大段冗余数据
- 提供“精简模式”与“详细模式”等可选响应格式
- 默认为人类可读的标识，而非技术代码（优先使用名称而非 ID）
- 将智能体的上下文预算视为稀缺资源

**设计可行动的错误信息：**
- 错误信息应引导智能体朝正确的用法前进
- 给出明确的下一步建议：“可尝试使用 filter='active_only' 来减少结果”
- 让错误提示具有教学意义，而不仅仅是诊断
- 帮助智能体通过清晰的反馈学习正确的工具用法

**遵循自然的任务拆分：**
- 工具命名应贴近人们的任务思维
- 使用一致的前缀把相关工具归类，方便发现与记忆
- 围绕自然工作流设计工具，而不是照搬 API 结构

**坚持基于评估驱动的开发：**
- 早期就创建真实评估场景
- 让智能体的反馈驱动工具改进
- 快速原型，基于智能体表现迭代

#### 1.3 学习 MCP 协议文档

**获取最新 MCP 协议文档：**

使用 WebFetch 访问：`https://modelcontextprotocol.io/llms-full.txt`

该文档包含完整的 MCP 规范与指南，务必通读。

#### 1.4 熟悉框架文档

**加载并阅读以下参考资料：**

- **MCP 最佳实践**：[📋 查看最佳实践](./reference/mcp_best_practices.md) —— 所有 MCP 服务器的核心准则

**若使用 Python 实现，还需阅读：**
- **Python SDK 文档**：使用 WebFetch 加载 `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Python 实现指南](./reference/python_mcp_server.md) —— Python 专用最佳实践与示例

**若使用 Node/TypeScript 实现，还需阅读：**
- **TypeScript SDK 文档**：使用 WebFetch 加载 `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ TypeScript 实现指南](./reference/node_mcp_server.md) —— Node/TypeScript 最佳实践与示例

#### 1.5 彻底研读目标 API 文档

要整合某个服务，需全面阅读**所有**可用 API 资料：
- 官方 API 参考文档
- 认证与授权要求
- 速率限制与分页策略
- 错误响应与状态码
- 可用端点及其参数
- 数据模型与架构

**必要时可使用网页搜索与 WebFetch 工具，获取完整信息。**

#### 1.6 制定详尽的实现方案

基于调研结果，编写包含以下内容的详细计划：

**工具选择：**
- 列出最具价值的端点 / 操作
- 优先实现覆盖常见且重要用例的工具
- 考虑工具之间如何协同完成复杂工作流

**通用工具与辅助函数：**
- 识别常见 API 请求模式
- 规划分页辅助函数
- 设计过滤与格式化工具
- 制定错误处理策略

**输入 / 输出设计：**
- 定义输入校验模型（Python 用 Pydantic，TypeScript 用 Zod）
- 设计统一的响应格式（例如 JSON 或 Markdown），并支持可配置的详情级别（如 Detailed/Concise）
- 规划大规模使用场景（成千上万用户 / 资源）
- 实施字符限制与截断策略（如 25,000 tokens）

**错误处理策略：**
- 考虑优雅的失败模式
- 设计清晰、可操作、适合 LLM 理解的自然语言错误信息，提示后续行动
- 处理速率限制与超时场景
- 覆盖认证与授权错误

---

### 第二阶段：实现

在完成全面规划后，按照语言特定的最佳实践开始实现。

#### 2.1 搭建项目结构

**Python：**
- 可将代码放在单个 `.py` 文件，或按复杂度拆分模块（见 [🐍 Python 指南](./reference/python_mcp_server.md)）
- 使用 MCP Python SDK 注册工具
- 使用 Pydantic 模型进行输入校验

**Node/TypeScript：**
- 建立规范的项目结构（见 [⚡ TypeScript 指南](./reference/node_mcp_server.md)）
- 设置 `package.json` 与 `tsconfig.json`
- 使用 MCP TypeScript SDK
- 使用 Zod 定义严格的输入校验

#### 2.2 先实现核心基础设施

**开始编码前，先编写共享工具：**
- API 请求辅助函数
- 错误处理工具
- 响应格式化函数（JSON 与 Markdown）
- 分页辅助工具
- 认证 / Token 管理逻辑

#### 2.3 系统性实现工具

针对计划中的每个工具：

**定义输入模式：**
- 使用 Pydantic（Python）或 Zod（TypeScript）进行验证
- 添加必要的约束（最小/最大长度、正则、数值范围等）
- 提供清晰、具体的字段说明
- 在字段描述中加入多样化示例

**撰写完整的文档注释 / 描述：**
- 一句话概述工具功能
- 详述用途与行为
- 明确各参数类型并附示例
- 描述完整的返回结构
- 提供使用示例（何时使用、何时不适用）
- 记录错误处理方式以及如何继续操作

**实现工具逻辑：**
- 复用共享工具，避免重复代码
- 所有 I/O 操作遵循 async/await 模式
- 实现完善的错误处理
- 支持多种响应格式（JSON 与 Markdown）
- 遵循分页参数
- 检查字符限制并在必要时截断

**添加工具注解：**
- `readOnlyHint`: true（读操作）
- `destructiveHint`: false（非破坏性操作）
- `idempotentHint`: true（重复调用结果一致）
- `openWorldHint`: true（与外部系统交互时）

#### 2.4 遵循语言特定的最佳实践

**此时，加载相应的语言指南：**

**Python：阅读 [🐍 Python 实现指南](./reference/python_mcp_server.md)，确保以下要点：**
- 正确使用 MCP Python SDK 注册工具
- 使用带 `model_config` 的 Pydantic v2 模型
- 全面使用类型注解
- 所有 I/O 采用 async/await
- 合理组织导入
- 在模块层定义常量（如 CHARACTER_LIMIT、API_BASE_URL）

**Node/TypeScript：阅读 [⚡ TypeScript 实现指南](./reference/node_mcp_server.md)，确保以下要点：**
- 正确调用 `server.registerTool`
- Zod 模式使用 `.strict()`
- 启用 TypeScript 严格模式
- 禁用 `any`，使用精确类型
- 返回值显式声明 `Promise<T>`
- 配置构建流程（如 `npm run build`）

---

### 第三阶段：审查与完善

完成初始实现后：

#### 3.1 代码质量检查

逐项自检以确保质量：
- **DRY 原则**：避免工具之间的重复代码
- **可组合性**：抽取共享逻辑作为函数
- **一致性**：类似操作返回一致格式
- **错误处理**：所有外部调用都有异常处理
- **类型安全**：完善的类型覆盖（Python 类型注解、TypeScript 类型）
- **文档完整**：每个工具具备完整注释 / 描述

#### 3.2 测试与构建

**重要提示：** MCP 服务器是长驻进程，通过 stdio/stdin 或 SSE/HTTP 等方式等待请求。若直接在主进程运行（如 `python server.py` 或 `node dist/index.js`），进程会一直阻塞。

**安全的测试方式：**
- 使用评估脚手架（见第四阶段）——推荐方案
- 在 tmux 中后台运行服务器，避免占用主进程
- 测试时使用超时命令：`timeout 5s python server.py`

**Python：**
- 检查语法：`python -m py_compile your_server.py`
- 确认导入配置无误
- 若需手动测试：在 tmux 中运行服务器，再在主进程中运行评估脚手架
- 或直接使用评估脚手架（可自行管理 stdio 传输的服务器）

**Node/TypeScript：**
- 执行 `npm run build`，确保无报错
- 确认生成 `dist/index.js`
- 手动测试时，在 tmux 中运行服务器，再在主进程执行评估脚手架
- 或直接使用评估脚手架（自动管理 stdio 传输）

#### 3.3 使用质量检查清单

为确保实现质量，请加载语言特定指南中的质量清单：
- Python：参阅 [🐍 Python 指南](./reference/python_mcp_server.md) 中的 “Quality Checklist”
- Node/TypeScript：参阅 [⚡ TypeScript 指南](./reference/node_mcp_server.md) 中的 “Quality Checklist”

---

### 第四阶段：创建评估

在完成 MCP 服务器开发后，需要构建全面的评估，验证其有效性。

**加载 [✅ 评估指南](./reference/evaluation.md)，获取完整评估说明。**

#### 4.1 明确评估目标

评估用于检验 LLM 是否能借助你的 MCP 服务器，在真实且复杂的问题中找到答案。

#### 4.2 编写 10 个评估问题

按照评估指南中的流程：

1. **工具检查**：列出可用工具并理解其能力
2. **内容探索**：使用只读操作探索可用数据
3. **问题生成**：设计 10 个复杂、真实的问题
4. **答案验证**：亲自解答并验证每个问题的答案

#### 4.3 评估要求

每个问题必须：
- **相互独立**：不依赖其他问题
- **只读**：仅需非破坏性操作
- **具备复杂度**：需要多次工具调用与深入探索
- **真实可信**：贴合实际用户关心的场景
- **可验证**：能通过字符串比对验证唯一答案
- **稳定**：答案不会随时间变化

#### 4.4 输出格式

使用以下 XML 结构：

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- 继续添加 qa_pair ... -->
</evaluation>
```

---

# 参考资源

## 📚 文档库

根据需要加载以下资源：

### 核心 MCP 文档（优先阅读）
- **MCP 协议**：通过 `https://modelcontextprotocol.io/llms-full.txt` 获取 —— 完整规格说明
- [📋 MCP 最佳实践](./reference/mcp_best_practices.md) —— 包含：
  - 服务器与工具命名规范
  - 响应格式指南（JSON vs Markdown）
  - 分页最佳实践
  - 字符限制与截断策略
  - 工具开发指南
  - 安全与错误处理标准

### SDK 文档（第一/二阶段加载）
- **Python SDK**：`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- **TypeScript SDK**：`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`

### 语言特定实现指南（第二阶段参考）
- [🐍 Python 实现指南](./reference/python_mcp_server.md)，包含：
  - 服务器初始化模式
  - Pydantic 模型示例
  - 使用 `@mcp.tool` 注册工具
  - 完整示例代码
  - 质量检查清单

- [⚡ TypeScript 实现指南](./reference/node_mcp_server.md)，包含：
  - 项目结构建议
  - Zod 模式示例
  - 使用 `server.registerTool` 注册工具
  - 完整示例代码
  - 质量检查清单

### 评估指南（第四阶段参考）
- [✅ 评估指南](./reference/evaluation.md)，涵盖：
  - 评估问题设计原则
  - 答案验证策略
  - XML 输出规范
  - 示例问答
  - 使用脚本运行评估的方法
