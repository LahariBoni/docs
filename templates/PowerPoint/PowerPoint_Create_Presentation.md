<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/PowerPoint/PowerPoint_Create_Presentation.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=PowerPoint+-+Create+Presentation:+Error+short+description">🚨 Bug report</a>

**Tags:** #powerpoint #naas #python #pythonpptx #asset #snippet #operations #slide #microsoft

**Author:** [Ayoub Berdeddouch](https://www.linkedin.com/in/ayoub-berdeddouch)

**Description:** This notebook creates a PowerPoint presentation with a cover page, 3 pages with graphs and text and a last page.

## Input

### Import libraries

* Installation: https://pypi.org/project/python-pptx/
* python-pptx documentation: https://python-pptx.readthedocs.io/en/latest/api/presentation.html


```python
try:
    from pptx import Presentation
except:
    !pip install python-pptx --user
from pptx.enum.shapes import MSO_SHAPE
from pptx.dml.color import RGBColor
from pptx.util import Inches, Pt
from pptx.enum.dml import MSO_THEME_COLOR
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE
from pptx.chart.data import ChartData
from pptx.util import Inches
import numpy as np 
import datetime
import requests
import plotly.graph_objects as go
import pandas as pd
import naas
```

### Setup Variables


```python
# Title
title = (
    'NaaS Automated Presentation Creating Process\n\
    How to Create PowerPoint Presentations with NaaS Jupyter Template'
)

# Logos URL
url_naas = "https://cdn.umso.co/jtci2pxwjczr/assets/axa9wbqm.png?w=1200&h=630&fit=crop" # url naas to get logo
url_ppt = "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQA_0dSO8kAxQFB3s6-cdJAvcYWJKFcHFwr2xzqBauweg&s" # url ppt to get logo

# Logos outputs
naaslogo = 'naaslogo.png' #name of logo naas downloaded on local
pptlogo = 'pptlogo.png' #name of logo ppt downloaded on local

# Presentation pptx
output_pptx = 'basic_presentation_naas.pptx'
```

## Model

### Download image on local


```python
# Function to download image = logo
def download_image(url, file_name):
    # Send GET request
    response = requests.get(url)
    # Save the image
    if response.status_code == 200:
        with open(file_name, "wb") as f:
            f.write(response.content)
    else:
        print(response.status_code)
```


```python
# Download naaslogo
download_image(url_naas, naaslogo)

# Download pptlogo
download_image(url_ppt, pptlogo)
```

### Set Presentation obj


```python
# Presentation obj
prs = Presentation()
prs.slide_width = Inches(16)
prs.slide_height = Inches(9)
```

### Font Page : Create a title and a line within the Presentation


```python
slide = prs.slides.add_slide(prs.slide_layouts[6])

shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE,
    0,
    Inches(9/1.5),
    Inches(16),
    Inches(9/8.5)
)
shape.shadow.inherit = False
fill = shape.fill
fill.solid()
fill.fore_color.rgb = RGBColor(0, 0, 0)
shape.text = title
line = shape.line
line.color.rgb = RGBColor(0, 0, 0)

logo1 = slide.shapes.add_picture(
    naaslogo,
    Inches(13.5),
    Inches(6.0),
    height=Inches(1.0),
    width=Inches(1.0)
)
logo2 = slide.shapes.add_picture(
    pptlogo,
    Inches(14.5),
    Inches(5.8),
    height=Inches(1.5),
    width=Inches(1.5)
)
```

### Page 1 : "How to Add a Chart"


```python
slide = prs.slides.add_slide(prs.slide_layouts[6])

shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE, 0, Inches(0.5),Inches(16),Inches(0.3))
shape.shadow.inherit = False
fill=shape.fill
fill.solid()
fill.fore_color.rgb=RGBColor(0,0,0)
shape.text= "How to Add a Chart"
line=shape.line
line.color.rgb=RGBColor(0,0,0)
logo1=slide.shapes.add_picture(naaslogo,Inches(14.5),Inches(0.4),height=Inches(0.5),width=Inches(0.5))
logo2=slide.shapes.add_picture(pptlogo,Inches(15.0),Inches(0.4),height=Inches(0.5),width=Inches(0.5))

N = 100

random_x = np.random.randn(N) + 10
random_y = np.random.randn(N)+5
random_z = np.random.randn(N) +20

dte=datetime.datetime.today()
dt_lst=[dte-datetime.timedelta(days=i) for i in range(N)]

chart_data = ChartData()
chart_data.categories = dt_lst
chart_data.add_series('Data 1', random_x)
chart_data.add_series('Data 2', random_y)
chart_data.add_series('Data 3', random_z)

x, y, cx, cy = Inches(1), Inches(2), Inches(14), Inches(6)
chart = slide.shapes.add_chart(
    XL_CHART_TYPE.LINE, x, y, cx, cy, chart_data
).chart
chart.has_legend = True
chart.legend.include_in_layout = False
chart.series[2].smooth = True
```

### Page 2 : "How to Add an Chart_2"


```python
slide = prs.slides.add_slide(prs.slide_layouts[6])

shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE, 0, Inches(0.5),Inches(16),Inches(0.3))
shape.shadow.inherit = False
fill=shape.fill
fill.solid()
fill.fore_color.rgb=RGBColor(0,0,0)
shape.text= "How to Add an Chart_2"
line=shape.line
line.color.rgb=RGBColor(0,0,0)
logo1=slide.shapes.add_picture(naaslogo,Inches(14.5),Inches(0.4),height=Inches(0.5),width=Inches(0.5))
logo2=slide.shapes.add_picture(pptlogo,Inches(15.0),Inches(0.4),height=Inches(0.5),width=Inches(0.5))

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/718417069ead87650b90472464c7565dc8c2cb1c/sunburst-coffee-flavors-complete.csv')

fig = go.Figure(go.Sunburst(
        ids = df.ids,
        labels = df.labels,
        parents = df.parents))
fig.update_layout(uniformtext=dict(minsize=10, mode='hide'))

fig.write_image("img.png")

imgpth='img.png'

left = top = Inches(1)
pic = slide.shapes.add_picture(imgpth, left, top)
```

### Page 3 : "How to Add a Text\ Paragraph"


```python
slide = prs.slides.add_slide(prs.slide_layouts[6])

shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE, 0, Inches(0.5),Inches(16),Inches(0.3))
shape.shadow.inherit = False
fill=shape.fill
fill.solid()
fill.fore_color.rgb=RGBColor(0,0,0)
shape.text= "How to Add a Text\ Paragraph"
line=shape.line
line.color.rgb=RGBColor(0,0,0)
logo1=slide.shapes.add_picture(naaslogo,Inches(14.5),Inches(0.4),height=Inches(0.5),width=Inches(0.5))
logo2=slide.shapes.add_picture(pptlogo,Inches(15.0),Inches(0.4),height=Inches(0.5),width=Inches(0.5))

left = Inches(1)
top = Inches(2)
width = Inches(12)
height = Inches(5)

text_box=slide.shapes.add_textbox(left, top, width, height)

tb=text_box.text_frame
tb.text='Ready to use data science templates, organized by tools to jumpstart your projects and data products in minutes.\n\
😎 published by the Naas community.'

prg=tb.add_paragraph()
prg.text=" "

prg=tb.add_paragraph()
prg.text="😎Naas Templates😎 ~~ (aka the awesome-notebooks)"
```

### Last Page : Add author name


```python
slide = prs.slides.add_slide(prs.slide_layouts[6])

shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE,
    0,
    Inches(4.0),
    Inches(16),
    Inches(1.0)
)
shape.shadow.inherit = False
fill = shape.fill
fill.solid()
fill.fore_color.rgb = RGBColor(0, 0, 0)
shape.text = "Thank You, by Ayoub Berdeddouch"
line = shape.line
line.color.rgb = RGBColor(0, 0, 0)

logo1 = slide.shapes.add_picture(
    naaslogo,
    Inches(13.5),
    Inches(4.0),
    height=Inches(1.0),
    width=Inches(1.0)
)
logo2 = slide.shapes.add_picture(
    pptlogo,
    Inches(14.5),
    Inches(4.0),
    height=Inches(1.0),
    width=Inches(1.0)
)
```

## Output

### Save result in csv


```python
prs.save(output_pptx)
```

### Share PowerPoint Presentation with naas asset


```python
# Share output with naas
naas.asset.add(output_pptx)

#-> Uncomment the line below and run the cell to remove your asset
# naas.asset.delete(output_pptx)
```
