---
name: pptx
description: “面向 PowerPoint 演示文稿（.pptx）的创建、编辑与分析”。当 Claude 需要创建新演示文稿、修改现有内容、处理版式、添加批注或演讲者备注等与幻灯相关的任务时，请使用本技能。
license: 专有许可。完整条款见 LICENSE.txt
---

# PPTX 文档的创建、编辑与分析

## 概览

用户可能要求你创建、编辑或分析 .pptx 文件。.pptx 实际上是包含 XML 及资源文件的 ZIP 压缩包，可通过读取或修改其内容完成操作。针对不同任务，需要选择不同的工具与流程。

## 阅读与分析内容

### 文本抽取
若仅需阅读演示文稿中的文本，可先转换为 Markdown：

```bash
# 将演示文稿转换为 markdown
python -m markitdown path-to-file.pptx
```

### 原始 XML 访问
当需要处理批注、演讲者备注、幻灯版式、动画、设计元素或复杂格式时，必须访问原始 XML。在此类场景下应先解包演示文稿并阅读其中 XML。

#### 解包文件
`python ooxml/scripts/unpack.py <office_file> <output_dir>`

**注意**：脚本位于项目根目录下的 `skills/pptx/ooxml/scripts/unpack.py`。若路径下未找到该脚本，可使用 `find . -name "unpack.py"` 搜索。

#### 核心文件结构
- `ppt/presentation.xml` —— 演示文稿的主元数据与幻灯引用
- `ppt/slides/slide{N}.xml` —— 每张幻灯的内容（slide1.xml、slide2.xml 等）
- `ppt/notesSlides/notesSlide{N}.xml` —— 各幻灯的演讲者备注
- `ppt/comments/modernComment_*.xml` —— 特定幻灯的批注
- `ppt/slideLayouts/` —— 幻灯版式模板
- `ppt/slideMasters/` —— 母版幻灯模板
- `ppt/theme/` —— 主题与样式信息
- `ppt/media/` —— 图片及其他媒体文件

#### 字体与配色抽取
**当需要模仿现有设计时**：务必先分析演示文稿的字体与颜色，可按以下步骤：
1. **读取主题文件**：在 `ppt/theme/theme1.xml` 中查找 `<a:clrScheme>`（颜色）与 `<a:fontScheme>`（字体）
2. **抽样幻灯内容**：查看 `ppt/slides/slide1.xml` 中的 `<a:rPr>`（实际字体）与颜色
3. **检索模式**：通过 grep 在所有 XML 中查找 `<a:solidFill>`、`<a:srgbClr>` 等颜色以及字体引用

## 创建全新 PowerPoint（无模板）

若要从零开始创建演示文稿，请使用 **html2pptx** 工作流，将 HTML 幻灯精确转换为 PowerPoint。

### 设计原则

**关键步骤**：在编写代码前先分析内容，并选择合适的设计元素：
1. **理解主题**：演示文稿的主题是什么？需要营造怎样的语气、行业感或氛围？
2. **检查品牌**：若提及公司/组织，应考虑其品牌色与视觉识别
3. **配色匹配内容**：选择与主题相符的配色
4. **说明设计思路**：在写代码前解释你的设计选择

**必要要求**：
- ✅ 在写代码前，先陈述基于内容的设计策略
- ✅ 仅使用 Web 安全字体：Arial、Helvetica、Times New Roman、Georgia、Courier New、Verdana、Tahoma、Trebuchet MS、Impact
- ✅ 通过字号、粗细、颜色建立清晰的视觉层级
- ✅ 确保可读性：强对比、合适字号、整洁对齐
- ✅ 保持一致性：重复使用版式、间距与视觉语言

#### 配色选择

