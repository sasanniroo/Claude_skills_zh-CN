# PDF 高阶处理参考

> 本文补充主技能文档未涵盖的高级功能、脚本示例与第三方库。涉及 Python、JavaScript 以及命令行工具的综合使用。

---

## pypdfium2（Apache/BSD）

Chromium PDFium 的 Python 封装，适合高性能渲染/截图，可替代 PyMuPDF。

### 渲染为图片
```python
import pypdfium2 as pdfium
from PIL import Image

pdf = pdfium.PdfDocument("document.pdf")
page = pdf[0]
bitmap = page.render(scale=2.0, rotation=0)
img = bitmap.to_pil()
img.save("page_1.png", "PNG")

for i, page in enumerate(pdf):
    bitmap = page.render(scale=1.5)
    img = bitmap.to_pil()
    img.save(f"page_{i+1}.jpg", "JPEG", quality=90)
```

### 提取文本
```python
pdf = pdfium.PdfDocument("document.pdf")
for i, page in enumerate(pdf):
    text = page.get_text()
    print(f"Page {i+1} text length: {len(text)} chars")
```

---

## JavaScript 库

### pdf-lib（MIT）

- 读写、创建复杂 PDF  
- 可嵌入字体、绘制图形、合并拆分页面

#### 加载并修改 PDF
```javascript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

async function manipulatePDF() {
  const existingPdfBytes = fs.readFileSync('input.pdf');
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  const pageCount = pdfDoc.getPageCount();
  console.log(`Document has ${pageCount} pages`);

  const newPage = pdfDoc.addPage([600, 400]);
  newPage.drawText('Added by pdf-lib', { x: 100, y: 300, size: 16 });

  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync('modified.pdf', pdfBytes);
}
```

#### 从零创建发票等复杂排版
```javascript
import { PDFDocument, rgb, StandardFonts } from 'pdf-lib';
import fs from 'fs';

async function createPDF() {
  const pdfDoc = await PDFDocument.create();
  const helveticaFont = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const helveticaBold = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
  const page = pdfDoc.addPage([595, 842]);
  const { width, height } = page.getSize();

  page.drawText('Invoice #12345', { x: 50, y: height - 50, size: 18, font: helveticaBold, color: rgb(0.2, 0.2, 0.8) });
  page.drawRectangle({ x: 40, y: height - 100, width: width - 80, height: 30, color: rgb(0.9, 0.9, 0.9) });
  ...
  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync('created.pdf', pdfBytes);
}
```

#### 合并/拆分
```javascript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

async function mergePDFs() {
  const mergedPdf = await PDFDocument.create();
  const pdf1 = await PDFDocument.load(fs.readFileSync('doc1.pdf'));
  const pdf2 = await PDFDocument.load(fs.readFileSync('doc2.pdf'));

  (await mergedPdf.copyPages(pdf1, pdf1.getPageIndices())).forEach(p => mergedPdf.addPage(p));
  (await mergedPdf.copyPages(pdf2, [0, 2, 4])).forEach(p => mergedPdf.addPage(p));

  fs.writeFileSync('merged.pdf', await mergedPdf.save());
}
```

### pdfjs-dist（Apache）

用于浏览器端渲染/解析 PDF。

```javascript
import * as pdfjsLib from 'pdfjs-dist';
pdfjsLib.GlobalWorkerOptions.workerSrc = './pdf.worker.js';

async function renderPDF() {
  const pdf = await pdfjsLib.getDocument('document.pdf').promise;
  const page = await pdf.getPage(1);
  const viewport = page.getViewport({ scale: 1.5 });
  ...
}
```

#### 获取文本坐标
```javascript
const textContent = await page.getTextContent();
const textWithCoords = textContent.items.map(item => ({
  text: item.str,
  x: item.transform[4],
  y: item.transform[5],
  width: item.width,
  height: item.height
}));
```

#### 读取批注/表单
```javascript
const annotations = await page.getAnnotations();
annotations.forEach(annotation => {
  console.log(annotation.subtype, annotation.contents, annotation.rect);
});
```

---

## 命令行工具

### poppler-utils 进阶用法

- `pdftotext -bbox-layout`：带坐标的文字提取  
- `pdftoppm`：高分辨率图片转换  
- `pdfimages -all`：提取原始嵌入图片

### qpdf 进阶用法

- 拆分页：`qpdf --split-pages=3 input.pdf output_%02d.pdf`  
- 复杂抽取：`qpdf input.pdf --pages input.pdf 1,3-5,8 -- out.pdf`  
- 线性化优化、清理、修复：`qpdf --linearize` / `--optimize-level=all` / `--fix-qdf`
- 加密解密：`qpdf --encrypt user owner 256 --print=none -- modify=none -- input.pdf secure.pdf`

---

## Python 进阶技巧

### pdfplumber 精确坐标
```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    page = pdf.pages[0]
    for char in page.chars[:10]:
        print(char['text'], char['x0'], char['y0'])
    bbox_text = page.within_bbox((100, 100, 400, 200)).extract_text()
```

### 高级表格解析
```python
table_settings = {
  "vertical_strategy": "lines",
  "horizontal_strategy": "lines",
  "snap_tolerance": 3,
  "intersection_tolerance": 15
}
tables = page.extract_tables(table_settings)
page.to_image(resolution=150).save("debug_layout.png")
```

### reportlab 生成报告
```python
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib import colors

doc = SimpleDocTemplate("report.pdf")
elements = [Paragraph("Quarterly Sales Report", getSampleStyleSheet()['Title'])]
table = Table(data)
table.setStyle(TableStyle([...]))
elements.append(table)
doc.build(elements)
```

---

## 综合工作流示例

### 提取图像
- 快速方案：`pdfimages -all document.pdf images/img`
- 定制方案：pypdfium2 渲染 → NumPy/OpenCV 识别 → PIL 裁剪

### 批量处理
- 遍历目录下 PDF，批量合并或提取文本  
- 使用 `logging` 记录异常，避免单文件出错中断整个流程

### 高级裁切
```python
reader = PdfReader("input.pdf")
page = reader.pages[0]
page.mediabox.left = 50
page.mediabox.right = 550
...
```

---

## 性能优化建议

1. **大文件**：分块处理、逐页流式读取。  
2. **文字提取**：`pdftotext -bbox-layout` 最快，复杂结构用 pdfplumber。  
3. **图片提取**：优先 `pdfimages`，需要预览可低分辨率渲染。  
4. **表单填充**：JavaScript 端推荐 pdf-lib，Python 推荐提前校验字段。  
5. **内存管理**：分批写入临时文件，释放对象。

---

## 常见问题排障

- **加密 PDF**：`PdfReader.is_encrypted` → `reader.decrypt(password)`  
- **损坏文件**：`qpdf --check` / `--replace-input` 修复  
- **扫描件文字**：结合 `pdf2image + pytesseract` 做 OCR  
- **提取失败**：尝试不同工具组合（pypdfium2、pdfplumber、qpdf）

---

## 许可证速览

| 库/工具        | License |
|----------------|---------|
| pypdf          | BSD     |
| pdfplumber     | MIT     |
| pypdfium2      | Apache/BSD |
| reportlab      | BSD     |
| poppler-utils  | GPL-2   |
| qpdf           | Apache  |
| pdf-lib        | MIT     |
| pdfjs-dist     | Apache  |

---

掌握上述工具和技巧，可以涵盖 PDF 渲染、表单处理、结构化提取、合并拆分、优化修复等复杂需求。可根据场景自由组合脚本与库，在自动化流程中灵活运用。
