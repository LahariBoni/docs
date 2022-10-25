<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Johns%20Hopkins/Johns_Hopkins_Covid19_Active_Cases.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Johns+Hopkins+-+Covid19+Active+Cases:+Error+short+description">🚨 Bug report</a>

**Tags:** #johnshopkins #opendata #analytics #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

## Input

### Import libraries


```python
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
import matplotlib.pyplot as plt
```

### Variables


```python
# URLs of the raw csv dataset
urls = [
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv',
    'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv'
]

confirmed_df, deaths_df, recovered_df = tuple(pd.read_csv(url) for url in urls)
```

## Model
Mostly adopted from this [COVID19 Data Processing Tutorial](https://towardsdatascience.com/covid-19-data-processing-58aaa3663f6)

Clean the dataset to show the cases by country

Steps:

1. Convert from Wide to Long Dataframe (Convert all datetimes to a single column)

2. Merge/Join the Confirmed, Deaths and Recovered tables into a single table

3. Converting Date from string to datetime

4. Replacing missing values/NaNs
5. Coronavirus cases reported from 3 cruise ships should be treated differently and adjustments need to be made for Canada (deciding to drop Canada due to missing recovery data)

6. Get Active Cases = Confirmed - Deaths - Recovered



```python
#Wide to Long DataFrame conversion
dates = confirmed_df.columns[4:]
confirmed_df_long = confirmed_df.melt(
    id_vars=['Province/State', 'Country/Region', 'Lat', 'Long'], 
    value_vars=dates, 
    var_name='Date', 
    value_name='Confirmed'
)
deaths_df_long = deaths_df.melt(
    id_vars=['Province/State', 'Country/Region', 'Lat', 'Long'], 
    value_vars=dates, 
    var_name='Date', 
    value_name='Deaths'
)
recovered_df_long = recovered_df.melt(
    id_vars=['Province/State', 'Country/Region', 'Lat', 'Long'], 
    value_vars=dates, 
    var_name='Date', 
    value_name='Recovered'
)

# Adjust for Canada
recovered_df_long = recovered_df_long[(recovered_df_long['Country/Region']!='Canada')]
```


```python
# Join into one single dataframe/table
# Merging confirmed_df_long and deaths_df_long
full_table = confirmed_df_long.merge(
  right=deaths_df_long, 
  how='left',
  on=['Province/State', 'Country/Region', 'Date', 'Lat', 'Long']
)
# Merging full_table and recovered_df_long
full_table = full_table.merge(
  right=recovered_df_long, 
  how='left',
  on=['Province/State', 'Country/Region', 'Date', 'Lat', 'Long']
)

# Convert date strings to actual dates
full_table['Date'] = pd.to_datetime(full_table['Date'])
# Handle some missing values / NaNs
full_table['Recovered'] = full_table['Recovered'].fillna(0).astype('int64')

```


```python
full_table.isna().sum()
# full_table.dtypes
```


```python
# Adjust for Canada and 3 cruise ships
ship_rows = full_table['Province/State'].str.contains('Grand Princess') | full_table['Province/State'].str.contains('Diamond Princess') | full_table['Country/Region'].str.contains('Diamond Princess') | full_table['Country/Region'].str.contains('MS Zaandam')
full_ship = full_table[ship_rows]
full_table = full_table[~(ship_rows)]

# Add one more entry for each day to get the entire world's counts/totals
world_dict = {"Country/Region": "World", "Confirmed": pd.Series(full_table.groupby(['Date'])['Confirmed'].sum()), "Deaths": pd.Series(full_table.groupby(['Date'])['Deaths'].sum()),"Recovered": pd.Series(full_table.groupby(['Date'])['Recovered'].sum())}
world_df = pd.DataFrame.from_dict(world_dict).reset_index()
print(world_df.columns)
full_table = pd.concat([full_table, world_df], ignore_index=True)
```


```python
# Active Cases = Confirmed - Deaths - Recovered
full_table['Active'] = full_table['Confirmed'] - full_table['Deaths'] - full_table['Recovered']

full_grouped = full_table.groupby(['Date', 'Country/Region'])['Confirmed', 'Deaths', 'Recovered', 'Active'].sum().reset_index()
```


```python
len(full_grouped["Country/Region"].unique())
```

### Interactive Dropdown Visualization for Active Cases by Country
First, need to go back from long to wide for a format suited to the visualization using `df.pivot()`

Mostly adopted from this [Interactive Dropdown Tutorial](https://towardsdatascience.com/how-to-create-an-interactive-dropdown-in-jupyter-322277f58a68)


```python
# Go back from long to wide for viz purposes
df = full_grouped
df.rename(columns={"Country/Region": "Country"}, inplace=True)
df_confirmed = df[["Date", "Country", "Confirmed"]]
df_deaths = df[["Date", "Country", "Deaths"]]
df_active = df[["Date", "Country", "Active"]]
df_recovered = df[["Date", "Country", "Recovered"]]

df_confirmed = df_confirmed.pivot(index="Date", columns="Country", values="Confirmed")
df_deaths = df_deaths.pivot(index="Date", columns="Country", values="Deaths")
df_recovered = df_recovered.pivot(index="Date", columns="Country", values="Recovered")
df_active = df_active.pivot(index="Date", columns="Country", values="Active")
```


```python
def create_layout_button(df, column):
    first, latest = df.index.values[0], df.index.values[-1]
    return dict(label = column,
                method = 'update',
                args = [{'visible': df.columns.isin([column]),
                         'title': column,
                         'xaxis.range': [first, latest],
                         'showlegend': True
                        }])

def multi_plot(df, title, addAll = True):
    first, latest = df.index.values[0], df.index.values[-1]
    fig = go.Figure()

    for column in df.columns.to_list():
        fig.add_trace(
            go.Scatter(
                x = df.index,
                y = df[column],
                name = column
            )
        )

    button_all = dict(label = 'All',
                  method = 'update',
                  args = [{'visible': df.columns.isin(df.columns),
                           'title': 'All',
                           'xaxis.range': [first, latest],
                           'showlegend':True}])
    
    # Need "World" to be the default choice if "All" is not shown
    button_world = create_layout_button(df, "World")

    fig.update_layout(
        updatemenus=[{
            "active": 0,
            "buttons": ([button_all] * addAll) + [button_world] + [create_layout_button(df, column) for column in df.columns if column != "World"],
            "showactive": True
            }
        ],
        yaxis_type="log"
    )
    
    # Update remaining layout properties
    fig.update_layout(
        title_text=title,
#         annotations=[dict(
#             text="Country:",
#             x=0, y=0
#         )]
    )
   
    fig.show()
```


```python
# test_df_active = df_active.swapaxes("index", "columns")
test_df_active = df_active
latest = test_df_active.index.values[-1]
print(latest)
test_df_active = test_df_active.T.sort_values(by=latest, ascending=False).head(11).T
test_df_active
```

## Output

### Logarithmic COVID-19 time series


```python
multi_plot(test_df_active, title="Logarithmic COVID-19 time series Active Cases by Country (Top 10)")
```


```python
multi_plot(df_active, title="Logarithmic COVID-19 time series Active Cases by Country", addAll=False)
```

### World Health Indicator (WHI)
Using a scale of **0 - 10** and rescaling the number of Active Cases / Confirmed Cases on the entire World's Data

(where 0 is the worst and 10 is the best)

<!-- \begin{equation*}
WHI = 10 - 10 \times \frac{\text{Current Monthly average} - Min(\text{Monthly  average})}{Max(\text{Monthly average}) - Min(\text{Monthly average})}
\end{equation*}
 -->
 
 \begin{equation*}
WHI = 10 - 10 \times \frac{Current - Min}{Max - Min}
\end{equation*}

 
(Using **Linear Scaling** for now, will discuss and develop a better scaling mechanism if required)


```python
# Uncomment to get a 30 day Moving Average Statistics and a health indicator based on that

# df_active["MonthlyAverage"] = df_active["World"].rolling('30D').mean().astype('int64')
# curr_30d = df_active.loc[latest, "MonthlyAverage"]
# max_30d = df_active["MonthlyAverage"].max()
# min_30d = df_active["MonthlyAverage"].min()
# WHI_30d = 10 - 10 * ((curr_30d - min_30d) / (max_30d - min_30d))
#print(f"World Health Indicator (30 day Moving Average): {round(WHI_30d, 2)}")
```


```python
WHI = 10 - 10 * ((df_active.loc[latest, "World"] - df_active["World"].min()) / (df_active["World"].max() - df_active["World"].min()))

print(f"World Health Indicator (Raw values): {round(WHI, 2)}")
WHI_data = pd.DataFrame.from_dict({"DATE_PROCESSED": pd.to_datetime("today").date(), "INDICATOR": "COVID-19 Active Cases", "VALUE": [round(WHI, 2)]})
WHI_data
```
