<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Bazimo/Bazimo_Get_export_Locataires.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Bazimo+-+Get+export+Locataires:+Error+short+description">🚨 Bug report</a>

**Tags:** #bazimo #pm #naas_drivers #asset #scheduler #naas #csv #operations #automation

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

## Input

### Import library


```python
from naas_drivers import bazimo
import naas
```

### Setup Bazimo


```python
# Credentials
bazimo_email = "bazimo_email"
bazimo_pwd = "bazimo_pwd"

# Export name
bazimo_export = "Locataires"

# Export output
csv_output = f"Export_{bazimo_export}.csv"
```

### Schedule your notebook


```python
# Get your data updated everyday 15 min from 9 to 20 during week days.
naas.scheduler.add(cron="*/15 9-20 * * 1-5")

#-> Uncomment this line and run the cell to delete your scheduler
# naas.scheduler.delete()
```

## Model

### Get export


```python
df_bazimo = bazimo.connect(bazimo_email, bazimo_pwd).exports.get(bazimo_export)
df_bazimo
```

## Output

### Save dataframe in csv


```python
df_bazimo.to_csv(csv_output)
```

### Share your dataframe with naas
-> You can integrate the link below into your Excel spreasheet. Check the procedure [here](https://support.microsoft.com/fr-fr/office/importer-des-donn%C3%A9es-%C3%A0-partir-du-web-b13eed81-33fe-410d-9247-1747269c28e4)


```python
naas.asset.add(csv_output)

#-> Uncomment this line and run the cell to delete your asset
# naas.asset.delete(csv_output)
```
