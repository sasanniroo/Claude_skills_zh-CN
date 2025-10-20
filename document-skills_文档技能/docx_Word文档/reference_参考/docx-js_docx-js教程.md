# DOCX 库教程（docx-js）

使用 JavaScript/TypeScript 生成 `.docx` 文件。

**重要：在动手前请完整阅读本教程。** 其中包含关键的格式规则与常见陷阱，跳读可能导致文档损坏或显示异常。

## 环境准备

假设已全局安装 `docx`

如未安装：`npm install -g docx`

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell, ImageRun, Media, 
        Header, Footer, AlignmentType, PageOrientation, LevelFormat, ExternalHyperlink, 
        InternalHyperlink, TableOfContents, HeadingLevel, BorderStyle, WidthType, TabStopType, 
        TabStopPosition, UnderlineType, ShadingType, VerticalAlign, SymbolRun, PageNumber,
        FootnoteReferenceRun, Footnote, PageBreak } = require('docx');

// 创建并保存文档
const doc = new Document({ sections: [{ children: [/* content */] }] });
Packer.toBuffer(doc).then(buffer => fs.writeFileSync("doc.docx", buffer)); // Node.js
Packer.toBlob(doc).then(blob => { /* 浏览器下载逻辑 */ }); // Browser
```

**关键提示：**
- 使用不同 `reference` 即可在同一文档中创建独立编号区块
- 需要继续编号时沿用同一 `reference`

## 表格

```javascript
// 带边距、边框、表头与列表的完整示例
const tableBorder = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const cellBorders = { top: tableBorder, bottom: tableBorder, left: tableBorder, right: tableBorder };

new Table({
  columnWidths: [4680, 4680], // ⚠️ 必须在表级设置列宽（DXA 单位）
  margins: { top: 100, bottom: 100, left: 180, right: 180 },
  rows: [
    new TableRow({
      tableHeader: true,
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 单元格同样要设置宽度
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR }, 
          verticalAlign: VerticalAlign.CENTER,
          children: [new Paragraph({ 
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "Header", bold: true, size: 22 })]
          })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          children: [new Paragraph({ 
            alignment: AlignmentType.CENTER,
            children: [new TextRun({ text: "Bullet Points", bold: true, size: 22 })]
          })]
        })
      ]
    }),
    new TableRow({
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          children: [new Paragraph({ children: [new TextRun("Regular data")] })]
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA },
          children: [
            new Paragraph({ numbering: { reference: "bullet-list", level: 0 }, children: [new TextRun("First bullet point")] }),
            new Paragraph({ numbering: { reference: "bullet-list", level: 0 }, children: [new TextRun("Second bullet point")] })
          ]
        })
      ]
    })
  ]
})
```

**表格要点：**
- 同时设置 `columnWidths` 与单元格 `width`
- DXA：Word 内部单位，1440 表示 1 英寸；Letter 纸（1" 边距）有效宽度 9360 DXA
- 边框要应用到 `TableCell`，不要直接加在 `Table`
- 滚动式列宽示例：
  - 2 列：`[4680, 4680]`
  - 3 列：`[3120, 3120, 3120]`

## 链接与导航

```javascript
// 自动目录（必须配合 HeadingLevel 标题）
new TableOfContents("Table of Contents", { hyperlink: true, headingStyleRange: "1-3" }),

// 外部链接
new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "Google", style: "Hyperlink" })],
    link: "https://www.google.com"
  })]
}),

