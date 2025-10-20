# OOXML 实操指南（DOCX 编辑）

> 适用于 `document-skills/docx` 中的 OOXML 直接编辑工作流。  
> 请在动手前完整阅读，尤其是“结构概览”“常见错误”部分，可避免生成损坏的 Word 文档。

---

## 目录
1. OOXML 基础概念速览  
2. 解包/重打包工作流  
3. 文档主体 `word/document.xml`  
4. 常用节点与属性示例  
5. 运行节点 `<w:r>` 与文本 `<w:t>`  
6. 段落 `<w:p>`、样式与编号  
7. 表格、图片、超链接  
8. 页眉页脚、页码、分节符  
9. 修订、批注、脚注等高级特性  
10. 常见错误与排障

---

## 1. OOXML 基础概念速览

- `.docx` 本质上是一个 ZIP 包，内部是 XML + 媒体文件。  
- 主体内容位于 `word/document.xml`，页眉/页脚在 `word/header*.xml`、`word/footer*.xml`。  
- Word 通过 **命名空间**（如 `w:`, `r:`）区分不同功能模块。  
- 任意改动后都需使用 `pack.py` 重新打包，确保使用 UTF-8、CRLF/行终止一致。  
- 修改 XML 时务必遵循 **大小写严格** 与 **完整闭合**。

---

## 2. 解包 / 重打包工作流

```bash
python ooxml/scripts/unpack.py input.docx unpacked/
# 在 unpacked 目录下编辑 XML / 媒体文件
python ooxml/scripts/pack.py unpacked/ output.docx
```

### 常见注意事项
1. **不要使用操作系统自带的压缩工具**，需依赖脚本维持 `[Content_Types].xml`、`_rels` 等结构。  
2. 若脚本报错，可先 `rm -rf unpacked` 再重新解包，避免残留文件。  
3. 编辑 XML 建议使用支持 XML 高亮的编辑器并开启格式化插件。

---

## 3. `word/document.xml` 结构概览

```xml
<w:document>
  <w:body>
    <w:p> ... 段落 ... </w:p>
    <w:tbl> ... 表格 ... </w:tbl>
    <w:sectPr> ... 分节设置（页眉/页脚、页边距等） ... </w:sectPr>
  </w:body>
</w:document>
```

- `w:p`：段落节点。  
- `w:r`：run（运行）节点，包含具体文本、格式。  
- `w:t`：文本内容。  
- `w:tbl`：表格；`w:tr` 行、`w:tc` 单元格。  
- `w:hyperlink`、`w:footnoteReference` 等用于高级特性。

**命名空间（常见）**
- `w:` → WordprocessingML 核心：`http://schemas.openxmlformats.org/wordprocessingml/2006/main`
- `r:` → 关系：`http://schemas.openxmlformats.org/officeDocument/2006/relationships`
- `wp:`, `a:`（DrawingML）→ 图片/图形相关

---

## 4. 关键节点速查

| 元素 | 用途 | 常见属性 |
|------|------|----------|
| `<w:p>` | 段落 | `w:pPr`（段落属性）、`w:r`、`w:bookmarkStart` 等 |
| `<w:pPr>` | 段落属性 | `w:spacing`（段距）、`w:ind`（缩进）、`w:numPr`（列表） |
| `<w:r>` | 运行（文本块） | 可包含 `w:rPr`（字体、大小）、`w:t` |
| `<w:t>` | 文本 | 若包含空格/特殊符号，需 `xml:space="preserve"` |
| `<w:tbl>` | 表格 | `w:tblPr`（属性）、`w:tblGrid`、`w:tr` |
| `<w:drawing>` | 图片/形状容器 | 内含 DrawingML 结构 |

---

## 5. Run 与文本节点

```xml
<w:p>
  <w:r>
    <w:rPr>
      <w:b/>                 <!-- Bold -->
      <w:color w:val="FF0000"/>
      <w:sz w:val="28"/>      <!-- 字号：26=13pt -->
    </w:rPr>
    <w:t xml:space="preserve">示例文本</w:t>
  </w:r>
</w:p>
```

### 技巧
- 多个不同格式需拆成多个 `w:r`。  
- 若文本开头/结尾包含空格，请添加 `xml:space="preserve"`。  
- 字号单位为 **半点**（Twenty points）→ 24 表示 12pt。

---

## 6. 段落、样式与列表

### 段落属性 `w:pPr`

```xml
<w:p>
  <w:pPr>
    <w:pStyle w:val="Heading1"/>
    <w:spacing w:before="240" w:after="120"/>
    <w:ind w:left="720" w:hanging="360"/>
    <w:numPr>
      <w:ilvl w:val="0"/>
      <w:numId w:val="2"/>
    </w:numPr>
  </w:pPr>
  ...
</w:p>
```

- `w:pStyle`：引用样式 ID（需在 `styles.xml` 中定义）。  
- `w:spacing`、`w:ind`：段距、缩进。  
- 列表需要 `w:numPr` → 先在 `numbering.xml` 中定义抽象列表，再引用 `numId`。

### 列表定义（`word/numbering.xml`）

