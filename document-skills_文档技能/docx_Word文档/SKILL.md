---
name: docx
description: 「面向 .docx 文件的文档创建、编辑与分析」——支持修订记录、批注、格式保持与文本抽取。当 Claude 需要处理专业 Word 文档（.docx）以执行以下任一任务时请使用本技能：(1) 创建新文档；(2) 修改或编辑内容；(3) 处理修订记录；(4) 添加批注；或其他文档相关工作。
license: 专有许可。完整条款见 LICENSE.txt
---

# DOCX 文档的创建、编辑与分析

## 概览

用户可能会要求你创建、编辑或分析 .docx 文件的内容。DOCX 本质上是包含 XML 文件与其他资源的 ZIP 压缩包，你可以读取或编辑其中的内容。针对不同任务，你可以选择不同的工具与工作流。

## 工作流决策树

### 阅读 / 分析内容
使用“文本抽取”或“原始 XML 访问”部分所述方法。

### 创建新文档
使用“创建全新 Word 文档”工作流。

### 编辑现有文档
- **你自己创作的文档 + 简单修改**  
  使用“基础 OOXML 编辑”工作流

- **他人提供的文档**  
  使用 **“修订模式工作流”**（推荐默认）

- **法律、学术、商务或政府文档**  
  使用 **“修订模式工作流”**（必须）

## 阅读与分析内容

### 文本抽取
如仅需阅读文档文字，可使用 pandoc 将其转换为 Markdown。Pandoc 在保持结构方面表现优秀，并能展示修订记录：

```bash
# 将文档转换成带修订的 Markdown
pandoc --track-changes=all path-to-file.docx -o output.md
# 可选项：--track-changes=accept/reject/all
```

### 原始 XML 访问
当需要处理批注、复杂格式、文档结构、嵌入媒体或元数据时，就必须访问原始 XML。此类需求应先解包文档并阅读其中的 XML 文件。

#### 解包文件
`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### 关键文件结构
* `word/document.xml` —— 主体内容
* `word/comments.xml` —— 与 document.xml 对应的批注
* `word/media/` —— 嵌入的图像与媒体
* 修订记录使用 `<w:ins>`（插入）与 `<w:del>`（删除）标签

## 创建全新 Word 文档

从头开始创建新文档时，请使用 **docx-js**，可通过 JavaScript/TypeScript 构建 Word 文件。

### 工作流
1. **强制要求 —— 彻底阅读**：完整阅读 [`docx-js.md`](docx-js.md)（约 500 行）。**阅读时严禁限制行号范围**。需通读文件，了解语法、关键格式规则与最佳实践，再着手创建文档。
2. 使用 Document、Paragraph、TextRun 组件编写 JavaScript/TypeScript 文件（可假设依赖均已安装，如未安装请参阅文档末尾依赖说明）。
3. 通过 `Packer.toBuffer()` 导出为 .docx。

## 编辑现有 Word 文档

编辑现有文档时，请使用 **Document library**（基于 Python 的 OOXML 操作库）。该库会自动处理基础设施，并提供多种文档操作方法。对于复杂场景，可以直接通过库访问底层 DOM。

### 工作流
1. **强制要求 —— 彻底阅读**：完整阅读 [`ooxml.md`](ooxml.md)（约 600 行）。**阅读时严禁限制行号范围。** 需通读文件，掌握 Document library API 以及直接编辑文档时的 XML 模式。
2. 解包文档：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 使用 Document library 编写并运行 Python 脚本（参阅 ooxml.md 中 “Document Library” 章节）
4. 重新打包最终文档：`python ooxml/scripts/pack.py <input_directory> <office_file>`

Document library 提供常用操作的高级方法，也允许你在复杂场景下直接操作 DOM。

## 修订模式工作流

该工作流允许你先用 Markdown 规划完整的修订，再在 OOXML 中逐步实现。**重点**：要完整记录修订，必须系统性地实施所有变更。

**批处理策略**：将相关变更按 3-10 项归为一批。这样既便于调试，也能保持效率。每批测试通过后再继续下一批。

**原则：最小且精准的编辑**
处理修订时，仅标记实际修改的内容。重复标记未变部分会让审阅更困难，也显得不专业。请将替换拆分为：【未改文本】+【删除段】+【插入段】+【未改文本】。未改文本需保留原 `<w:r>` 的 RSID，可通过提取原文档中的 `<w:r>` 元素后重复使用来实现。

示例 —— 将句子中的 “30 days” 改为 “60 days”：
```python
# 错误示例：替换整句
'<w:del><w:r><w:delText>The term is 30 days.</w:delText></w:r></w:del><w:ins><w:r><w:t>The term is 60 days.</w:t></w:r></w:ins>'

