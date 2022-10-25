<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Johns%20Hopkins/Johns_Hopkins_Get_Covid19_data.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Johns+Hopkins+-+Get+Covid19+data:+Error+short+description">🚨 Bug report</a>

**Tags:** #johnshopkins #opendata #analytics #csv

**Author:** [Florent Ravenel](https://www.linkedin.com/in/ACoAABCNSioBW3YZHc2lBHVG0E_TXYWitQkmwog/)

## Input

### Import libraries


```python
import pandas as pd
import naas
```

### Variables


```python
# Input URLs of the raw csv dataset
urls = [
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv'
]

# Output paths
title = "DB_COVID19_JHU"
output_csv = f"{title}.csv"
```

## Model

### Get data from JHU


```python
def get_data_url(urls):
    df = pd.DataFrame()
    for url in urls:
        tmp_df = pd.read_csv(url)
        tmp_df["Indicator"] = url.split("/time_series_covid19_")[-1].split("_global.csv")[0].capitalize()
        df = pd.concat([df, tmp_df])
    return df

df_init = get_data_url(urls)
df_init
```

### Get all data from JHU


```python
def get_all_data(df_init):
    df = df_init.copy()
    # Cleaning
    df = df.drop("Province/State", axis=1)
    
    # Melt data
    df = pd.melt(df,
                 id_vars=["Country/Region", "Lat", "Long", "Indicator"],
                 var_name="Date",
                 value_name="Value").fillna(0)
    df["Date"] = pd.to_datetime(df["Date"])
    
    # Calc active cases
    df_active = df.copy()
    df_active.loc[df_active["Indicator"].isin(["Deaths", "Recovered"]), "Value"] = df_active["Value"] * (-1)
    df_active["Indicator"] = "Active cases"
    
    # Concat data
    df = pd.concat([df, df_active])
    
    # Group by country/region
    to_group = ["Country/Region", "Lat", "Long", "Indicator", "Date"]
    df = df.groupby(to_group, as_index=False).agg({"Value": "sum"})
    
    # Cleaning
    df = df.rename(columns={"Country/Region": "COUNTRY"})
    df.columns = df.columns.str.upper()
    return df.reset_index(drop=True)

df_clean = get_all_data(df_init)
df_clean
```

## Output

### Save dataframe in csv


```python
df_clean.to_csv(output_csv, index=False)
```
