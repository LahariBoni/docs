<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Pillow/Pillow_Add_data_to_image.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Pillow+-+Add+data+to+image:+Error+short+description">🚨 Bug report</a>

**Tags:** #pillow #opendata #snippet #data #image

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import libraries


```python
from PIL import Image, ImageDraw, ImageFont
import naas
```

### Setup variables


```python
# Input
input_image = "layout.png"
input_font = "ArchivoBlack-Regular.ttf"

# Model
text_position = (50, 900)
text = "Your text"
font_size = 90

# Output
output_image = "output_image.png"
```

## Model

### Create output image


```python
def create_image(input_image,
                 output_image,
                 text_position,
                 text,
                 font_size=90,
                 input_font=None):
    img = Image.open(input_image)
    d = ImageDraw.Draw(img)
    
    font = ImageFont.truetype(input_font, font_size)
    fill = (255, 255, 255)
    
    d.text(text_position, text, font=font, fill=fill)
    img.save(output_image)
    print("💾 Image saved :", output_image)
    return img

img = create_image(input_image, output_image, text_position, text, font_size, input_font)
display(img)
```

## Output

### Save and share image


```python
naas.asset.add(output_image)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(output_image)
```
