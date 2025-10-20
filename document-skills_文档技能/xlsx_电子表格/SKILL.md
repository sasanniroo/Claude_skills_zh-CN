---
name: xlsx
description: “面向电子表格（.xlsx、.xlsm、.csv、.tsv 等）的创建、编辑与分析”，支持公式、格式、数据分析与可视化。当 Claude 需要（1）创建带公式与格式的新表格，（2）读取或分析数据，（3）在保留公式的前提下修改已有表格，（4）在表格内做数据分析与可视化，以及（5）重新计算公式时，请使用本技能。
license: 专有许可。完整条款见 LICENSE.txt
---

# 输出要求

## 所有 Excel 文件

### 零公式错误
- 交付的 Excel 文件必须**没有任何公式错误**（#REF!、#DIV/0!、#VALUE!、#N/A、#NAME? 等）。

### 保留既有模板（更新模板时）
- 在修改既有文件时，必须认真学习并**完全复刻**现有的格式、风格与约定。
- 不要在已有模式的文件中强行套用统一格式。
- 若模板已有约定，此处指南一律让路。

## 财务模型

### 色彩编码规范
除非用户或既有模板另有说明，默认遵循行业标准：
- **蓝色文本（RGB 0,0,255）**：用户可调的手动输入值与场景参数
- **黑色文本（RGB 0,0,0）**：所有公式与计算
- **绿色文本（RGB 0,128,0）**：来自同一工作簿其他工作表的引用
- **红色文本（RGB 255,0,0）**：引用其他文件的外部链接
- **黄色填充（RGB 255,255,0）**：需特别看护的关键假设或待更新的单元格

### 数字格式规范
- **年份**：使用文本格式（如 "2024"，而非 "2,024"）
- **货币**：使用 `$#,##0`，并在表头标注单位（如 “Revenue ($mm)”）
- **零值**：通过格式设置将 0 显示为 “-”，包括百分比（如 `$#,##0;($#,##0);-`）
- **百分比**：默认保留一位小数（0.0%）
- **倍数**：估值倍数（EV/EBITDA、P/E）写作 0.0x
- **负数**：使用括号表示 (123)，而非 -123

### 公式构建规范

#### 假设分离
- 所有假设（增长率、利润率、倍数等）必须放在专门的假设单元格中
- 公式中使用单元格引用，而非硬编码常数
- 示例：使用 `=B5*(1+$B$6)`，不要写 `=B5*1.05`

#### 防错措施
- 核对所有单元格引用是否正确
- 检查范围是否出现 off-by-one 错误
- 在所有预测期内保持公式一致
- 用极端值（0、负数）测试公式
- 确认没有非预期的循环引用

#### 硬编码值的记录要求
- 在单元格旁或表格末尾备注来源，格式：`"Source: [来源系统/文档], [日期], [具体说明], [URL 如适用]"`
- 示例：
  - `"Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]"`
  - `"Source: Company 10-Q, Q2 2025, Exhibit 99.1, [SEC EDGAR URL]"`
  - `"Source: Bloomberg Terminal, 8/15/2025, AAPL US Equity"`
  - `"Source: FactSet, 8/20/2025, Consensus Estimates Screen"`

# XLSX 的创建、编辑与分析

## 概览

用户可能要求你创建、编辑或分析 .xlsx 文件。不同任务适合不同工具与流程。

## 重要要求

**公式重算需要 LibreOffice**：可默认 LibreOffice 已安装，并使用 `recalc.py` 脚本重新计算公式。脚本首次运行会自动配置 LibreOffice。

## 数据读取与分析

### 使用 pandas 进行数据分析
pandas 提供强大的数据处理与可视化能力，适用于分析与基础操作：

```python
import pandas as pd

# 读取 Excel
df = pd.read_excel('file.xlsx')  # 默认读取第一个工作表
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)  # 所有工作表组成的字典

# 分析
df.head()      # 预览
df.info()      # 列信息
df.describe()  # 统计摘要

# 写回 Excel
df.to_excel('output.xlsx', index=False)
```

## Excel 文件工作流

## 关键原则：使用公式，不要硬编码结果

**必须让 Excel 公式完成计算，而不是在 Python 中算出数字再写入**，以保持表格的可维护性。

### ❌ 错误示例 —— 在 Python 中硬编码结果
```python
# 错误：Python 计算后写死总和
total = df['Sales'].sum()
sheet['B10'] = total  # 写入 5000

# 错误：Python 计算增长率
growth = (df.iloc[-1]['Revenue'] - df.iloc[0]['Revenue']) / df.iloc[0]['Revenue']
sheet['C5'] = growth  # 写入 0.15

# 错误：Python 计算平均值
avg = sum(values) / len(values)
sheet['D20'] = avg  # 写入 42.5
```

### ✅ 正确示例 —— 使用 Excel 公式
```python
# 正确：让 Excel 自己求和
sheet['B10'] = '=SUM(B2:B9)'

# 正确：增长率公式
sheet['C5'] = '=(C4-C2)/C2'

# 正确：平均值
sheet['D20'] = '=AVERAGE(D2:D19)'
```

