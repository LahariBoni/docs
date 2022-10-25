<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Excel/Excel_Custom_sheet.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Excel+-+Custom+sheet:+Error+short+description">🚨 Bug report</a>

**Tags:** #excel #openpyxl #font #border #background #naas #finance #snippet

**Author:** [Sébastien Grech](https://www.linkedin.com/in/s%C3%A9bastien-grech-4433a7150/)

Here below we will apply different formatting on an Excel sheet

## Input

### Import libraries


```python
import naas
from openpyxl import load_workbook
from openpyxl.cell import Cell
from openpyxl.styles import Color, PatternFill, Font, Border
from openpyxl.styles.borders import Border, Side
```

### Setup your variables


```python
# Inputs
excel_init_path = "Excel_Template.xlsx"

# Outputs
excel_out_path = "Excel_Custom.xlsx"
```

### Setup your custom style
NB: Colors must be aRGB hex values : 'black' = '000000'


```python
# Sheet Range
sheet_range = "A1:M54"

# Sheet Font
sheet_font = Font(name='Arial', bold=False, color='000000', size='11')

# Border style
sheet_border = Border(
    left=Side(border_style='thin', color='000000'),
    right=Side(border_style='thin',color='000000'),
    top=Side(border_style='thin', color='000000'),
    bottom=Side(border_style='thin',color='000000')
)
```


```python
# Number range
number_range = "B2:M54"

# Number format
number_format =  '#,##0'
```


```python
# Header range
header_range = "1:1"

# Header background
header_bg = PatternFill(start_color='24292e',
                        end_color='24292e',
                        fill_type= 'solid' )

# Header font
header_font = Font(name='Arial',
                   bold=True,
                   color='FFFFFF',
                   size='11')
```


```python
# Total range
total_range = "54:54"

# Total background
total_bg = PatternFill(start_color='47DD82',
                       end_color='47DD82',
                       fill_type= 'solid' )
```

## Model

### Load Excel file and get active worksheet


```python
wb = load_workbook(excel_init_path)
ws = wb.active
ws
```

### Apply sheet style : Font and border


```python
cell_range = ws[sheet_range]
for row in cell_range:
    for cell in row:
        cell.font = sheet_font
        cell.border = sheet_border
```

### Apply number format


```python
cell_range = ws[number_range]
for row in cell_range:
    for cell in row:
        cell.number_format = number_format 
```

### Apply header format


```python
for cell in ws[header_range]:    
    cell.fill = header_bg
    cell.font = header_font
```

### Apply total format


```python
for cell in ws[total_range]:    
    cell.fill = total_bg
```

## Output

### Save new excel


```python
wb.save(excel_out_path)
```

### Share your excel


```python
naas.asset.add(excel_out_path)
```
