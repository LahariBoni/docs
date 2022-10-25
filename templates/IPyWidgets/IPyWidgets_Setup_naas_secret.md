<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/IPyWidgets/IPyWidgets_Setup_naas_secret.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=IPyWidgets+-+Setup+naas+secret:+Error+short+description">🚨 Bug report</a>

**Tags:** #ipywidgets #naas #secret #snippet #operation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook uses ipywidgets to set your naas secret.

## Input

### Import libraries


```python
from IPython.display import display, clear_output
from ipywidgets import widgets
import naas
```

### Setup Variables


```python
SECRET_KEY = "LINKEDIN_PROFILE_URL"
DEFAULT_VALUE = "https://www.linkedin.com/in/my-profile/"
```

## Model

### Function


```python
style = {'description_width': '150px'}
layout = widgets.Layout(width='500px')

value = DEFAULT_VALUE

# Check if secret exists
if naas.secret.get(SECRET_KEY):
    value = naas.secret.get(SECRET_KEY)

# Setup ipywidgets
text = widgets.Text(description='Profile URL:',
                    display='flex',
                    value=value,
                    style=style,
                    layout=layout)
save_button = widgets.Button(description="Save profile", button_style='success')
connect_button = widgets.Button(description="Connect LinkedIn")
Box = widgets.HBox([text, save_button])

# Setup output
output = widgets.Output()

# Events on click
def save_secret(b):
    naas.secret.add(SECRET_KEY, text.value)
    output.clear_output()
    with output:
        print("✅ Secrets successfully pushed:", text.value)
```

## Output

### Display secret
If secret empty, default value is displayed.


```python
# Display
display(Box, output)
save_button.on_click(save_secret)
```


```python

```
