# MCP 服务器评估指南

## 概览

本文指导如何为 MCP 服务器编写完整评估。评估的目标是测试：仅使用 MCP 服务器提供的工具时，LLM 是否能解答真实且复杂的问题。

---

## 快速参考

### 评估要求
- 编写 10 个可读性良好的问题
- 问题必须 **只读、互不依赖、非破坏性**
- 每个问题都需要多次工具调用（可能高达数十次）
- 答案必须是单一且可验证的值
- 答案必须 **稳定**（不会随时间变化）

### 输出格式
```xml
<evaluation>
   <qa_pair>
      <question>你的问题</question>
      <answer>唯一可验证答案</answer>
   </qa_pair>
</evaluation>
```

---

## 评估的意义

评估 MCP 服务器的标准不是“工具实现得多完善”，而是工具的输入/输出模式、说明与功能能否让 **仅依赖 MCP 服务器的 LLM** 解答真实、复杂的问题。

## 评估概述

编写 10 个仅需执行 **只读、独立、非破坏、幂等** 操作的问题。每个问题应当：
- 真实可信
- 清晰简洁
- 无歧义
- 复杂，需要数十步或多次工具调用
- 拥有你事先确认的、单一可验证答案

## 出题指南

### 核心要求

1. **问题必须相互独立**
   - 不得依赖其他问题的答案
   - 不假定之前执行过任何写操作

2. **问题必须仅使用非破坏性、幂等的工具**
   - 不得要求修改状态

3. **问题必须真实、清晰、简洁且复杂**
   - 需要另一个 LLM 调用多次工具才能解答

### 复杂度与深度

4. **问题应要求深入探索**
   - 可以是多跳问题，需要连续子问题与工具调用
   - 每一步都依赖前面获得的信息

5. **允许需要大量分页**
   - 可能需要遍历多页结果
   - 可以要求查询历史数据（1-2 年前）寻找细节
   - **问题必须困难**

6. **问题应考察深层理解**
   - 不局限于表面信息
   - 可设置判断题，需要证据支持
   - 可用多选题，迫使 LLM 验证不同假设

7. **不得通过简单关键词搜索解决**
   - 避免直接给出目标内容的关键词
   - 使用同义词、相关概念或转述
   - 需要多次检索、分析多个条目、整合上下文再得出答案

### 工具压力测试

8. **问题应考验工具返回值**
   - 可能触发大 JSON / 列表，考验 LLM 承载能力
   - 涉及多种数据类型：ID、名称、时间戳、文件信息、URL 等
   - 确认工具能返回所有有用数据

9. **问题应贴近真实人类需求**
   - 关注人类+LLM 协作时真正关心的信息检索

10. **问题可以需要几十次调用**
    - 逼迫 LLM 面对上下文限制
    - 促使工具减少冗余输出

11. **可包含模糊问题**
    - 迫使 LLM 决定调用何种工具
    - 虽然存在歧义，但仍需有唯一可验证答案

### 稳定性

12. **答案必须不会变化**
    - 不要依赖动态“当前状态”
    - 避免询问反应数、回复数、成员数等会变动的数据

13. **不要被 MCP 服务器限制**
    - 提出具有挑战性的问题
    - 即便有些题目工具无法完成也没关系
    - 可要求特定输出格式（日期、JSON、Markdown 等）
    - 可需要大量调用才能完成

## 答案指南

### 可验证性

1. **答案必须可通过字符串直接比较**
   - 如答案可能有多种格式，需在问题中明确格式要求
   - 例：“使用 YYYY/MM/DD”“仅回答 True 或 False”“仅回复 A/B/C/D”
   - 答案应为单一可验证值，例如：
     - 用户 ID、用户名、显示名、姓、名
     - 频道 ID、频道名称
     - 消息 ID、消息内容
     - URL、标题
     - 数值
     - 时间戳、日期时间
     - 布尔值
     - 邮箱、电话
     - 文件 ID、文件名、扩展名
     - 多选题答案
   - 不得要求复杂格式或多段输出
   - 验证方式为“直接字符串比较”

### 可读性

2. **优先使用人类易读格式**
   - 如姓名、日期时间、文件名、URL、Yes/No、True/False、A/B/C/D
   - 虽然允许使用 ID，但建议多数答案可读

### 稳定性

3. **答案必须稳定**
   - 优先使用历史数据、已经完成的项目
   - 可限定固定时间窗口，避免结果波动
   - 依赖不太可能改变的上下文
   - 若寻找论文名称等内容，要具体到不会与后续发布的资料混淆

4. **答案必须明确无歧义**
   - 设计问题时保证答案唯一且可由工具推导

### 多样性

5. **答案要多样**
   - 覆盖多种类型（用户、频道、消息、文件等）
   - 举例：用户名、邮箱、频道名称、消息时间、文件扩展名等

6. **避免复杂结构**
   - 不要要求返回列表或复杂对象
   - 除非可以直接比较字符串、且格式容易保持一致

## 评估流程