# 正确示例：仅标记变化部分，并复用未改文本的 <w:r>
'<w:r w:rsidR="00AB12CD"><w:t>The term is </w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t> days.</w:t></w:r>'
```

### 修订工作流步骤

1. **获取 Markdown 表示**：将文档转换为保留修订记录的 Markdown：
   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **识别并分组所有修改**：审阅文档，找出所有需要的修改，并按逻辑分批：

   **定位方法**（在 XML 中定位需要修改的位置）：
   - 章节或标题编号（如 “Section 3.2”“Article IV”）
   - 若有编号的段落标识
   - 使用独特上下文的 grep 模式
   - 文档结构（如“第一段”“签名区”）
   - **切勿使用 Markdown 行号** —— 它们与 XML 结构不对应

   **批次组织**（每批 3-10 项变更）：
   - 按章节：如“批次 1：第 2 节修改”“批次 2：第 5 节更新”
   - 按类型：如“批次 1：日期修正”“批次 2：当事人名称变更”
   - 按复杂度：先处理简单替换，再处理复杂结构调整
   - 按顺序：如“批次 1：第 1-3 页”“批次 2：第 4-6 页”

3. **阅读文档并解包**：
   - **强制要求 —— 彻底阅读**：完整阅读 [`ooxml.md`](ooxml.md)（约 600 行）。**切勿限制行号范围。**重点关注“Document Library”与“Tracked Change Patterns”章节。
   - **解包文档**：`python ooxml/scripts/unpack.py <file.docx> <dir>`
   - **记录建议的 RSID**：解包脚本会输出建议使用的 RSID，复制备用（用于第 4 步的修订）。

4. **分批实施修改**：按逻辑将变更分组（按章节、按类型或按位置），每批在同一个脚本中实现。这种方式：
   - 便于调试（批量较小更易定位问题）
   - 便于迭代推进
   - 保持效率（3-10 项一批通常效果较好）

   **推荐分组方式：**
   - 按文档章节（如“第 3 节修改”“定义部分”“终止条款”）
   - 按变更类型（如“日期变更”“当事人名称更新”“法律术语替换”）
   - 按位置（如“第 1-3 页修改”“文档前半部分”）

   对于每批相关变更：

   **a. 将文本映射到 XML**：在 `word/document.xml` 中 grep 目标文本，确认其在 `<w:r>` 中的拆分方式。

   **b. 编写并运行脚本**：使用 `get_node` 定位节点，实施修改后调用 `doc.save()`。具体模式详见 ooxml.md 中的 **“Document Library”** 章节。

   **注意**：每次编写脚本前都要立即在 `word/document.xml` 中 grep，获取最新行号并确认文本。每次脚本运行后行号可能改变。

5. **重新打包文档**：全部批次完成后，将解包目录打包回 .docx：
   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最终验证**：对整份文档进行全面检查：
   - 将最终文档转换为带修订的 Markdown：
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - 核实所有修改均已生效：
     ```bash
     grep "original phrase" verification.md  # 不应再出现原短语
     grep "replacement phrase" verification.md  # 应能找到替换短语
     ```
   - 确认未引入意外改动

## 将文档转换为图像

如需从视觉上分析 Word 文档，可使用两步流程转换为图像：

1. **将 DOCX 转为 PDF**：
   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **将 PDF 页面转为 JPEG 图像**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   将生成 `page-1.jpg`、`page-2.jpg` 等文件。

常用选项：
- `-r 150`：分辨率 150 DPI（可根据质量/大小调整）
- `-jpeg`：输出 JPEG（若需 PNG，请用 `-png`）
- `-f N`：起始页（如 `-f 2` 从第 2 页开始）
- `-l N`：结束页（如 `-l 5` 到第 5 页结束）
- `page`：输出文件前缀

指定页码范围示例：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page  # 仅转换第 2-5 页
```

## 代码风格指南
**重要**：编写与 DOCX 相关的代码时请注意：
- 代码要简洁
- 避免冗长变量名与重复操作
- 避免多余的 print 输出

## 依赖项

如环境中缺少，请安装以下依赖：

- **pandoc**：`sudo apt-get install pandoc`（用于文本抽取）
- **docx**：`npm install -g docx`（用于创建新文档）
- **LibreOffice**：`sudo apt-get install libreoffice`（用于 PDF 转换）
- **Poppler**：`sudo apt-get install poppler-utils`（pdftoppm 所需，用于将 PDF 转成图像）
- **defusedxml**：`pip install defusedxml`（安全解析 XML）
