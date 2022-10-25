<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Streamlit/Streamlit_Create_prediction_app.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Streamlit+-+Create+prediction+app:+Error+short+description">🚨 Bug report</a>

**Tags:** #streamlit #app #ml #ai #operations #plotly

**Author:** [Gagan Bhatia](https://github.com/gagan3012)

## Input

### Import library


```python
from naas_drivers import streamlit
```

## Model

Create the Python file necessary to deploy Streamlit app.


```python
%%writefile streamlit_app.py

from naas_drivers import streamlit, plotly, yahoofinance, prediction
import streamlit as st

TICKER = "TSLA"
date_from = -100 # 1OO days max to feed the naas_driver for prediction
date_to = "today"
DATA_POINT = 20

df_yahoo = yahoofinance.get(tickers=TICKER,
                            date_from=date_from,
                            date_to=date_to).dropna().reset_index(drop=True)

df_predict = prediction.get(dataset=df_yahoo,
                            date_column='Date',
                            column="Close",
                            data_points=DATA_POINT,
                            prediction_type="all").sort_values("Date", ascending=False).reset_index(drop=True)

fig = plotly.linechart(df_predict,
                       x="Date",
                       y=["Close", "ARIMA", "SVR", "LINEAR", "COMPOUND"],
                       showlegend=True,
                       title=f"{TICKER} predictions as of today, for next {str(DATA_POINT)} days.")



st.write("# Prediction for {}".format(TICKER))
st.plotly_chart(fig, width=1200)
```

## Output

### Deploy the app from Python file and serve the URL where the app is exposed.


```python
streamlit.add("streamlit_app.py", port=9999, debug=True)
```


```python

```
