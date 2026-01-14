---
name: xlsx
description: “全面的表格创建、编辑与分析能力，支持公式、格式、数据分析与可视化。用于处理 .xlsx/.xlsm/.csv/.tsv 等表格：①创建含公式和格式的新表；②读取与分析数据；③在保留公式的前提下修改现有表；④在表格内进行数据分析与可视化；⑤重新计算公式。”
license: 专有，完整条款见 LICENSE.txt
---

# Requirements for Outputs

## All Excel files

### Zero Formula Errors
- 所有模型必须 0 公式错误（#REF!、#DIV/0!、#VALUE!、#N/A、#NAME?）。

### Preserve Existing Templates
- 更新模板时，先研究原有格式/风格/习惯并精准匹配。
- 已有模板优先级高于本指南，不要套用标准化格式破坏原样。

## Financial models

### Color Coding Standards
默认为行业标准（除非用户/模板另有规定）：
- **蓝色 (0,0,255)**：手工输入/Scenario 将修改的数字
- **黑色 (0,0,0)**：所有公式与计算
- **绿色 (0,128,0)**：来自同一工作簿其他 sheet 的引用
- **红色 (255,0,0)**：外部文件链接
- **黄色底 (255,255,0)**：关键假设或需要更新的单元格

### Number Formatting Standards
- **Years**：作为文本（“2024” 而非 “2,024”）
- **Currency**：使用 `$#,##0`，标题注明单位（如 “Revenue ($mm)”）
- **Zeros**：用格式把 0 显示为 “-”，含百分比（`$#,##0;($#,##0);-`）
- **Percentages**：默认 0.0%
- **Multiples**：估值倍数使用 `0.0x`
- **Negatives**：用括号 (123) 而不是 -123

### Formula Construction Rules
- **Assumptions**：所有假设（增长、margin、倍数等）放在单独单元格，公式引用这些单元格，不允许硬编码。例如 `=B5*(1+$B$6)`。
- **错误防范**：核对引用、检查 ranges off-by-one、保证所有 projection 期的公式一致、以零/负数等边界值测试、避免循环引用。
- **硬编码文档**：对不可避免的 hardcode，添加注释或邻近单元格说明 `Source: [System/Doc], [Date], [Reference], [URL]`。

# XLSX creation, editing, and analysis

## Overview

用户可能要求创建、编辑或分析 Excel。需根据任务选择合适工具与流程。

## Important Requirements

**必备 LibreOffice**：使用 `recalc.py` 重新计算公式。脚本首次运行会自动配置 LibreOffice。

## Reading and analyzing data

### pandas
用于分析、可视化、基础操作：
```python
import pandas as pd

df = pd.read_excel('file.xlsx')
all_sheets = pd.read_excel('file.xlsx', sheet_name=None)

df.head(); df.info(); df.describe()

df.to_excel('output.xlsx', index=False)
```

## Excel File Workflows

### CRITICAL: Use Formulas, Not Hardcoded Values
所有计算都应由 Excel 公式完成，不要在 Python 中算好再写死。

**错误示例：**
```python
total = df['Sales'].sum()
sheet['B10'] = total
```

**正确示例：**
```python
sheet['B10'] = '=SUM(B2:B9)'
sheet['C5'] = '=(C4-C2)/C2'
sheet['D20'] = '=AVERAGE(D2:D19)'
```

### Common Workflow
1. 选工具：pandas（数据）或 openpyxl（公式/格式）
2. 创建或加载工作簿
3. 修改数据、公式、格式
4. 保存
5. **若包含公式，必须运行 `recalc.py`**：
   ```bash
   python recalc.py output.xlsx
   ```
6. 检查 JSON 输出：若 `status` 为 `errors_found`，按 `error_summary` 修复（#REF!、#DIV/0!、#VALUE!、#NAME? 等），再重算。

### Creating new Excel files
```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])
sheet['B2'] = '=SUM(A1:A10)'

sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet['A1'].alignment = Alignment(horizontal='center')
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### Editing existing Excel files
```python
from openpyxl import load_workbook

wb = load_workbook('existing.xlsx')
sheet = wb.active
for sheet_name in wb.sheetnames:
    sheet = wb[sheet_name]

sheet['A1'] = 'New Value'
sheet.insert_rows(2)
sheet.delete_cols(3)
new_sheet = wb.create_sheet('NewSheet')
new_sheet['A1'] = 'Data'

wb.save('modified.xlsx')
```

## Recalculating formulas

openpyxl 只写入公式字符串，不会计算。使用 `recalc.py`：
```bash
python recalc.py <excel_file> [timeout_seconds]
```
脚本会：
- 首次运行配置 LibreOffice 宏
- 重算所有 sheet 的公式
- 扫描所有单元格的错误并返回 JSON（含错误类型、位置、数量）
- 适用于 Linux/macOS

## Formula Verification Checklist

**基础核对**：
- [ ] 抽查 2-3 个引用是否正确
- [ ] 列号映射：确认 Excel 列（如 64=BL）
- [ ] 行号偏移：Excel 行从 1 开始

**常见陷阱**：
- [ ] NaN：使用 `pd.notna()`
- [ ] 远端列：预算数据常在第 50+ 列
- [ ] 多重匹配：搜索所有匹配
- [ ] 除法前检查分母（防 #DIV/0!）
- [ ] 确认引用不会导致 #REF!
- [ ] 跨 sheet 引用使用 `Sheet1!A1`

**测试策略**：
- [ ] 小范围试验后再扩展
- [ ] 校验依赖单元格存在
- [ ] 测边界值（0、负数、极大值）

**理解 recalc.py 输出**：
```json
{
  "status": "success" | "errors_found",
  "total_errors": 0,
  "total_formulas": 42,
  "error_summary": {
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    }
  }
}
```

## Best Practices

### Library Selection
- **pandas**：数据分析、批量操作、快速导出
- **openpyxl**：公式、格式、Excel 功能

### openpyxl 注意事项
- 行列从 1 开始
- `load_workbook(..., data_only=True)` 可读取值但会丢失公式（保存后不可恢复）
- 大文件可用 `read_only=True` 或 `write_only=True`
- 公式不会自动计算，需 `recalc.py`

### pandas 注意事项
- 指定 dtype：`pd.read_excel('file.xlsx', dtype={'id': str})`
- 只读必要列：`usecols=['A','C','E']`
- 正确解析日期：`parse_dates=['date']`

## Code Style Guidelines

**Python 代码**：保持精炼、避免冗余注释/变量/打印。

**Excel 文件**：对复杂公式或 hardcode 添加 cell comment，记录数据来源，对关键计算/模块写注释。
