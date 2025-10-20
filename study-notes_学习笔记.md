# 学习笔记（基于本项目）

本笔记旨在帮助你高效上手本仓库的技能体系，并扩展到更多可直接落地的实用场景。适合：
- 企业落地 Copilot/智能体方案的工程师/产品/运营
- 需要在 Claude Code、Claude.ai 或 API 中快速复用技能的个人开发者
- 想要参考文档/二进制文件处理范式、产研流程与最佳实践的团队

你将获得：
- 学习路径与关键概念速记
- 可即用的场景 Playbook（按业务域/岗位拆解）
- 端到端实战示例（含命令、提示词与步骤）
- 技能组合与落地建议

---

## 一、学习路径（建议 2-5 小时）

1) 快速浏览（20 分钟）
- 先阅读根目录 README.md，了解技能库的结构、安装与使用方式。
- 打开 document-skills_文档技能 目录，熟悉 docx/pdf/pptx/xlsx 四类文档技能的作用边界。

2) 深入一类文档技能（60-90 分钟）
- 以 docx 或 pptx 为例，完整阅读其 SKILL.md（注意：文件内给出的“必须完整阅读”章节，直接决定你是否能一次成功）。
- 跟着文档跑通 1 个端到端任务（例如：将一份合同转换成带修订的 Markdown，分批实施修订并回写为 .docx）。

3) 体验创意与技术类技能（30-60 分钟）
- 打开 algorithmic-art_算法艺术 与 artifacts-builder_制品构建器，感受“用代码生成高质量可交付产物”的套路。
- 看看 webapp-testing_网页应用测试 的 Playwright 最佳实践，掌握如何让 Claude 执行本地 UI 自动化。

4) 组合使用（30-60 分钟）
- 从下方“技能组合 Playbook”中挑 1-2 个场景，做一次端到端连贯演练（比如：品牌主题 → PPT 模板 → 销售方案 Deck → PDF 导出）。

---

## 二、关键概念速记

- 技能（Skill）：一个包含 YAML 前言区和长文档指令的文件夹，激活后为 Claude 提供稳定的、可复用的专业行为。
- 文档技能：围绕 OOXML/PDF 的“强约束工作流”，强调可复现、最小编辑、按批次推进与严格验证。
- 成品（Artifact）：在 Claude.ai 或 Claude Code 中直接运行/预览的 HTML/图像/文件产出。推荐以单文件、可分享、可复现为目标。
- 修订模式工作流：对于法律、学术、商务文档，强制使用 Tracked Changes；按 3-10 项一批推进，避免一次性大改。

---

## 三、实用场景大全（按岗位/业务域）

以下为“拿来就能用”的高频落地场景，附推荐技能与关键步骤。

1) 法务/合规：合同/制度修订与审校
- 技能：docx + pdf
- 步骤：
  - 用 pandoc 将 DOCX 转为带修订的 Markdown；
  - 依据“修订模式工作流”分批定位 XML；
  - 使用 ooxml Document library 写入 <w:del>/<w:ins>；
  - 打包验证并导出 PDF 供签批。
- 价值：规范、可追溯、审计友好。

2) 销售/市场：品牌一致的方案 Deck 生成
- 技能：theme-factory + brand-guidelines + pptx
- 步骤：
  - 选定主题（主题工厂或自定义主题）；
  - 应用品牌色与字体；
  - 依据销售模板自动生成章节与版式；
  - 按产品/行业插入图表与占位符；
  - 导出 PDF 发送客户。

3) 运营/HR：SOP、周报与知识库文档
- 技能：docx + template-skill
- 步骤：
  - 定义统一章节结构（目标/数据/问题/行动/下周计划）；
  - 以模板批量生成周报；
  - 汇总为月报与季度复盘，自动附图表与关键指标解释。

4) 财务/分析：Excel 数据处理与报告
- 技能：xlsx + pptx
- 步骤：
  - 从 CSV/数据库导入数据到 xlsx；
  - 生成数据透视表、公式与条件格式；
  - 输出图表并插入 PPT 汇报；
  - 形成“数据 → 结论 → 行动项”的闭环页面。

5) 客服/交付：PDF 表单与资料合并
- 技能：pdf
- 步骤：
  - 批量提取 PDF 表单字段；
  - 自动填充、合并与加水印；
  - 生成交付包与目录页（TOC）。

6) 研发/QA：本地网页应用的 UI 自动化
- 技能：webapp-testing
- 步骤：
  - 用 Playwright 录制/整理关键路径用例；
  - 在 Claude 中执行并收集截图/日志；
  - 自动生成缺陷报告与重现步骤。

7) 创意设计：活动视觉与艺术化封面
- 技能：algorithmic-art + canvas-design
- 步骤：
  - 写“算法哲学”→ p5.js 生成互动成品；
  - 导出指定分辨率 PNG/PDF；
  - 转为社媒/海报/封面多尺寸。

