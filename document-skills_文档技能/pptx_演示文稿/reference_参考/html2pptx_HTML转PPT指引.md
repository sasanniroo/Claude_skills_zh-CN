# HTML 转 PowerPoint 全流程指南

结合 `html2pptx.js` 与 PptxGenJS，将 HTML 幻灯片精准转换为 `.pptx` 演示文稿。

---

## 目录
1. HTML 幻灯片规范  
2. `html2pptx` 调用方法  
3. PptxGenJS 进阶用法（文本、图像、图表、表格）

---

## 1. HTML 幻灯片规范

### 布局尺寸
- **16:9**（默认）：`body` 尺寸 720pt × 405pt  
- **4:3**：720pt × 540pt  
- **16:10**：720pt × 450pt  
PptxGenJS 的 `pptx.layout` 必须与 HTML 尺寸匹配。

### 支持的元素
- 文本标签：`<p>`, `<h1>`~`<h6>`
- 列表：`<ul>`, `<ol>`（禁止手写 •、- 等）
- 内联格式：`<b>`, `<strong>`, `<i>`, `<em>`, `<u>`, `<span style="...">`
- 换行：`<br>`
- 形状：带背景/边框的 `<div>`
- 图片：`<img>`
- 预留图表区域：`class="placeholder"`（返回 `{ id, x, y, w, h }`）

**核心规则：** 所有文本内容必须置于 `<p>`／`<h*>`／`<ul>/<ol>` 内，否则转换时会被忽略。

### 字体与样式
- 仅使用 Web 安全字体：`Arial`、`Helvetica`、`Times New Roman`、`Georgia`、`Courier New`、`Verdana`、`Tahoma`、`Trebuchet MS`、`Impact`、`Comic Sans MS`
- 支持的行内样式：`font-weight`、`font-style`、`text-decoration`、`color`
- 颜色须为 `#RRGGBB`
- 推荐 `body` 使用 `display:flex` 防止溢出检测失真

### 形状（只作用于 `<div>`）
- 背景色：`background` / `background-color`
- 边框：`border` / 单边 `border-left` 等
- 圆角：`border-radius`
- 阴影：`box-shadow`（仅支持外阴影）
- 文本标签上设置背景/边框/阴影会被忽略

### 渐变与图标
- CSS 渐变不会转换，必须先使用 Sharp 等工具将 SVG 渐变/图标栅格化为 PNG，再通过 `<img>` 引用。

---

## 2. 使用 `html2pptx` 库

### 依赖
已预装：`pptxgenjs`, `playwright`, `sharp`。

### 基本调用
```javascript
const pptxgen = require('pptxgenjs');
const html2pptx = require('./html2pptx');

const pptx = new pptxgen();
pptx.layout = 'LAYOUT_16x9';

const { slide, placeholders } = await html2pptx('slide1.html', pptx);

// 使用 placeholder 插入图表
if (placeholders.length > 0) {
  slide.addChart(pptx.charts.LINE, chartData, placeholders[0]);
}

await pptx.writeFile('output.pptx');
```

### API
```javascript
await html2pptx(htmlFile, pres, options?)
```

- `htmlFile`：HTML 文件路径
- `pres`：PptxGenJS 演示文稿实例
- `options.tmpDir`：临时目录（默认 `/tmp`）
- `options.slide`：复用已有 slide

返回值包含 `slide` 与 `placeholders` 列表。

### 自动校验
- HTML 尺寸与布局不匹配  
- 内容溢出 `body` 边界  
- 文本节点错误（没有文本标签、使用渐变等）  
- 在文本标签上使用背景/边框/阴影等不支持样式  
所有错误会一次性汇报，方便整体修复。

---

## 3. PptxGenJS 进阶用法

### 颜色规则
- Hex 颜色不得带 `#`，否则会生成损坏文件：`color: "FF0000"`

### 图片
1. 读取原图尺寸计算宽高比
2. 设置 `x`, `y`, `w`, `h`，确保不失真

```javascript
slide.addImage({ path: "chart.png", x, y: 1.5, w, h });
```

### 文本
```javascript
slide.addText([
  { text: "Bold ", options: { bold: true } },
  { text: "Italic ", options: { italic: true } },
  { text: "Normal" }
], { x: 1, y: 2, w: 8, h: 1 });
```

### 形状
```javascript
slide.addShape(pptx.shapes.RECTANGLE, {
  x: 1, y: 1, w: 3, h: 2,
  fill: { color: "4472C4" },
  line: { color: "000000", width: 2 }
});
```

### 图表
- 单系列：`labels`/`values` 在同一条数据对象中
- 多系列：每个系列包含自己的 `labels` 与 `values`
- 需设置坐标轴标题 `catAxisTitle` / `valAxisTitle`
- 时间序列根据时长选择日/月/年粒度

```javascript
const { placeholders } = await html2pptx('slide.html', pptx);

slide.addChart(pptx.charts.BAR, [{
  name: "Sales 2024",
  labels: ["Q1", "Q2", "Q3", "Q4"],
  values: [4500, 5500, 6200, 7100]
}], {
  ...placeholders[0],
  showTitle: true,
  title: 'Quarterly Sales',
  showCatAxisTitle: true,
  catAxisTitle: 'Quarter',
  showValAxisTitle: true,
  valAxisTitle: 'Sales ($000s)',
  chartColors: ["4472C4"]
});
```

**Scatter 图**：第一组数据为 X 值，其余为 Y 值。  
**Pie 图**：必须使用单数据系列。  
**Chart 颜色**：提供无 `#` 的 hex 值；多系列图需确保颜色对比明显。

### 表格
```javascript
slide.addTable([
  ["Header 1", "Header 2"],
  ["Row 1", "Row 1"],
  ["Row 2", "Row 2"]
], {
  x: 0.5, y: 1, w: 9, h: 3,
  border: { pt: 1, color: "999999" },
  fill: { color: "F1F1F1" },
  align: "center",
  valign: "middle",
  fontSize: 14
});
```

可使用 cell `options` 设置单元格填充、合并单元格（`colspan`）、字体等。

---

## 实战提示

1. **先在 HTML 中确认版面**，通过浏览器预览确保无滚动溢出。  
2. **渐变、图标等复杂视觉元素需提前栅格化**，再用 `<img>` 引用。  
3. **生成 PPT 后用 PowerPoint 打开自测**：检查字体、颜色是否正常，图表和表格是否与占位区域对齐。  
4. **善用 placeholders**：HTML 中的 `.placeholder` 会返回准确位置，便于后续插入图表。  
5. **若报错**，根据 html2pptx 的综合错误信息逐项修复。

按照上述规范，即可实现高保真 HTML → PowerPoint 转换，并在 PptxGenJS 中添加动态图表与数据可视化内容。
