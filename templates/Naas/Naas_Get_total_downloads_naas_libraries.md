<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Get_total_downloads_naas_libraries.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Get+total+downloads+naas+libraries:+Error+short+description">🚨 Bug report</a>

**Tags:** #pypi #downloads #package #operations #analytics #plotly #html #csv #image #png

**Author:** [Sanjeet Attili](https://linkedin.com/in/sanjeet-attili-760bab190/)

This notebook enables you to get a plot of number of downloads of naas package since past 180 days

## Input

### Import libraries


```python
try:
    import pypistats
except:
    !pip install -U pypistats --user
    !pip install --upgrade pypistats
    import pypistats
# from pprint import pprint
from datetime import datetime
import plotly.graph_objects as go
import naas
import pandas as pd
```

### Variables


```python
packages = ["naas", "naas-drivers"]
```

## Model

**Difference between with_mirrors and without_mirrors**

The with_mirrors and without_mirrors are not mutually exclusive sets of download counts like the other segmentations provided.
In fact, the without_mirrors downloads are a subset of the downloads in with_mirrors.

i.e If you sum the with (a+b) and without (a) mirrors, you count the without mirrors twice (a+b+a).


```python
df_final = pd.DataFrame()
for package in packages:
    df = pypistats.overall(package, total=False, format="pandas")
    df_final = df_final.append(df, ignore_index=True)
df_final
```


```python
# Gives us the cumulative number of downloads over a period of 180 days
def get_cumulative_dataframe(df):
    
    data = df.groupby('category').get_group('with_mirrors').sort_values(
        'date').reset_index(drop='index').groupby(
        'date').agg({'downloads':'sum'}).reset_index()
    
    cum_sum = 0
    for idx, num in enumerate(data['downloads']):
        cum_sum+=num
        data.loc[idx, 'cumulative_downloads'] = cum_sum

    data['cumulative_downloads'] = data.cumulative_downloads.astype('int')
    data.drop(columns = 'downloads', inplace=True)
    
    return data

df_downloads = get_cumulative_dataframe(df_final)
df_downloads.head()
```

## Output

### Plotting a line chart for number of downloads


```python
def create_linechart(df, package, date, value):
    # Get last value
    last_value = "{:,.0f}".format(df.loc[df.index[-1], value]).replace(",", " ")
    
    # Init
    fig = go.Figure()
    
    # Create fig
    fig.add_trace(
        go.Scatter(
            x=df[date].to_list(),
            y=df[value].to_list(),
            mode="lines+text",
            line=dict(color="black"),
        )
    )
    fig.update_traces(marker_color='black')
    fig.update_layout(
        title=f"⭐<b> Total number of downloads for naas libraries </b><br><span style='font-size: 13px;'> Total Downloads as of today: {last_value}</span>",
        title_font=dict(family="Arial", size=18, color="black"),
        plot_bgcolor="#ffffff",
        width=1200,
        height=800,
        paper_bgcolor="white",
        xaxis_title="Date",
        xaxis_title_font=dict(family="Arial", size=13, color="black"),
        yaxis_title='No. of downloads',
        yaxis_title_font=dict(family="Arial", size=13, color="black"),
        margin_pad=10,
    )
    fig.show()
    return fig

fig = create_linechart(df_downloads, packages, "date", "cumulative_downloads")
```

### Save and share your csv file


```python
# Save your dataframe in CSV
df_downloads.to_csv(csv_output, index=False)

# Share output with naas
naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(csv_output)
```

### Save and share your graph in HTML


```python
# Save your graph in HTML
fig.write_html(html_output)

# Share output with naas
naas.asset.add(html_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(html_output)
```

### Save and share your graph in image


```python
# Save your graph in PNG
fig.write_image(image_output)

# Share output with naas
naas.asset.add(image_output, params={"inline": True})

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