### 第一步：阅读文档
- 了解目标 API 的端点与功能
- 若不明确，可上网补充资料
- 尽量并行进行
- 只阅读文件系统或互联网上的文档，**不要**读服务器源代码

### 第二步：工具梳理
- 列出 MCP 服务器提供的所有工具
- 熟悉输入/输出模式与说明
- 此阶段**不要**实际调用工具

### 第三步：加深理解
- 在步骤 1 与 2 间反复迭代
- 思考要构造哪些任务
- 形成合理而具挑战性的题目
- 同样**不要**阅读服务器实现代码

### 第四步：只读内容检索
- 在理解工具后，开始调用 MCP 工具
- **仅**使用只读、非破坏操作
- 目标：锁定可用于出题的具体内容（用户、频道、消息、项目、任务等）
- 不调用修改状态的工具
- 可并行进行，每个子智能体独立探索
- 仅执行只读、幂等操作
- 注意：有些工具返回数据量巨大，谨慎控制上下文
- 采用小步、精准的调用
- 所有请求都使用 `limit` 参数限制结果数（<10）
- 合理分页

### 第五步：生成任务
- 根据收集到的内容编写 10 个问题
- 满足前文所有问题与答案的要求

## 输出格式

每个 QA 对由问题与答案组成，使用如下 XML 结构：

```xml
<evaluation>
   <qa_pair>
      <question>在 2024 年 Q2 创建的项目中，完成任务数最多的是哪一个？请给出项目名称。</question>
      <answer>Website Redesign</answer>
   </qa_pair>
   ...
</evaluation>
```

## 评估示例

### 优秀问题

**示例 1：需要多步探索（GitHub MCP）**
```xml
<qa_pair>
   <question>找出在 2023 年 Q3 被归档、且归档前曾是组织内被 fork 次数最多的仓库。这个仓库使用的主要编程语言是什么？</question>
   <answer>Python</answer>
</qa_pair>
```
优点：
- 需多次检索归档仓库
- 比较归档前的 fork 数
- 读取仓库详情查看语言
- 答案简单可验证
- 基于已完成的历史数据

**示例 2：避免直接关键词（项目管理 MCP）**
```xml
<qa_pair>
   <question>定位 2023 年底完成的、聚焦改进客户引导体验的计划。项目负责人在收尾后写了复盘文档。他当时的职务头衔是什么？</question>
   <answer>Product Manager</answer>
</qa_pair>
```
优点：
- 未直接给出项目名称
- 需要按时间筛选项目
- 找到负责人及其角色
- 需要阅读复盘文档
- 答案稳定且可读

**示例 3：复杂汇总（缺陷追踪 MCP）**
```xml
<qa_pair>
   <question>在 2024 年 1 月被标记为关键优先级的所有缺陷中，哪位受理人 48 小时内解决的占比最高？请给出用户名。</question>
   <answer>alex_eng</answer>
</qa_pair>
```
优点：
- 需筛选日期、优先级、状态
- 按受理人分组计算解决率
- 使用时间戳判断 48 小时
- 可能需要分页处理大量缺陷
- 答案是单一用户名

**示例 4：跨多数据类型（CRM MCP）**
```xml
<qa_pair>
   <question>查找在 2023 年 Q4 从 Starter 升级到 Enterprise 方案、且年度合同价值最高的账户。该账户所在行业是什么？</question>
   <answer>Healthcare</answer>
</qa_pair>
```
优点：
- 涉及订阅层级变化
- 需要识别特定时间段的升级事件
- 比较合同价值
- 获取账户行业信息
- 答案简单可验证

### 不佳问题

**示例 1：答案随时间变化**
```xml
<qa_pair>
   <question>目前有多少开放问题指派给工程团队？</question>
   <answer>47</answer>
</qa_pair>
```
问题：答案会随状态变化而改变。

**示例 2：过于依赖关键词**
```xml
<qa_pair>
   <question>找到标题为 “Add authentication feature” 的 PR，并告诉我是谁创建的。</question>
   <answer>developer123</answer>
</qa_pair>
```
问题：通过搜索精确标题即可完成，不需深入分析。

**示例 3：答案格式不明确**
```xml
<qa_pair>
   <question>列出所有主要语言为 Python 的仓库。</question>
   <answer>repo1, repo2, repo3, data-pipeline, ml-tools</answer>
</qa_pair>
```
问题：答案是列表，顺序不固定，难以通过字符串验证。

## 验证流程

生成评估后：
1. **检查 XML**，确认结构正确
2. **加载每道题**，并使用 MCP 工具亲自求解，确认答案
3. **标记需要写操作的任务**（如有）
4. **收集所有正确答案**，替换错误项
5. **删除任何需要写/破坏性操作的 `<qa_pair>`**

尽量并行求解，最后汇总并修改文件。

## 高质量评估小贴士

1. 在出题前充分思考与规划
2. 有条件就并行处理，节约上下文
3. 聚焦真实用户场景
4. 设计具挑战性的问题，考验 MCP 能力边界
5. 使用历史数据保障稳定性
6. 亲自用 MCP 工具验证答案
7. 在过程中不断迭代优化

