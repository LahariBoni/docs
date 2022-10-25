<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Vizzu/Vizzu_Create_interactive_data_story.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Vizzu+-+Create+interactive+data+story:+Error+short+description">🚨 Bug report</a>

**Tags:** #ipyvizzu #googlesheets #charts #cash

**Author:** [Jeremy Ravenel](https://www.linkedin.com/in/jeremyravenel/)

With this notebook, you can create a dynamic chart using [ipyvizzu](https://ipyvizzu.vizzuhq.com/), a library to build animated charts in Jupyter notebook using a simple Python synthax, and share it using Naas asset feature. The data presented is a simple example analysis of cash repartition by bank and maturity.

## Input

### Install packages


```python
!pip install ipyvizzu 
!pip install ipyvizzu-story 
```

### Install librairies


```python
import naas
import pandas as pd
from ipyvizzu import Data, Config, Style
from ipyvizzustory import Story, Slide, Step
import naas_drivers
```

### Setup Google Sheet
Using Naas gsheet driver, we will retrieve data from a remote spreadsheet to feed the charts. More on [docs.naas.ai](https://docs.naas.ai/drivers/google-sheets)


```python
spreadsheet_id = "1YPsBwgBReAaEjVqI6rp3mB2MkGX6tL0kR7x_d07k8WI"
sheet_name = "vizzu_story_example1"
```

## Model

### Get data from Google Sheet


```python
df = naas_drivers.gsheet.connect(spreadsheet_id).get(sheet_name=sheet_name)
df
```

### Add data frame to Vizzu


```python
data = Data()
data.add_data_frame(df)
```

### Configure Vizzu slides


```python
story = Story(data=data)
story.set_size("100%", "550px")

slide1 = Slide(
    Step( 
        Style({
            "legend": {"label": {"fontSize": "1.1em"}, "paddingRight": "-1em"},
            "plot": { 
                "marker": { "label": { "fontSize": "1.1em"}}, 
                "paddingLeft": "10em",
                "xAxis": {"title": { "color": "#00000000"}, "label": { "fontSize": "1.1em"}},
                "yAxis": {"label": { "fontSize": "1.1em"}}},
        }),
        Config({
            "x": {"set": ["Cash percentage [%]","Maturity"]}, 
            "y": "Group",
            "color": "Maturity",
            "label": "Cash percentage [%]",
            "title": "💰 What is the cash repartition and maturity per bank?"
        })
    )
)
story.add_slide(slide1)

slide2 = Slide(
    Step(
        Style({ "plot": { "xAxis": { "label": { "color": "#00000000"}}}}),
        Config({ "split": True, "title": "🗓 What's the ranking by maturity?"})
    )
)
story.add_slide(slide2)

slide3 = Slide(
    Step(
        Style({ "plot": { "marker": { "label": { "fontSize": "0.916667em"}}}}),
        Config({ "x": {"set": ["Cash amount","Maturity"]}, "label": "Cash amount", "title": "⚡️ What's the actual cash positon in M$ by maturity?"}),
    )
)
story.add_slide(slide3)

slide4 = Slide()
slide4.add_step(
    Step(
        Style({ "plot": { "yAxis": { "title": { "color": "#00000000"}}}}),
        Config({ "x": "Maturity", "y": ["Group","Cash amount"], "split": False, "legend": "color"})
    )
)

slide4.add_step(
    Step(
        Style({ "plot": { "marker": { "label": { "fontSize": "1.1em"}}}}),
        Config({ "y": "Cash amount", "title": "👀 What's the total cash position in M$ by maturity?"}),
    )
)
story.add_slide(slide4)

slide5 = Slide()
slide5.add_step(
    Step(
        Config({ "x": ["Interest rate [%]","Maturity"], "y": None, "label":"Cash amount"})
    )
)

slide5.add_step(
    Step(
        Style({ "plot": { "xAxis": {"label": {"color": "#00000000"}}}}),
        Config({ "coordSystem": "polar", "title":"What's the cash repartition in M$ by maturity?"})
    )
)
story.add_slide(slide5)


```

## Output

### Play Vizzu story


```python
story.play()
```

### Export story to HTML


```python
story.export_to_html(filename="mystory.html")
```

### Create shareable asset with Naas


```python
naas.asset.add("mystory.html", params = {"inline": True})
```


```python

```