8) 工具整合：企业内外部系统打通
- 技能：mcp-builder（MCP 服务器）
- 步骤：
  - 为 Jira/Notion/Slack/自有 API 提供工具端点；
  - 在技能中约束“何时调用/如何调用”；
  - 统一成“拉数据 → 写文档 → 发通知”的一体化流程。

---

## 四、技能组合 Playbook（示例串联）

A. 招投标/RFP 快速响应
- 组合：theme-factory + brand-guidelines + docx + pptx + pdf
- 流程：
  1) 读取 RFP 并抽取关键要求；
  2) 基于模板生成方案 Word 文档；
  3) 同步生成答辩 Deck（统一品牌与主题）；
  4) 合并附录并导出签报 PDF；
  5) 生成提交清单与版式校验报告。

B. 合同版本对比与批量修订
- 组合：docx + pdf
- 流程：
  1) pandoc 转 Markdown 审阅差异；
  2) 标注 <w:ins>/<w:del> 最小编辑块；
  3) 按章节批次执行脚本；
  4) 打包回 DOCX，导出 PDF 验证；
  5) 输出“修订清单 + 影响评估”。

C. 数据看板周报自动化
- 组合：xlsx + pptx + artifacts-builder
- 流程：
  1) 拉取一周指标至 xlsx 并计算；
  2) 生成图表并插入 PPT；
  3) 用 artifacts-builder 输出一个交互式 HTML 附件，便于领导自助筛选查看。

---

## 五、端到端实战示例

示例 1：法律条款数值替换（30 → 60）且保留修订
- 命令：
  - pandoc --track-changes=all input.docx -o current.md
  - python ooxml/scripts/unpack.py input.docx unpacked
  - 在 word/document.xml 中 grep 目标文本并定位 <w:r>
  - 使用 Document library 写入 <w:del>30</w:del> 与 <w:ins>60</w:ins>
  - python ooxml/scripts/pack.py unpacked reviewed.docx
  - pandoc --track-changes=all reviewed.docx -o verify.md
- 验证：grep 原短语应消失，替换短语应出现；打开 Word 看到清晰的红蓝修订。

示例 2：销售方案 Deck 生成
- 步骤：
  1) 在 theme-factory 选择主题；
  2) 读取品牌规范（brand-guidelines）；
  3) 用 pptx 技能定义封面、目录、章节、结尾页版式；
  4) 将客户行业案例/数据以图表形式插入；
  5) 导出为 PPTX 和 PDF。

示例 3：Excel → PPT 报表链路
- 步骤：
  1) xlsx 技能创建工作簿并写入数据/公式；
  2) 生成图表图片；
  3) 插入到 pptx 指定占位符；
  4) 一键导出周会材料。

---

## 六、在 Claude 中使用（速查）

Claude Code 插件市场：
- /plugin marketplace add anthropics/skills
- /plugin install document-skills@anthropic-agent-skills
- /plugin install example-skills@anthropic-agent-skills

提示词建议：
- 指明使用的技能与目标文件路径；
- 说明验收标准（是否要修订记录/是否保持格式/输出什么校验报告）；
- 指定分批策略与终止条件（每批不超过 10 项变更，逐批验证）。

Claude API：
- 参考 README 中的 Skills API 快速入门；
- 将本仓库的技能作为自定义技能上传，或直接调用预构建技能。

---

## 七、最佳实践与避坑

- 最小编辑策略：只修改必要的 XML 片段，保留未改文本的 <w:r> 与 RSID。
- 分批推进：每批 3-10 项变更，批批验证，严禁一次性大改。
- 可复现：固定种子、固定模板、固定参数范围，保留生成过程说明。
- 单文件成品：HTML 成品应可离线/可分享，避免外链依赖（除 p5.js CDN）。
- 审核与回滚：所有关键产物（合同/方案/报告）在导出前生成“差异与检查清单”，便于审阅与回滚。

---

## 八、更多延展场景（灵感）

- 招聘流程：解析简历（pdf）→ 结构化表格（xlsx）→ 面试评分表（docx）→ 汇总报告（pptx）。
- 供应链：批量对账（xlsx）→ 合同变更修订（docx）→ 对账回执（pdf）。
- 培训与教育：课程讲义（pptx）→ 练习册与答案（docx）→ 测试卡（pdf 表单）。
- 媒体内容：播客/视频逐字稿（docx）→ 精选摘要图卡（canvas-design）→ 社媒套版（theme-factory）。

---

如果你希望把上述任何场景做成“团队一键复用”的固定流程，建议将具体的输入/输出、验收标准与工具调用，固化为一个新的自定义 Skill（参考 template-skill_模板技能 与 skill-creator_技能创建器）。