上述原则适用于所有计算：总和、百分比、比率、差值等。只要源数据更新，表格必须能重新计算。

## 常见工作流
1. **选择工具**：数据处理用 pandas，公式/格式用 openpyxl
2. **创建 / 加载**：新建工作簿或载入现有文件
3. **修改**：写入数据、设置公式与格式
4. **保存**：写回文件
5. **重新计算公式（若使用公式则必做）**：
   ```bash
   python recalc.py output.xlsx
   ```
6. **核对并修复错误**：
   - 脚本输出 JSON，包含错误详情
   - 若 `status` 为 `errors_found`，查看 `error_summary` 中错误类型与位置
   - 修复后再次运行脚本
   - 常见错误：
     - `#REF!`：引用无效单元格
     - `#DIV/0!`：除以 0
     - `#VALUE!`：数据类型不匹配
     - `#NAME?`：公式名称拼写错误或函数不可用

### 创建新 Excel

```python
# 使用 openpyxl 控制公式与格式
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active

# 写入数据
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])

# 添加公式
sheet['B2'] = '=SUM(A1:A10)'

# 设置格式
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')

# 列宽
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### 编辑现有 Excel

```python
from openpyxl import load_workbook

# 载入文件
wb = load_workbook('existing.xlsx')
sheet = wb.active  # 或 wb['SheetName']

# 遍历所有工作表
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]
    print(f"Sheet: {sheet_name}")

# 修改单元格
sheet['A1'] = 'New Value'
sheet.insert_rows(2)  # 在第 2 行插入
sheet.delete_cols(3)  # 删除第 3 列

# 新增工作表
new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = 'Data'

wb.save('modified.xlsx')
```

## 重新计算公式

openpyxl 写入的公式只是字符串，并不会计算结果。请使用 `recalc.py`：

```bash
python recalc.py <excel_file> [timeout_seconds]
```

示例：
```bash
python recalc.py output.xlsx 30
```

脚本会：
- 首次运行时自动配置 LibreOffice 宏
- 重新计算所有工作表内的公式
- 检查全部单元格的 Excel 错误（#REF!、#DIV/0! 等）
- 返回包含错误位置与数量的 JSON
- 适用于 Linux 与 macOS

## 公式核对清单

### 基础检查
- [ ] **抽测 2-3 个引用**：在建立完整模型前，确认引用值正确
- [ ] **列匹配**：确认 Excel 列号（如第 64 列是 BL 而非 BK）
- [ ] **行偏移**：Excel 行号从 1 开始（DataFrame 第 5 行对应 Excel 第 6 行）

### 常见陷阱
- [ ] **NaN 处理**：用 `pd.notna()` 检查空值
- [ ] **右侧列**：年度数据常在第 50 列之后
- [ ] **多匹配**：检索时要找到所有匹配项
- [ ] **除零检查**：在公式中使用 `/` 之前确保分母非零（避免 #DIV/0!）
- [ ] **引用正确性**：确认所有引用目标正确（避免 #REF!）
- [ ] **跨表引用**：遵守 `Sheet1!A1` 格式

### 测试策略
- [ ] **由小到大**：先在 2-3 个单元格测试公式，再批量应用
- [ ] **核对依赖**：确保公式引用的所有单元格均已存在
- [ ] **测试边界**：包括 0、负数、极大值等情境

### 解读 recalc.py 输出

```json
{
  "status": "success",           // 或 "errors_found"
  "total_errors": 0,              // 错误数量
  "total_formulas": 42,           // 公式总数
  "error_summary": {              // 仅在有错误时包含
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## 最佳实践

### 库选择
- **pandas**：适合数据分析、大批量操作与快速导出
- **openpyxl**：适合复杂格式、公式与 Excel 特性

### 使用 openpyxl
- 单元格索引从 1 开始（row=1, column=1 -> A1）
- 若需读取公式结果可使用 `data_only=True`：`load_workbook('file.xlsx', data_only=True)`
- **警告**：若以 `data_only=True` 打开并保存，会将公式永久替换为数值
- 大文件可使用 `read_only=True` 或 `write_only=True`
- 公式仅以字符串形式保存，需结合 recalc.py 更新数值

### 使用 pandas
- 指定数据类型以避免推断错误：`pd.read_excel('file.xlsx', dtype={'id': str})`
- 大文件可指定列：`pd.read_excel('file.xlsx', usecols=['A', 'C', 'E'])`
- 正确处理日期：`pd.read_excel('file.xlsx', parse_dates=['date_column'])`

## 代码风格指南
**重要**：编写 Excel 相关 Python 代码时：
- 代码简洁、必要注释最少
- 避免冗长变量名与重复逻辑
- 避免无意义的 print 输出

**对于 Excel 文件本身**：
- 在复杂公式或关键假设处添加单元格批注
- 为硬编码数据注明来源
- 为重要计算与模型结构添加说明
