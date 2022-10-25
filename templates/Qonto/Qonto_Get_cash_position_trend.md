<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Qonto/Qonto_Get_cash_position_trend.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Qonto+-+Get+cash+position+trend:+Error+short+description">🚨 Bug report</a>

**Tags:** #qonto #bank #statement #naas_drivers #plotly #linechart #finance #analytics #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import qonto
from datetime import datetime
import pandas as pd
import plotly.graph_objects as go
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
```

## Model

### Get statement aggregated by date


```python
df_statement = qonto.connect(QONTO_USER_ID, QONTO_SECRET_KEY).statements.get(
    to_group=["IBAN", "DATE"],
    date_from=date_from,
    date_to=date_to
)
df_statement
```

## Output

### Plotting linechart to follow cash position


```python
def get_trend(df_statement,
              date_col_name,
              value_col_name):
    
    # Init dataframe
    df = df_statement.copy()
    
    # Format date
    df[date_col_name] = pd.to_datetime(df[date_col_name]).dt.strftime("%Y-%m-%d")
    
    # Fill empty date
    d = datetime.now().date()
    d2 = df.loc[df.index[0], date_col_name]
    idx = pd.date_range(d2, d, freq = "D")
    
    df.set_index(date_col_name, drop=True, inplace=True)
    df.index = pd.DatetimeIndex(df.index)
    df = df.reindex(idx, fill_value=0)
    df[date_col_name] = pd.DatetimeIndex(df.index)
    df = df.reset_index(drop=True)
    for _, row in df.iterrows():
        if _ > 0:
            iban = df.loc[df.index[_-1], "IBAN"]
            n_1 = df.loc[df.index[_-1], value_col_name]
            n = df.loc[df.index[_], value_col_name]
            if n == 0:
                df.loc[_, value_col_name] = n_1
                df.loc[_, "IBAN"] = iban
    return df

df_trend = get_trend(df_statement, "DATE", "POSITION")
df_trend.tail(10)
```


```python
def create_linechart(df, date, value, var):    
    # Get last value
    df["VALUE_D"] = df[value].map("{:,.2f} €".format).str.replace(",", " ")
    df["VAR_D"] = df[var].map("{:,.2f} €".format).str.replace(",", " ")
    df.loc[df[var].astype(float) > 0, "VAR_D"] = "+" + df["VAR_D"]
    df["TEXT"] = ("<b>Cash position as of " + df["DATE"].astype(str) + " : </b>" + 
                  df["VALUE_D"] + "<br>" + 
                  df["VAR_D"] + " vs yesterday")
    
    last_value = df.loc[df.index[-1], "VALUE_D"]
    last_var = df.loc[df.index[-1], "VAR_D"]
    
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[date],
            y=df[value],
            mode="lines",
            hoverinfo="text",
            text=df["TEXT"],
            line=dict(color="#6b5aed"),
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=f"💵<b> Qonto - Cash position trend</b><br><span style='font-size: 13px;'>Last position : {last_value} ({last_var} vs yesterday)</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Date",
        xaxis_title_font=dict(family="Arial", size=11, color="black"),
        yaxis_title='Amount',
        yaxis_title_font=dict(family="Arial", size=11, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_trend, "DATE", "POSITION", "AMOUNT")
```


```python

```