// 内部链接（需要书签）
const bookmarkId = "bookmark-internal-link";
new Paragraph({
  children: [
    new Bookmark({
      id: bookmarkId,
      children: [new TextRun({ text: "Bookmark destination" })]
    })
  ]
}),
new Paragraph({
  children: [new InternalHyperlink({
    children: [new TextRun({ text: "Go to bookmark", style: "Hyperlink" })],
    anchor: bookmarkId
  })]
});
```

## 页面设置

```javascript
const doc = new Document({
  sections: [{
    properties: {
      page: {
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }, // 1 英寸
        size: { orientation: PageOrientation.LANDSCAPE },
        pageNumbers: { start: 1, formatType: "decimal" }
      }
    },
    headers: {
      default: new Header({ children: [new Paragraph({ alignment: AlignmentType.RIGHT, children: [new TextRun("Header Text")] })] })
    },
    footers: {
      default: new Footer({ children: [new Paragraph({
        alignment: AlignmentType.CENTER,
        children: [
          new TextRun("Page "),
          new TextRun({ children: [PageNumber.CURRENT] }),
          new TextRun(" of "),
          new TextRun({ children: [PageNumber.TOTAL_PAGES] })
        ]
      })] })
    },
    children: [/* 内容 */]
  }]
});
```

## 页眉页脚与分页

```javascript
// 分页符 - 必须放在 Paragraph 内
new Paragraph({ children: [new PageBreak()] }),

new Paragraph({
  pageBreakBefore: true,
  children: [new TextRun("This starts on a new page")]
})
```

⚠️ **绝对不要**单独创建 `new PageBreak()`，那会生成 Word 无法打开的无效 XML。

## Tabs（制表位）

```javascript
new Paragraph({
  tabStops: [
    { type: TabStopType.LEFT, position: TabStopPosition.MAX / 4 },
    { type: TabStopType.CENTER, position: TabStopPosition.MAX / 2 },
    { type: TabStopType.RIGHT, position: TabStopPosition.MAX * 3 / 4 }
  ],
  children: [new TextRun("Left\tCenter\tRight")]
})
```

## 图片与媒体

```javascript
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [new ImageRun({
    type: "png",                      // 必须指定类型
    data: fs.readFileSync("image.png"),
    transformation: { width: 200, height: 150, rotation: 0 },
    altText: { title: "Logo", description: "Company logo", name: "Name" } // 三个字段都要填
  })]
})
```

## 常用常量速查

- 下划线：`SINGLE`, `DOUBLE`, `WAVY`, `DASH`
- 边框：`SINGLE`, `DOUBLE`, `DASHED`, `DOTTED`
- 编号：`DECIMAL`, `UPPER_ROMAN`, `LOWER_LETTER`
- 制表位：`LEFT`, `CENTER`, `RIGHT`, `DECIMAL`
- 常见符号代码：`"2022"`(•)、`"00A9"`(©)、`"00AE"`(®)、`"2122"`(™)、`"00B0"`(°)、`"F070"`(✓)、`"F0FC"`(✗)

## 易错点与排障

- **分页符必须嵌在 Paragraph 中**，否则文档无法打开
- 表格底色需使用 `ShadingType.CLEAR`，避免 Word 渲染成黑底
- 所有长度用 DXA 表示；表格单元格至少包含一个 Paragraph；目录需使用 `HeadingLevel` 标题
- 建议使用自定义样式与 Arial 字体，保持专业层级
- 默认字体通过 `styles.default.document.run.font` 设置
- 表格需搭配 `columnWidths` + 单元格宽度；边框要设在 `TableCell`
- 项符号列表必须使用 `LevelFormat.BULLET`，不要使用字符串 `"bullet"`
- 不允许用 `\n` 换行；每行生成独立 Paragraph
- Paragraph 的 `children` 必须是 `TextRun` 等内容对象，不能直接写文本
- `ImageRun` 必须指定 `type`
- 目录依赖 `HeadingLevel`，不要同时指定其他自定义样式
- 为每个需要重新编号的段落使用新的编号 reference

阅读完以上内容后，即可按照规范安全生成 `.docx` 文档。
## 文本与格式

```javascript
// 重要：不要使用 \n 换行，必须使用多个 Paragraph 元素
// ❌ 错误：new TextRun("Line 1\nLine 2")
// ✅ 正确：new Paragraph({ children: [new TextRun("Line 1")] }), new Paragraph({ children: [new TextRun("Line 2")] })

