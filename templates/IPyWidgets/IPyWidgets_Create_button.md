<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/IPyWidgets/IPyWidgets_Create_button.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=IPyWidgets+-+Create+button:+Error+short+description">🚨 Bug report</a>

**Tags:** #ipywidgets #naas #secret #snippet #operation #button

**Author:** [Ismail CHIHAB](https://www.linkedin.com/in/ismail-chihab-4b0a04202/)

This notebook uses ipywidgets to create a button.

## Input

### Import libraries


```python
from IPython.display import display, clear_output
from ipywidgets import widgets
```

### Setup Variables


```python
#Button variables:
button_description = 'Click me'
button_style = 'info'  #You can also enter: 'success', 'info', 'warning', 'danger' or ''
button_icon = 'check'

#Result variables
button_message = 'Button Clicked' #Message display on click
```

## Model

### Create button and event on click


```python
# Setup ipywidgets
button = widgets.Button(
    description=button_description,
    button_style=button_style,
    button_icon=button_icon
)

# Setup output
output = widgets.Output()

# Event on click
def click(b):
    output.clear_output()
    with output:
        print(button_message)
```

## Output

### Display button


```python
# Display
display(button, output)

# Action on click
button.on_click(click)
```
