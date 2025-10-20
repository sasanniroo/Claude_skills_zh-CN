# PowerPoint OOXML 结构速览

> 适用于直接编辑 `ppt/slides/slideN.xml` 等文件时参考。请务必在操作前完整阅读，错误的节点顺序或属性会导致 PowerPoint 无法打开生成的文件。

---

## 1. 基础规范

- **命名空间**：幻灯片主文件使用 `p:`（PresentationML）、文本 `a:`（DrawingML）、关系 `r:`。  
- **文本节点顺序**：`<p:txBody>` 内部必须依次为 `<a:bodyPr>`、`<a:lstStyle>`、`<a:p>`。  
- **空白字符**：若 `<a:t>` 含首尾空格，需设 `xml:space="preserve"`。  
- **Unicode**：在 XML 中使用实体，比如直引号写成 `&#8220;`。  
- **图片**：文件放在 `ppt/media/`，并在 `ppt/slides/_rels/slideN.xml.rels` 中建立 `<Relationship>`。  
- **dirty 属性**：建议在 `<a:rPr>` 与 `<a:endParaRPr>` 添加 `dirty="0"`，表示格式已同步。

---

## 2. 幻灯片骨架

```xml
<p:sld>
  <p:cSld>
    <p:spTree>
      <p:nvGrpSpPr>…</p:nvGrpSpPr>
      <p:grpSpPr>…</p:grpSpPr>
      <!-- 形状/文本放这里 -->
    </p:spTree>
  </p:cSld>
</p:sld>
```

### 文本框（标题形状示例）

```xml
<p:sp>
  <p:nvSpPr>
    <p:cNvPr id="2" name="Title"/>
    <p:cNvSpPr><a:spLocks noGrp="1"/></p:cNvSpPr>
    <p:nvPr><p:ph type="ctrTitle"/></p:nvPr>
  </p:nvSpPr>
  <p:spPr>
    <a:xfrm><a:off x="838200" y="365125"/><a:ext cx="7772400" cy="1470025"/></a:xfrm>
  </p:spPr>
  <p:txBody>
    <a:bodyPr/><a:lstStyle/>
    <a:p><a:r><a:t>Slide Title</a:t></a:r></a:p>
  </p:txBody>
</p:sp>
```

---

## 3. 文本与段落格式

### 常用属性

```xml
<a:r>
  <a:rPr b="1" i="1" u="sng" lang="en-US" sz="2400" dirty="0">
    <a:solidFill><a:srgbClr val="FF0000"/></a:solidFill>
  </a:rPr>
  <a:t xml:space="preserve">Bold Italic Underlined</a:t>
</a:r>
```

- `b="1"` 粗体；`i="1"` 斜体；`u="sng"` 下划线；`sz="2400"` 表示 24pt。  
- 高亮：`<a:highlight><a:srgbClr val="FFFF00"/></a:highlight>`。  
- 段落层级与列表通过 `<a:pPr>` 控制：

```xml
<a:p>
  <a:pPr lvl="0"><a:buChar char="•"/></a:pPr>
  <a:r><a:t>普通项目符号</a:t></a:r>
</a:p>
```

数字列表：`<a:buAutoNum type="arabicPeriod"/>`；多级列表调整 `lvl=1,2…`。

---

## 4. 使用几何图形

```xml
<p:sp>
  <p:nvSpPr><p:cNvPr id="3" name="Rectangle"/></p:nvSpPr>
  <p:spPr>
    <a:xfrm><a:off x="1000000" y="1000000"/><a:ext cx="3000000" cy="2000000"/></a:xfrm>
    <a:prstGeom prst="rect"><a:avLst/></a:prstGeom>
    <a:solidFill><a:srgbClr val="FF0000"/></a:solidFill>
    <a:ln w="25400"><a:solidFill><a:srgbClr val="000000"/></a:solidFill></a:ln>
  </p:spPr>
</p:sp>
```

