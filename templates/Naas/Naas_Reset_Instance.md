<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Reset_Instance.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Reset+Instance:+Error+short+description">🚨 Bug report</a>

**Tags:** #naas #scheduler #operations #snippet

**Author:** [Maxime Jublou](https://www.linkedin.com/in/maximejublou)

The goal of this notebook is to reset your naas instance.

## Input

### Load libraries


```python
import naas
```

## Model

### Reset naas instance


```python
def reset_naas_instance():
    ! rm -rf ~/.clean || true
    ! mkdir ~/.clean || true
    ! mv ~/.* ~/.clean
    naas.update()
```

## Output


```python
reset_naas_instance()
```