---

# 执行评估

撰写评估文件后，可使用提供的评估脚本测试 MCP 服务器。

## 环境准备

1. **安装依赖**
   ```bash
   pip install -r scripts/requirements.txt
   ```
   或单独安装：
   ```bash
   pip install anthropic mcp
   ```

2. **设置 API Key**
   ```bash
   export ANTHROPIC_API_KEY=your_api_key_here
   ```

## 评估文件格式

评估采用 XML，包含 `<qa_pair>` 元素，结构例如：

```xml
<evaluation>
   <qa_pair>
      <question>在 2024 年 Q2 创建的项目中，完成任务数最多的是哪一个？</question>
      <answer>Website Redesign</answer>
   </qa_pair>
   <qa_pair>
      <question>在 2024 年 3 月关闭、带 “bug” 标签的问题中，谁关闭的数量最多？请给出用户名。</question>
      <answer>sarah_dev</answer>
   </qa_pair>
</evaluation>
```

## 运行评估

评估脚本（`scripts/evaluation.py`）支持三种传输方式：

- **stdio（默认）**：脚本会自动启动并管理 MCP 服务器，不要手动运行服务器。
- **sse/http**：需要你先行启动服务器，脚本会连接指定 URL。

### 1. 本地 STDIO 服务器

```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  evaluation.xml
```

带环境变量：
```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_mcp_server.py \
  -e API_KEY=abc123 \
  -e DEBUG=true \
  evaluation.xml
```

### 2. SSE（Server-Sent Events）

```bash
python scripts/evaluation.py \
  -t sse \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  -H "X-Custom-Header: value" \
  evaluation.xml
```

### 3. HTTP（可流式 HTTP）

```bash
python scripts/evaluation.py \
  -t http \
  -u https://example.com/mcp \
  -H "Authorization: Bearer token123" \
  evaluation.xml
```

## 命令行参数

```
usage: evaluation.py [-h] [-t {stdio,sse,http}] [-m MODEL] [-c COMMAND]
                     [-a ARGS [ARGS ...]] [-e ENV [ENV ...]] [-u URL]
                     [-H HEADERS [HEADERS ...]] [-o OUTPUT]
                     eval_file
```

- `eval_file`：评估 XML 路径
- `-t/--transport`：传输方式，默认为 stdio
- `-m/--model`：使用的 Claude 模型，默认 `claude-3-7-sonnet-20250219`
- `-o/--output`：输出报告路径（默认打印到标准输出）
- `-c/--command`：启动 MCP 服务器的命令（如 `python`、`node`）
- `-a/--args`：命令参数（如 `server.py`）
- `-e/--env`：设置环境变量 `KEY=VALUE`
- `-u/--url`：SSE/HTTP 模式的服务器地址
- `-H/--header`：HTTP 头（`Key: Value`）

## 输出报告

脚本会生成详细报告，包含：
- **总体统计**：准确率、平均任务时长、平均调用次数、总调用次数
- **单题结果**：指令、预期答案、实际回答、是否正确、耗时、调用细节、智能体总结与反馈

可将报告输出到文件：
```bash
python scripts/evaluation.py \
  -t stdio \
  -c python \
  -a my_server.py \
  -o evaluation_report.md \
  evaluation.xml
```

## 完整示例流程

1. **编写评估文件**（`my_evaluation.xml`）：
   ```xml
   <evaluation>
      <qa_pair>
         <question>找出在 2024 年 1 月创建问题最多的用户，并给出用户名。</question>
         <answer>alice_developer</answer>
      </qa_pair>
      <qa_pair>
         <question>在 2024 年 Q1 合并的所有 PR 中，哪一个仓库的合并数量最高？</question>
         <answer>backend-api</answer>
      </qa_pair>
      <qa_pair>
         <question>查找 2023 年 12 月完成且周期最长的项目，持续了多少天？</question>
         <answer>127</answer>
      </qa_pair>
   </evaluation>
   ```

2. **安装依赖并设置环境变量**
   ```bash
   pip install -r scripts/requirements.txt
   export ANTHROPIC_API_KEY=your_api_key
   ```

3. **运行评估**
   ```bash
   python scripts/evaluation.py \
     -t stdio \
     -c python \
     -a github_mcp_server.py \
     -e GITHUB_TOKEN=ghp_xxx \
     -o github_eval_report.md \
     my_evaluation.xml
   ```

4. **查看报告**（`github_eval_report.md`），了解通过/失败题目、工具反馈与改进方向。

## 故障排查

### 连接错误
- **STDIO**：确认命令与参数正确
- **SSE/HTTP**：检查 URL 可达、头部是否正确
- 确保所需 API Key 已设置

### 准确率低
- 阅读每题的工具反馈
- 检查工具说明是否清晰、参数是否完善
- 评估工具返回数据量是否过多或不足
- 确认错误信息是否具备指导意义

### 超时问题
- 尝试更强的模型（如 `claude-3-7-sonnet-20250219`）
- 检查工具是否返回过多数据