- 圆角矩形：`prst="roundRect"`  
- 椭圆/圆：`prst="ellipse"`  
- 线条：使用 `<p:cxnSp>` 或 `<p:sp>` 搭配 `prstGeom` `line`。

---

## 5. 图片与媒体

1. 将文件放入 `ppt/media/image1.png`  
2. 在对应 slide 的 `.rels` 文件创建关系：

```xml
<Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="../media/image1.png"/>
```

3. 在幻灯片 XML 中引用 `<p:pic>`，并设置 `<a:blip r:embed="rId5"/>`。

---

## 6. 图表（Chart）

图表由三部分组成：
1. `ppt/charts/chart1.xml` 描述数据与样式  
2. `ppt/charts/_rels/chart1.xml.rels` 指向数据源（`.xml` 或 `.xlsx`）  
3. 幻灯片内 `<p:graphicFrame>` 引用图表

创建流程概要：
1. 在 `chart1.xml` 指定 `<c:barChart> / <c:lineChart>...` 及 `<c:ser>` 数据  
2. `chart1.xml.rels` 建立指向 `../embeddings/` 数据或 `../media/` 图片  
3. 在 slide 中添加：
```xml
<p:graphicFrame>
  <p:nvGraphicFramePr>...</p:nvGraphicFramePr>
  <p:xfrm>...</p:xfrm>
  <a:graphic>
    <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/chart">
      <c:chart r:id="rId6"/>
    </a:graphicData>
  </a:graphic>
</p:graphicFrame>
```

---

## 7. 表格

```xml
<p:graphicFrame>
  <p:xfrm>...</p:xfrm>
  <a:graphic>
    <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/table">
      <a:tbl>
        <a:tblPr firstRow="1"/>
        <a:tblGrid>
          <a:gridCol w="3048000"/>
        </a:tblGrid>
        <a:tr h="370840">
          <a:tc>
            <a:txBody>...</a:txBody>
            <a:tcPr><a:solidFill>...</a:solidFill></a:tcPr>
          </a:tc>
        </a:tr>
      </a:tbl>
    </a:graphicData>
  </a:graphic>
</p:graphicFrame>
```

- 表头启用 `firstRow="1"`  
- 单元格格式在 `<a:tcPr>` 内设置填充、边框等。

---

## 8. 幻灯片母版与版式

- 母版：`ppt/slideMasters/slideMaster1.xml`  
- 版式：`ppt/slideLayouts/slideLayout1.xml`（`<p:ph>` 定义占位符类型/索引）  
- 幻灯片引用版式：
```xml
<p:slideLayoutIdList>
  <p:slideLayoutId id="257" r:id="rId1"/>
</p:slideLayoutIdList>
```

---

## 9. 关系文件与打包

- 每个 `slideN.xml` 对应 `slideN.xml.rels`，用于图片、图表、音频等引用。  
- 在根的 `[Content_Types].xml` 中注册新资源类型。  
- 编辑完成后使用 `pack.py` 重新打包，保持原目录结构。

---

## 10. 常见错误与排障

| 问题 | 可能原因 |
|------|----------|
| PowerPoint 提示文件损坏 | 节点顺序错误、缺少 namespace、关系遗漏 |
| 文本丢失 | 未使用 `<a:t>`，或忘记 `xml:space="preserve"` |
| 列表格式异常 | `lvl` / `<a:bu*>` 配置错误或缺少 `a:pPr` |
| 图片不显示 | 未更新 `.rels` 或路径写错 |
| 图表报错 | 缺少 chart 关系或数据范围不合法 |

建议使用 `unzip` 检查目录结构，或让 PowerPoint 自带的“检查问题”功能验证。

---

## 参考

- ECMA-376 & ISO/IEC 29500 标准  
- Microsoft Open Specifications（Office DrawingML）  
- 现有 PPTX 文件 → 解包 → 对比模板

掌握以上要点，便可安全地在 XML 层面修改幻灯片内容，并与自动化脚本结合生成高质量 PowerPoint。