```xml
<w:numbering>
  <w:abstractNum w:abstractNumId="1">
    <w:lvl w:ilvl="0">
      <w:numFmt w:val="bullet"/>
      <w:lvlText w:val="•"/>
      <w:lvlJc w:val="left"/>
      <w:pPr>
        <w:ind w:left="720" w:hanging="360"/>
      </w:pPr>
    </w:lvl>
  </w:abstractNum>
  <w:num w:numId="2">
    <w:abstractNumId w:val="1"/>
  </w:num>
</w:numbering>
```

---

## 7. 表格、图片、超链接

### 表格

```xml
<w:tbl>
  <w:tblPr>
    <w:tblW w:w="9360" w:type="dxa"/> <!-- 表宽 -->
    <w:tblLook w:firstRow="1" w:band1Horz="1"/>
  </w:tblPr>
  <w:tblGrid>
    <w:gridCol w:w="4680"/>
    <w:gridCol w:w="4680"/>
  </w:tblGrid>
  <w:tr>
    <w:tc>
      <w:tcPr>
        <w:tcW w:w="4680" w:type="dxa"/>
        <w:shd w:fill="D5E8F0" w:val="clear"/>
      </w:tcPr>
      <w:p>...</w:p>
    </w:tc>
  </w:tr>
</w:tbl>
```

### 图片

位于 `w:drawing` 下，指向 `word/media/` 中的图片，并通过 `word/_rels/document.xml.rels` 中的 `<Relationship>` 建立关联。

```xml
<w:drawing>
  <wp:inline>
    <a:graphic>
      <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
        <pic:pic>
          ...
          <a:blip r:embed="rId5"/> <!-- 对应 media 文件的关系 ID -->
        </pic:pic>
      </a:graphicData>
    </a:graphic>
  </wp:inline>
</w:drawing>
```

### 超链接

```xml
<w:hyperlink r:id="rId10">
  <w:r>
    <w:rPr>
      <w:rStyle w:val="Hyperlink"/>
    </w:rPr>
    <w:t>访问链接</w:t>
  </w:r>
</w:hyperlink>
```

`r:id` 需在 `document.xml.rels` 中声明链接地址。

---

## 8. 页眉页脚、分节符

- 每个分节 (`<w:sectPr>`) 可绑定不同的 header/footer。  
- 页边距、纸张方向等也位于 `sectPr`。  
- 分节符可通过在段落属性中插入 `<w:sectPr/>` 或使用 `w:type` 值为 `nextPage` 的 `w:break`。

```xml
<w:sectPr>
  <w:pgSz w:w="12240" w:h="15840" w:orient="landscape"/>
  <w:pgMar w:top="1440" w:right="1440" w:bottom="1440" w:left="1440"/>
  <w:headerReference r:id="rId13" w:type="default"/>
  <w:footerReference r:id="rId14" w:type="default"/>
</w:sectPr>
```

对应的 header/footer 内容位于 `word/header1.xml`、`word/footer1.xml`。

---

## 9. 高级特性

### 修订（Tracked Changes）

```xml
<w:p>
  <w:r>
    <w:ins w:id="5" w:author="User" w:date="2024-01-01T12:00:00Z">
      <w:r>
        <w:t>新增文本</w:t>
      </w:r>
    </w:ins>
  </w:r>
</w:p>
```

删除：`<w:del><w:r><w:delText>...</w:delText></w:r></w:del>`。务必保持原始 RSID、Author 信息一致以避免 Word 报错。

### 批注

- 在正文中插入 `<w:commentReference w:id="0"/>`。  
- 批注内容存放于 `word/comments.xml`。

### 脚注与尾注

- 正文使用 `<w:footnoteReference w:id="2"/>`。  
- 具体内容在 `word/footnotes.xml` 中。

---

## 10. 常见错误与排障

1. **忘记更新关系文件**：新增图片、超链接后必须同步更新 `document.xml.rels`。  
2. **非法命名空间或大小写**：XML 区分大小写，如 `<w:Tbl>` 会导致 Word 报错。  
3. **段落/运行缺少闭合**：务必确保 `<w:p>`、`<w:r>` 成对出现。  
4. **空 `w:t` 节点**：若需要空行，应该创建空 `w:p`，不要保留空 `w:t`。  
5. **编号重复覆盖**：若列表编号混乱，检查 `numbering.xml` 中的 `abstractNumId`/`numId` 是否复用错误。  
6. **媒体未复制**：插入图片后务必将文件复制到 `word/media/` 并保持名称一致。  
7. **页眉页脚缺失**：若 Word 警告“页眉/页脚缺失”，确认 `header*.xml`、`footer*.xml` 有对应关系 ID。  
8. **打包失败**：确认 `pack.py` 输出日志中 `done`，若失败可删除 `~$` 临时文件后重试。

---

## 结语

通过解包-编辑-重打包流程，可对 DOCX 进行像素级控制。操作 OOXML 时，保持“结构完备”“命名准确”“关系同步”是最重要的三条原则。建议在大量修改前先对小文档试验，确认 Word 能正常打开并通过“文件 → 信息 → 检查问题”自检工具。祝你写出高质量的 `.docx` 模板！
