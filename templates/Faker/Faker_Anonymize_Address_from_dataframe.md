<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Faker/Faker_Anonymize_Address_from_dataframe.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Faker+-+Anonymize+Address+from+dataframe:+Error+short+description">🚨 Bug report</a>

**Tags:** #faker #operations #snippet #database #dataframe

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

Faker is a Python package that generates fake data for you.<br>
Whether you need to bootstrap your database, create good-looking XML documents, fill-in your persistence to stress test it, or anonymize data taken from a production service, Faker is for you.<br>
This notebook shows how to anonymize address from a column within a dataframe.

<u>References:</u>
- https://pypi.org/project/Faker/
- https://towardsdatascience.com/how-to-quickly-anonymize-personal-names-in-python-6e78115a125b

## Input

### Import libraries


```python
try:
    from faker import Faker
except:
    !pip install faker
    from faker import Faker
import pandas as pd
```

### Setup Data


```python
data = [
    {"Name": "Mike", "Address": "x", "Score": 12},
    {"Name": "Peter", "Address": "z", "Score": 10},
    {"Name": "Lisa", "Address": "z", "Score": 11},
]
df = pd.DataFrame(data)
df
```

### Setup Faker
Use faker.Faker() to create and initialize a faker generator, which can generate data by accessing properties named after the type of data you want.


```python
faker = Faker()

# Column to be anonymize
col_name = "Address"
```

## Model

### Fake address
Through use of the `.unique` property on the generator, you can guarantee that any generated values are unique for this specific instance.


```python
def fake_address(df, col_name):
    dict_names = {name: faker.unique.address() for name in df[col_name].unique()}
    df['New Address'] = df[col_name].map(dict_names)
    return df
```

## Output

### Display result


```python
df_fake_address = fake_address(df, col_name)
df_fake_address
```