**创意选择配色时：**
- **不要机械套用默认色**：考虑与主题真正契合的颜色
- **多角度思考**：主题、行业、情绪、节奏、受众、品牌识别
- **保持冒险精神**：医疗不一定非得是绿色，金融也不必都是海军蓝
- **构建色板**：选择 3-5 种互补颜色（主色 + 辅色 + 点缀色）
- **注意对比度**：确保文字在背景上清晰可读

**示例配色**（可借鉴、调整或自创）：

1. **Classic Blue**：深海军蓝 (#1C2833)、石板灰 (#2E4053)、银色 (#AAB7B8)、灰白 (#F4F6F6)
2. **Teal & Coral**：水鸭青 (#5EA8A7)、深青 (#277884)、珊瑚红 (#FE4447)、白色 (#FFFFFF)
3. **Bold Red**：红色 (#C0392B)、亮红 (#E74C3C)、橙色 (#F39C12)、黄色 (#F1C40F)、绿色 (#2ECC71)
4. **Warm Blush**：淡紫灰 (#A49393)、腮红色 (#EED6D3)、玫瑰色 (#E8B4B8)、米白 (#FAF7F2)
5. **Burgundy Luxury**：勃艮第 (#5D1D2E)、深红 (#951233)、铁锈橙 (#C15937)、金色 (#997929)
6. **Deep Purple & Emerald**：紫色 (#B165FB)、深蓝 (#181B24)、祖母绿 (#40695B)、白色 (#FFFFFF)
7. **Cream & Forest Green**：奶油色 (#FFE1C7)、森林绿 (#40695B)、白色 (#FCFCFC)
8. **Pink & Purple**：粉色 (#F8275B)、珊瑚 (#FF574A)、玫瑰 (#FF737D)、紫色 (#3D2F68)
9. **Lime & Plum**：柠檬绿 (#C5DE82)、李子紫 (#7C3A5F)、珊瑚 (#FD8C6E)、蓝灰 (#98ACB5)
10. **Black & Gold**：金色 (#BF9A4A)、黑色 (#000000)、米白 (#F4F6F6)
11. **Sage & Terracotta**：鼠尾草绿 (#87A96B)、砖粉 (#E07A5F)、奶油色 (#F4F1DE)、炭黑 (#2C2C2C)
12. **Charcoal & Red**：木炭色 (#292929)、红色 (#E33737)、淡灰 (#CCCBCB)
13. **Vibrant Orange**：橙色 (#F96D00)、淡灰 (#F2F2F2)、炭灰 (#222831)
14. **Forest Green**：黑色 (#191A19)、绿色 (#4E9F3D)、深绿 (#1E5128)、白色 (#FFFFFF)
15. **Retro Rainbow**：紫色 (#722880)、粉色 (#D72D51)、橙色 (#EB5C18)、琥珀色 (#F08800)、金色 (#DEB600)
16. **Vintage Earthy**：芥末黄 (#E3B448)、鼠尾草绿 (#CBD18F)、森林绿 (#3A6B35)、奶油色 (#F4F1DE)
17. **Coastal Rose**：旧玫瑰 (#AD7670)、褐灰 (#B49886)、蛋壳色 (#F3ECDC)、灰绿 (#BFD5BE)
18. **Orange & Turquoise**：淡橙 (#FC993E)、灰绿松石色 (#667C6F)、白色 (#FCFCFC)

#### 视觉细节选项

**几何图形：**
- 使用对角线分割取代水平分割
- 非对称列宽（30/70、40/60、25/75）
- 将标题文本旋转 90° 或 270°
- 图像使用圆形/六边形框
- 角落添加三角形点缀
- 通过重叠形状增加层次

**边框与框架：**
- 在单侧使用 10-20pt 的实色粗边
- 使用对比色形成双线边框
- 用角支架替代完整边框
- L 型边框（仅上 + 左或下 + 右）
- 在标题下方添加 3-5pt 的下划线

**字体处理：**
- 极端的字号对比（如 72pt 标题 vs 11pt 正文）
- 全大写标题并加大字间距
- 以超大显示字体标注章节编号
- 数据/技术内容使用等宽字体（Courier New）
- 信息密集时使用窄体字体（Arial Narrow）
- 使用描边文字强化重点

**图表与数据：**
- 单色调图表，关键数据使用点缀色
- 采用横向柱状图替代竖向
- 使用点图代替柱状图
- 减少或移除网格线
- 在图形上直接标注数据（避免图例）
- 使用超大数字突出核心指标

**版式创新：**
- 全屏图片 + 文字覆盖
- 侧边栏（占宽 20-30%）用于导航或背景信息
- 模块化网格布局（3×3、4×4）
- Z 型或 F 型内容流
- 彩色形状上悬浮文本框
- 类杂志风格的多栏排版

**背景处理：**
- 40-60% 的画面使用实色块
- 垂直或对角渐变
- 分割背景（双色，垂直或对角）
- 边缘延伸的色带
- 将留白作为设计元素

### 版式提示

**包含图表或表格的幻灯：**
- **首选双栏布局**：标题横跨全宽，下方左右分栏 —— 一栏放文本或要点，另一栏放图表/表格。可使用不等宽（如 40%/60%）的 flex 布局，提升可读性。
- **全屏布局**：让图表或表格占满整页，强调视觉冲击与可读性。
- **禁止垂直堆叠**：不要让图表/表格在单栏布局中位于文本下方，会严重影响阅读。

### 工作流
1. **必须完整阅读** [`html2pptx.md`](html2pptx.md)。**禁止只读部分内容**。该文件提供详细语法、编码规范与最佳实践，是创建演示文稿的前置要求。
2. 为每张幻灯创建尺寸正确的 HTML 文件（如 720pt × 405pt 适配 16:9）：
   - 所有文本使用 `<p>`、`<h1>`-`<h6>`、`<ul>`、`<ol>` 标签
   - 若需要图表/表格位置，使用 `class="placeholder"` 的区域（并用灰色背景标示）
   - **重点**：使用 Sharp 先将渐变与图标栅格化为 PNG，再在 HTML 中引用
   - **版式**：含图表/表格/图片的幻灯，应使用全屏布局或双栏布局
3. 创建并运行基于 [`html2pptx.js`](scripts/html2pptx.js) 的 JavaScript 文件，将 HTML 幻灯转换为 PowerPoint 并保存
   - 使用 `html2pptx()` 函数处理每个 HTML 文件
   - 通过 PptxGenJS API 向占位区添加图表或表格
   - 使用 `pptx.writeFile()` 保存最终演示文稿
4. **视觉验证**：生成缩略图，检查版式问题
   - 创建缩略图网格：`python scripts/thumbnail.py output.pptx workspace/thumbnails --cols 4`
   - 仔细检查缩略图，重点关注：
     - **文本裁切**：文字是否被标题栏、形状或幻灯边缘遮挡
     - **文本重叠**：文本是否互相冲突
     - **位置问题**：元素是否过于靠近边界或其他对象
     - **对比问题**：文字与背景是否形成足够对比
   - 如有问题，调整 HTML 的边距 / 间距 / 颜色，并重新生成
   - 重复以上步骤直至所有幻灯视觉正确

## 编辑现有 PowerPoint

当需要修改现有演示文稿时，必须操作 OOXML 原始文件：解包 .pptx → 编辑 XML → 再打包。

### 工作流
1. **必须完整阅读** [`ooxml.md`](ooxml.md)（约 500 行）。**禁止限定阅读范围**。在进行任何编辑之前，要充分理解 OOXML 结构与工作流程。
2. 解包演示文稿：`python ooxml/scripts/unpack.py <office_file> <output_dir>`
3. 编辑相关 XML 文件（主要是 `ppt/slides/slide{N}.xml` 等）
4. **关键**：每次修改后立即验证，修复所有问题再继续：`python ooxml/scripts/validate.py <dir> --original <file>`
5. 打包最终演示文稿：`python ooxml/scripts/pack.py <input_directory> <office_file>`

## 使用模板创建演示文稿

若需沿用现有模板的设计，应先复制、重排模板幻灯，再替换内容。

### 工作流
1. **提取模板文本并生成缩略图**：
   * 提取文本：`python -m markitdown template.pptx > template-content.md`
   * 完整阅读 `template-content.md`，理解模板内容
   * 生成缩略图：`python scripts/thumbnail.py template.pptx`
   * 详见下文 “创建缩略图网格”

2. **分析模板并建立清点文件**：
   * **视觉分析**：查看缩略图，理解版式、设计模式与结构
   * 在 `template-inventory.md` 中记录：
     ```markdown
     # Template Inventory Analysis
     **Total Slides: [count]**
     **IMPORTANT: Slides are 0-indexed (first slide = 0, last slide = count-1)**

     ## [Category Name]
     - Slide 0: [Layout code if available] - Description/purpose
     - Slide 1: [Layout code] - Description/purpose
     - Slide 2: [Layout code] - Description/purpose
     [... EVERY slide must be listed individually with its index ...]
     ```
   * 使用缩略图识别：
     - 版式模式（标题页、内容页、章节页等）
     - 图片占位的位置与数量
     - 各组幻灯之间的设计一致性
     - 视觉层级结构
   * 该清点文件是后续选择模板的必需材料

3. **基于模板清点创建演示文稿大纲**：
   * 参考第 2 步列出的模板
   * 首张幻灯选择引言/标题模板（通常在前几页）
   * 其余幻灯选择安全、以文本为主的版式
   * **关键：版式必须匹配实际内容结构**：
     - 单栏版式用于单一主题或统一叙述
     - 双栏版式仅在确有两项独立内容时使用
     - 三栏版式仅在确有三项内容时使用
     - 图文版式仅在手头已有图像可插入时使用
     - 引言版式必须用于真实引用（含引用人），代替强调文本的用途
     - 避免使用多余占位符的版式（内容不足会显得空洞）
     - 若有 2 条内容，不要勉强塞进 3 栏；若有 4 条以上，考虑拆分或使用列表
   * 在选择布局前，统计实际的内容项数量
   * 确认每个占位符都能填入有意义的内容
   * 保存 `outline.md`，记录内容与模板映射关系，例如：
     ```
     # Template slides to use (0-based indexing)
     # WARNING: Verify indices are within range! Template with 73 slides has indices 0-72
     # Mapping: slide numbers from outline -> template slide indices
     template_mapping = [
         0,   # Use slide 0 (Title/Cover)
         34,  # Use slide 34 (B1: Title and body)
         34,  # Use slide 34 again (duplicate for second B1)
         50,  # Use slide 50 (E1: Quote)
         54,  # Use slide 54 (F2: Closing + Text)
     ]
     ```

4. **使用 `rearrange.py` 复制、重排、删除幻灯**：
   * 执行：`
     python scripts/rearrange.py template.pptx working.pptx 0,34,34,50,52
     `
   * 脚本会自动完成复制、删除与重新排序
   * 幻灯编号为 0 起始，可重复引用以复制同一幻灯

5. **使用 `inventory.py` 提取所有文本**：
   * 运行：`python scripts/inventory.py working.pptx text-inventory.json`
   * 完整阅读 `text-inventory.json`，了解所有形状与属性
   * JSON 结构示例：
     ```json
      {
        "slide-0": {
          "shape-0": {
            "placeholder_type": "TITLE",
            "left": 1.5,
            "top": 2.0,
            "width": 7.5,
            "height": 1.2,
            "paragraphs": [
              {
                "text": "Paragraph text",
                "bullet": true,
                "level": 0,
                "alignment": "CENTER",
                "space_before": 10.0,
                "space_after": 6.0,
                "line_spacing": 22.4,
                "font_name": "Arial",
                "font_size": 14.0,
                "bold": true,
                "italic": false,
                "underline": false,
                "color": "FF0000"
              }
            ]
          }
        }
      }
     ```
   * 关键特点：
     - 幻灯命名为 `slide-0`、`slide-1`...
     - 形状按视觉顺序（从上到下、从左到右）命名为 `shape-0`...
     - `placeholder_type` 可能为 TITLE、CENTER_TITLE、SUBTITLE、BODY、OBJECT 或 null
     - 若提供，`default_font_size` 表示版式占位符的默认字号
     - 幻灯编号形状（SLIDE_NUMBER）会被自动过滤
     - 设置 `bullet: true` 时总会包含 `level`
     - `space_before`、`space_after`、`line_spacing` 以磅为单位，仅在非默认值时出现
     - 颜色可表示为 RGB（`color`）或主题色（`theme_color`）
     - 仅非默认属性会输出

6. **生成替换文本并保存为 JSON**
   - 先确认库存中存在哪些形状，仅引用真实存在的对象
   - **替换脚本会验证**：若引用不存在的幻灯或形状，会一次性报错
   - 替换脚本会使用 inventory.py 提供的形状信息
   - **自动清空**：所有形状的文本会被清除，除非在 JSON 中定义了 `paragraphs`
   - 为需要填充内容的形状添加 `paragraphs` 字段（不是 `replacement_paragraphs`）
   - 未在 JSON 中提到的形状会被清空
   - 带 `bullet: true` 的段落会自动左对齐，不要另设 `alignment`
   - 根据形状尺寸控制文本长度，避免溢出
   - **关键**：保留库存中的段落属性，不要只提供纯文本
   - **特别提醒**：当 `bullet: true` 时，不要在文本中加入项目符号（•、-、*），脚本会自动添加
   - **格式要求**：
     - 标题通常需要 `"bold": true`
     - 列表项需 `"bullet": true, "level": 0`
     - 对于特殊对齐方式，保留 `"alignment": "CENTER"`
     - 若字体不同于默认值，注明 `"font_size": 14.0`、`"font_name": "Lora"`
     - 颜色可用 `"color": "FF0000"` 或 `"theme_color": "DARK_1"`
     - 替换脚本需要完整的段落属性，而不是纯文本字符串
     - 若存在重叠形状，应优先选择 `default_font_size` 较大或 `placeholder_type` 更合适者
   - 将替换后的库存保存为 `replacement-text.json`
   - **警告**：不同版式的形状数量可能不同，创建替换文件前务必确认实际库存

   示例 `paragraphs` 字段：
   ```json
   "paragraphs": [
     {
       "text": "New presentation title text",
       "alignment": "CENTER",
       "bold": true
     },
     {
       "text": "Section Header",
       "bold": true
     },
     {
       "text": "First bullet point without bullet symbol",
       "bullet": true,
       "level": 0
     },
     {
       "text": "Red colored text",
       "color": "FF0000"
     },
     {
       "text": "Theme colored text",
       "theme_color": "DARK_1"
     },
     {
       "text": "Regular paragraph text without special formatting"
     }
   ]
   ```

   **未在替换 JSON 中列出的形状会被自动清空**：
   ```json
   {
     "slide-0": {
       "shape-0": {
         "paragraphs": [...]
       }
       // inventory 中的 shape-1 与 shape-2 将被清空
     }
   }
   ```

   **常见排版模式**：
   - 标题页：文本加粗，常居中
   - 幻灯内章节标题：加粗
   - 项目符号列表：每项 `"bullet": true, "level": 0`
   - 正文：通常无需额外属性
   - 引言：可能具备特殊对齐或字体

7. **使用 `replace.py` 应用替换**
   ```bash
   python scripts/replace.py working.pptx replacement-text.json output.pptx
   ```

   脚本会：
   - 先通过 inventory.py 提取所有文本形状
   - 验证替换 JSON 中引用的形状是否存在
   - 清空所有被识别出的形状
   - 仅对带 `paragraphs` 的形状应用新的文本
   - 根据 JSON 中的段落属性保留格式
   - 自动处理项目符号、对齐、字体与颜色
   - 输出更新后的演示文稿

   示例报错：
   ```
   ERROR: Invalid shapes in replacement JSON:
     - Shape 'shape-99' not found on 'slide-0'. Available shapes: shape-0, shape-1, shape-4
     - Slide 'slide-999' not found in inventory
   ```

   ```
   ERROR: Replacement text made overflow worse in these shapes:
     - slide-0/shape-2: overflow worsened by 1.25" (was 0.00", now 1.25")
   ```

## 创建缩略图网格

要快速分析与参考幻灯内容，可生成缩略图网格：

```bash
python scripts/thumbnail.py template.pptx [output_prefix]
```

**功能**：
- 输出 `thumbnails.jpg`（或 `thumbnails-1.jpg`、`thumbnails-2.jpg` 等）
- 默认 5 列，每张网格最多 30 张幻灯（5×6）
- 自定义前缀：`python scripts/thumbnail.py template.pptx my-grid`
  - 若需输出到特定目录，前缀需包含路径（如 `workspace/my-grid`）
- 调整列数：`--cols 4`（范围 3-6，会影响每网格的幻灯数量）
  - 3 列：12 页；4 列：20 页；5 列：30 页；6 列：42 页
- 幻灯采用 0 基索引（Slide 0、Slide 1…）

**使用场景**：
- 模板分析：快速了解版式与设计模式
- 内容审查：全局预览演示文稿
- 导航参考：凭视觉定位特定幻灯
- 质量检查：确认所有幻灯格式正确

**示例**：
```bash
# 基本用法
python scripts/thumbnail.py presentation.pptx

# 自定义前缀 + 列数
python scripts/thumbnail.py template.pptx analysis --cols 4
```

## 幻灯转换为图片

若需以图像形式分析幻灯，可按以下两步操作：

1. **PPTX 转 PDF**：
   ```bash
   soffice --headless --convert-to pdf template.pptx
   ```

2. **PDF 转 JPEG**：
   ```bash
   pdftoppm -jpeg -r 150 template.pdf slide
   ```
   将生成 `slide-1.jpg`、`slide-2.jpg` 等文件。

常用选项：
- `-r 150`：分辨率 150 DPI（可根据质量与大小权衡调整）
- `-jpeg`：输出 JPEG（若需 PNG，请改用 `-png`）
- `-f N`：起始页（如 `-f 2` 表示从第 2 页开始）
- `-l N`：结束页（如 `-l 5` 表示至第 5 页结束）
- `slide`：输出文件前缀

指定范围示例：
```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 template.pdf slide  # 仅转换第 2-5 页
```

## 代码风格指南
**重要**：编写与 PPTX 操作相关的代码时：
- 代码要简洁
- 避免冗长变量名与重复操作
- 避免不必要的 print 输出

## 依赖项

需具备以下依赖（若尚未安装，请补齐）：

- **markitdown**：`pip install "markitdown[pptx]"`（用于文本抽取）
- **pptxgenjs**：`npm install -g pptxgenjs`（配合 html2pptx 创建演示文稿）
- **playwright**：`npm install -g playwright`（html2pptx 渲染 HTML）
- **react-icons**：`npm install -g react-icons react react-dom`（图标）
- **sharp**：`npm install -g sharp`（栅格化 SVG 与图像处理）
- **LibreOffice**：`sudo apt-get install libreoffice`（用于 PDF 转换）
- **Poppler**：`sudo apt-get install poppler-utils`（pdftoppm 转换 PDF 为图片）
- **defusedxml**：`pip install defusedxml`（安全解析 XML）
