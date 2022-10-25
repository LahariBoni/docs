<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Qonto/Qonto_Get_statement_barline.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Qonto+-+Get+statement+barline:+Error+short+description">🚨 Bug report</a>

**Tags:** #qonto #bank #statement #plotly #barline #naas_drivers #finance #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import qonto
```

### Get your Qonto credentials
<a href='https://www.notion.so/naas-official/Qonto-driver-Get-your-credentials-0cc97828b4e7467c8bfbcf704a77e5f4'>How to get your credentials ?</a>


```python
QONTO_USER_ID = 'YOUR_USER_ID'
QONTO_SECRET_KEY = 'YOUR_SECRET_KEY'
```

### Parameters


```python
# Date to start extraction, format: "AAAA-MM-JJ", example: "2021-01-01"
date_from = None
# Date to end extraction, format: "AAAA-MM-JJ", example: "2021-01-01", default = now
date_to = None
# Title of the graph, default = "Evolution du {date_from} au {date_to}"
title = f"💵<b> Qonto - Suivi des encaissements / Décaissements</b><br>"
# Name of line displayed in legend
line_name = "Solde"
# Line color
line_color = "#1ea1f1"
# Name of cash in bar displayed in legend
cashin_name = "Encaissements"
# Cash in bar color
cashin_color = "#47dd82"
# Name of cash out bar displayed in legend
cashout_name = "Décaissements"
# Cash out bar color
cashout_color = "#ea484f"
```

## Model

### Create barline chart


```python
barline = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.barline(date_from=date_from,
                                                                            date_to=date_to,
                                                                            title=title,
                                                                            line_name=line_name,
                                                                            line_color=line_color,
                                                                            cashin_name=cashin_name,
                                                                            cashin_color=cashin_color,
                                                                            cashout_name=cashout_name,
                                                                            cashout_color=cashout_color)
```

## Output

### Display chart


```python
barline
```


```python

```
