<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/WSR/WHI_Create_indicator.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=WSR+-+WHI+Create+indicator:+Error+short+description">🚨 Bug report</a>

**Tags:** #wsr #whi #indicators #opendata #worldsituationroom #analytics #dataframe #image

**Author:** [Peter Turner](https://www.linkedin.com/in/peter-turner-0839aa116/)

## Input

### Import libraries


```python
import pandas as pd
from PIL import Image, ImageDraw, ImageFont
from datetime import date
```

### Variables


```python
# Input extracted from your open source data
data = [
    {'DATE_PROCESSED': '2021-05-28', 'INDICATOR': 'COVID-19 Active Cases', 'VALUE': 0.21, 'WEIGHT': 4},
    {'DATE_PROCESSED': '2021-05-28', 'INDICATOR': 'Sea Level', 'VALUE': 4.951165245651996, 'WEIGHT': 2},
    {'DATE_PROCESSED': '2021-06-10', 'INDICATOR': 'Delta global temperature', 'VALUE': 4.9, 'WEIGHT': 4},
    {'DATE_PROCESSED': '2021-06-10', 'INDICATOR': 'Arctic Sea Ice level (million square km)', 'VALUE': 4.9, 'WEIGHT': 2}
]

# Input image
input_image = "layout.png"

# Input font
input_font = "ArchivoBlack-Regular.ttf"

# Output image
output_image = "WHI.png"
```

## Model

### Get the name of the files present in output


```python
df = pd.DataFrame(data)
df
```

### Get weighted WHI


```python
def whi(df):
    return round((df['VALUE']*df['WEIGHT']).sum() / df['WEIGHT'].sum(), 2)

whi(df)
```

### Create output image


```python
def create_image(value, datetime):
    img = Image.open(input_image)
    d = ImageDraw.Draw(img)
    
    font = ImageFont.truetype(input_font, 90)
    fill = (255,255,255)
    
    d.text((50,900), "{indicator}/10, {date}".format(date=datetime.strftime("%d/%m/%Y"), indicator=value), font=font, fill=fill)
    return img
```

## Output

### Display image


```python
img = create_image(f'{whi(df)}' , date.today())
display(img)
```

### Save and share image


```python
img.save(output_image)

naas.asset.add(output_image)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(output_image)
```
