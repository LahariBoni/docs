<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Delete_Webhook.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Naas+-+Delete+Webhook:+Error+short+description">Bug report</a> | <a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Start_data_product.ipynb" target="_parent">Generate Data Product</a>

**Tags:** #naas #scheduler #snippet #operations

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel)

**Description:** This notebook will show how to delete a naas webhook.

**References:**
- [Naas Documentation](https://docs.naas.ai/)
- [Naas Webhook Documentation](https://docs.naas.ai/features/api)

## Input

### Import libraries


```python
import naas
```

### Setup Variables
- `path`: Go to your Naas manager, section "Jobs" and copy/paste the path of the scheduler you want to delete.

If this notebook is not on your root folder, you need to set absolute path of your notebook starting with "/home/ftp/".   


```python
path = ""
```

## Model

### Delete Webhook


```python
naas.webhook.delete(path=path)
```

## Output

### Display result


```python
print("🗑 Done! Your Notebook has been remove from production.")
```

 