<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Knative/Knative_Create_command_file.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Knative+-+Create+command+file:+Error+short+description">🚨 Bug report</a>

**Tags:** #knative #operations #dashboards #dash #snippet

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook create a command file to deploy your dashboard in production using Knative.<br>
The deployment or your dashboard can only be realized by a member of Naas Core Team regarding your user plan.

## Input

### Setup Variables


```python
file_output = "knative_deploy.txt"
endpoint = "search"
github_projet = "https://github.com/jupyter-naas/naas-search-insights.git"
dash_notebook = "Dash_Create_Search.ipynb"
```

## Model

### Create dynamic variables


```python
repos_name = github_projet.split("/")[-1].replace(".git", "")
dash_notebook_out = dash_notebook.replace(".ipynb", ".out.ipynb")
```

### Write template


```python
%%writefile $file_output
kn service create [endpoint] \
    -n live \
    --port 8050 \
    --image jupyternaas/naas:latest \
    --scale-min 1 \
    --cmd /bin/bash \
    --cmd "-c" \
    --cmd "echo hello && git clone [github_projet] && cd [repos_name] && pip install -r requirements.txt && pip install jupytext awswrangler dash dash_bootstrap_components && cd models && papermill -p KNATIVE True [dash_notebook] [dash_notebook_out]"
```

### Replace values in template


```python
content = open(file_output, "r").read()
content = content.replace("[endpoint]", endpoint)
content = content.replace("[github_projet]", github_projet)
content = content.replace("[repos_name]", repos_name)
content = content.replace("[dash_notebook]", dash_notebook)
content = content.replace("[dash_notebook_out]", dash_notebook_out)
content
```

## Output

### Write command in txt


```python
with open(file_output, 'w') as f:
    f.write(content)
print("✅ Knative command file successfully created.")
```
