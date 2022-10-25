<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Convert_CSV_to_Excel.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Convert+CSV+to+Excel:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #csv #excel #pandas #file

**Author:** [Sophia Iroegbu](www.linkedin.com/in/sophia-iroegbu)

The objective of this notebook is to convert a CSV file into an Excel file.

## Input

### Import library


```python
import pandas as pd
```

### Variables


```python
# Declare the absolute path to the CSV file
csv_input = "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv"

# Name the XSLX spreasheet needed as an output
excel_output = 'covid_dataset.xlsx' 
```

## Model

### Use pandas to: 
- read the CSV file link, 
- then convert it to Excel spreadsheet file by using the `to_excel` function. 

The `sep` argument helps to seperate the rows. It might be a `,` or a `;` and also, ensure the CSV link ends with a CSV file extension. 


```python
convert_csv = pd.read_csv(csv_input, sep=",")
convert_csv
```

## Output

### Save result in .xlsx file

The `header=True` ensures the header from CSV file is converted. This saves the .xlsx file on your root directory.


```python
convert_csv.to_excel(excel_output, index=False, header=True)
```
