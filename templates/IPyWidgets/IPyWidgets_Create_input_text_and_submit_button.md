<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/IPyWidgets/IPyWidgets_Create_input_text_and_submit_button.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=IPyWidgets+-+Create+input+text+and+submit+button:+Error+short+description">🚨 Bug report</a>

**Tags:** #ipywidgets #naas #secret #snippet #operation #button #text

**Author:** [Ismail CHIHAB](https://www.linkedin.com/in/ismail-chihab-4b0a04202/)

This notebook creates an input text area and a submit button to get your input.

## Input

### Import libraries


```python
from IPython.display import display, clear_output
from ipywidgets import widgets
```

### Setup variable


```python
#Text area variables:
input_description = 'Description' 
style = {
    'description_width': '150px'
}
layout = widgets.Layout(width='400px')
value ='Enter your value here'

#Button variables:
button_description = 'Submit'
button_style = 'success'

#Result variables:
result_message = 'Your input is:'
```

## Model

### Setup widgets and event on click


```python
# Setup input text widget
text = widgets.Text(
    description=input_description,
    value=value,
    style=style,
    layout=layout
)

# Setup button widget
button = widgets.Button(
    description=button_description,
    button_style=button_style
)

# Setup HBox to display widget horizontally
Box = widgets.HBox([text, button])

# Setup output
output = widgets.Output()

# Event on click
def save(b):
    output.clear_output()
    with output:
        print(result_message, text.value)
```

## Output

### Display secret
If secret empty, default value is displayed.


```python
# Display
display(Box, output)
button.on_click(save)
```
