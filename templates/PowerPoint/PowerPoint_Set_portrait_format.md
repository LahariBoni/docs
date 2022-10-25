<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/PowerPoint/PowerPoint_Set_portrait_format.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=PowerPoint+-+Set+portrait+format:+Error+short+description">🚨 Bug report</a>

**Tags:** #powerpoint #slide #portrait #pptx #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This template set portrait format in PowerPoint presentation.<br>
Reference: https://python-pptx.readthedocs.io/en/latest/user/quickstart.html

## Input

### Import libraries


```python
try:
    from pptx import Presentation
except:
    !pip install python-pptx --user
    from pptx import Presentation
from pptx.util import Inches
```

### Setup Variables


```python
output_pptx = 'MyPowerPoint_Slide.pptx'
```

## Model

### Create Presentation Obj


```python
prs = Presentation()
```

### Setup Width & Height (Portrait)


```python
prs.slide_height = Inches(11.69)
prs.slide_width = Inches(8.27)
```

### Add slide


```python
blank_slide_layout = prs.slide_layouts[6]
slide = prs.slides.add_slide(blank_slide_layout)
```

## Output

### Save presentation


```python
prs.save(output_pptx)
```