// 带常见格式的示例
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 200, after: 200 },
  indent: { left: 720, right: 720 },
  children: [
    new TextRun({ text: "Bold", bold: true }),
    new TextRun({ text: "Italic", italics: true }),
    new TextRun({ text: "Underlined", underline: { type: UnderlineType.DOUBLE, color: "FF0000" } }),
    new TextRun({ text: "Colored", color: "FF0000", size: 28, font: "Arial" }), // Arial 默认字体
    new TextRun({ text: "Highlighted", highlight: "yellow" }),
    new TextRun({ text: "Strikethrough", strike: true }),
    new TextRun({ text: "x2", superScript: true }),
    new TextRun({ text: "H2O", subScript: true }),
    new TextRun({ text: "SMALL CAPS", smallCaps: true }),
    new SymbolRun({ char: "2022", font: "Symbol" }), // 项符号 •
    new SymbolRun({ char: "00A9", font: "Arial" })   // 版权符号 ©，使用 Arial
  ]
})
```

## 样式与专业级排版

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 默认 12pt
    paragraphStyles: [
      // 重写内置 Title 样式
      { id: "Title", name: "Title", basedOn: "Normal",
        run: { size: 56, bold: true, color: "000000", font: "Arial" },
        paragraph: { spacing: { before: 240, after: 120 }, alignment: AlignmentType.CENTER } },
      // 关键：使用内置标题的原始 ID 才能覆盖
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 32, bold: true, color: "000000", font: "Arial" }, // 16pt
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } }, // TOC 需要 outlineLevel
      { id: "Heading2", name: "Heading 2", basedOn: "Normal", next: "Normal", quickFormat: true,
        run: { size: 28, bold: true, color: "000000", font: "Arial" }, // 14pt
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 } },
      // 自定义样式使用自定义 ID
      { id: "myStyle", name: "My Style", basedOn: "Normal",
        run: { size: 28, bold: true, color: "000000" },
        paragraph: { spacing: { after: 120 }, alignment: AlignmentType.CENTER } }
    ],
    characterStyles: [{ id: "myCharStyle", name: "My Char Style",
      run: { color: "FF0000", bold: true, underline: { type: UnderlineType.SINGLE } } }]
  },
  sections: [{
    properties: { page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } } },
    children: [
      new Paragraph({ heading: HeadingLevel.TITLE, children: [new TextRun("Document Title")] }),
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("Heading 1")] }),
      new Paragraph({ style: "myStyle", children: [new TextRun("Custom paragraph style")] }),
      new Paragraph({ children: [
        new TextRun("Normal with "),
        new TextRun({ text: "custom char style", style: "myCharStyle" })
      ]})
    ]
  }]
});
```

**专业字体组合：**
- **Arial（标题）+ Arial（正文）**：最通用、简洁专业
- **Times New Roman（标题）+ Arial（正文）**：经典衬线 + 现代无衬线
- **Georgia（标题）+ Verdana（正文）**：屏幕阅读友好、对比优雅

**核心排版原则：**
- 使用内置样式原始 ID（例如 “Heading1”、“Heading2”）覆盖默认样式
- `HeadingLevel.HEADING_1` 对应 “Heading1”，以此类推
- 设置 `outlineLevel`（H1=0、H2=1…）以确保目录正确
- 尽量使用自定义样式而非逐段手动格式
- 通过 `styles.default.document.run.font` 设置默认字体（推荐 Arial）
- 建立明确的视觉层级：标题字号 > 小标题 > 正文
- 使用段前/段后的 `spacing` 控制段距
- 保持配色克制，建议标题和正文以黑色或灰度为主
- 标准页边距：`1440 = 1 inch`

## 列表（务必使用真正的列表配置）

```javascript
// 使用 numbering 配置，而非手写 unicode
const doc = new Document({
  numbering: {
    config: [
      { reference: "bullet-list",
        levels: [{ level: 0, format: LevelFormat.BULLET, text: "•", alignment: AlignmentType.LEFT,
          style: { paragraph: { indent: { left: 720, hanging: 360 } } } }] },
      { reference: "first-numbered-list",
        levels: [{ level: 0, format: LevelFormat.DECIMAL, text: "%1.", alignment: AlignmentType.LEFT,
