**重要：以下步骤必须按顺序执行，不要跳跃。**

当需要填写 PDF 表单时，先判断是否包含可填写字段。  
1. 在本目录下运行：`python scripts/check_fillable_fields <file.pdf>`  
2. 根据输出结果分别遵循“可填写字段”或“不可填写字段”的指引。

---

# 一、可填写字段（fillable form fields）

1. 运行 `extract_form_field_info.py`  
   ```bash
   python scripts/extract_form_field_info.py <input.pdf> <field_info.json>
   ```  
   该脚本会输出每个字段的基础信息（字段 ID、页码、矩形范围、类型等）。

2. 将 PDF 转换为 PNG 图像，辅助判读字段含义：  
   ```bash
   python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>
   ```  
   对照图像，结合 `rect` 信息（注意 PDF 坐标原点在左下角），确定各字段用途。

3. 编写 `field_values.json`，填写所需的字段内容：  
   ```json
   [
     {
       "field_id": "last_name",
       "description": "用户的姓氏",
       "page": 1,
       "value": "Simpson"
     },
     {
       "field_id": "Checkbox12",
       "description": "大于 18 岁的复选框",
       "page": 1,
       "value": "/On"     // 对应 checkbox 的 checked_value；单选按钮使用 radio_options 的 value
     }
   ]
   ```

4. 使用 `fill_fillable_fields.py` 填表：  
   ```bash
   python scripts/fill_fillable_fields.py <input.pdf> <field_values.json> <output.pdf>
   ```  
   如果脚本报错，说明字段 ID 或值不匹配，请修正后重试。

---

# 二、不可填写字段（non-fillable form）

需要人工识别文本位置并生成注释，以模拟手动填写。务必完成以下四步：

## Step 1：视觉分析（必做）
- 将 PDF 转换为 PNG：`python scripts/convert_pdf_to_images.py <file.pdf> <output_dir>`  
- 仔细查看每一页，识别所有需要填写的区域；为每个字段确定两个独立的矩形框：
  - **label_bounding_box**：字段标签区域
  - **entry_bounding_box**：实际输入区域（不得与 label 重叠）
- 对于常见结构（框内文字、横线、上下排列、复选框）按照说明定位。

## Step 2：创建 `fields.json` 与校验图（必做）

`fields.json` 示例：
```json
{
  "pages": [
    { "page_number": 1, "image_width": 1700, "image_height": 2200 }
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "用户姓氏",
      "field_label": "Last name",
      "label_bounding_box": [30, 125, 95, 142],
      "entry_bounding_box": [100, 125, 280, 142],
      "entry_text": { "text": "Johnson", "font_size": 14, "font_color": "000000" }
    },
    {
      "page_number": 2,
      "description": "是否年满 18 岁（勾选）",
      "field_label": "Yes",
      "label_bounding_box": [100, 525, 132, 540],
      "entry_bounding_box": [140, 525, 155, 540],
      "entry_text": { "text": "X" }
    }
  ]
}
```

生成校验图（标记红色=输入区域，蓝色=标签）：
```bash
python scripts/create_validation_image.py <page_number> <fields.json> <input_image> <output_image>
```

## Step 3：校验矩形框（必做）

- **自动检测**：`python scripts/check_bounding_boxes.py <fields.json>` 确保矩形不重叠、尺寸合理。  
- **人工检视**：逐一查看校验图，核对：
  - 红框只覆盖填写区域且不包含文字  
  - 蓝框覆盖标签文本  
  - 勾选框红框需精准覆盖方框

如有问题，修改 `fields.json`、重新生成校验图直至完全准确。

## Step 4：生成带注释的 PDF（必做）

```bash
python scripts/fill_pdf_form_with_annotations.py <input.pdf> <fields.json> <output.pdf>
```

脚本会根据 `entry_bounding_box` 添加文本注释，完成“手写”效果。

---

## 结语

无论表单是否可填写，都必须严格按照脚本和格式要求操作，确保：  
- 字段 ID/坐标准确无误  
- 生成的填表 PDF 通过视觉检查  
- 所有自动脚本均无错误输出  

完成以上步骤后，即可提交最终填好的 PDF。祝你操作顺利！
