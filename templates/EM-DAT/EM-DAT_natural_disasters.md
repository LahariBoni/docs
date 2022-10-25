<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/EM-DAT/EM-DAT_natural_disasters.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=EM-DAT+-+Natural+disasters:+Error+short+description">🚨 Bug report</a>

**Tags:** #em-dat #emdat #opendata #analytics #plotly

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/ACoAAAJHE7sB5OxuKHuzguZ9L6lfDHqw--cdnJg/)

In 1988, the Centre for Research on the Epidemiology of Disasters (CRED) launched the Emergency Events Database (EM-DAT). [EM-DAT](https://www.emdat.be/) was created with the initial support of the World Health Organisation (WHO) and the Belgian Government.

**Idea of improvements :**
- Put all the curves of natural disasters in a logarithmic graph

## Input

In order to use this script, you need to download the dataset on this link :
https://public.emdat.be/
- Create an account and connect
- Go to "Data" tab
- Enter your filters criteria
- Press "Download"

### Import librairies


```python
import pandas as pd
import plotly.express as px
```

### Variable


```python
PATH_CSV = 'path_to_your_file.csv'
```

## Model

### Read the CSV


```python
df = pd.read_csv(PATH_CSV)
```

### Configure the plot


```python
# Types
types_df = df[['Year', 'Disaster Type']]
total_line = types_df[['Year']].value_counts().reset_index(name="value")
total_line['Disaster Type'] = "All"
types_df = types_df.groupby(['Year', 'Disaster Type']).size().reset_index(name="value")
types_df = types_df.append(total_line).sort_values(by=["Year"])

# Countries   
count_by_countries = df[['Year', 'ISO', 'Country']].groupby(['Year', 'ISO', 'Country']).size().reset_index(name='counts')
```

## Output


```python
fig = px.choropleth(
    count_by_countries, locations="ISO",
    color="counts",
    hover_name="Country",
    animation_frame="Year",
    title = "Number of natural disasters per country",
    color_continuous_scale=px.colors.sequential.OrRd,
    range_color=[0, count_by_countries['counts'].max()]
)

fig.update_layout(
    width=850,
    height=600,
    autosize=False,
    template="plotly_white",
    title_x=0.5
)
fig.show()
```


```python
common_kwargs = {'x': "Year", 'y': "value", 'title': "Number of natural disasters per year"}

line_fig = px.line(types_df[types_df['Disaster Type'] == "All"], **common_kwargs)
lineplt_all = px.line(types_df[types_df['Disaster Type'] == "All"], **common_kwargs)
lineplt_filtered = {
    disaster_type: px.line(types_df[types_df['Disaster Type'] == disaster_type], **common_kwargs)
    for disaster_type in types_df['Disaster Type'].unique() if disaster_type != "All"
}
# Add dropdown
line_fig.update_layout(
    updatemenus=[
        dict(
            buttons=list(
                [
                    dict(
                        label="All disasters",
                        method="restyle",
                        args=[{
                            "y": [data.y for data in lineplt_all.data]
                        }]
                    )
                ] + [
                    dict(
                        label=disaster_type,
                        method="restyle",
                        args=[
                            {
                                "y": [data.y for data in lineplt.data],
                            }
                        ]
                    )
                    for disaster_type, lineplt in lineplt_filtered.items()
                ]
            ),
        ),
    ],
    title_x=0.5,
    plot_bgcolor='rgba(0,0,0,0)',
)
line_fig.update_xaxes(gridcolor="grey")
line_fig.update_yaxes(gridcolor="grey")
line_fig.show()
```
