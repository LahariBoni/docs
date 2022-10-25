<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Get_Word_Definition_and_Translation.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Get+Word+Definition+and+Translation:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #dictionary #project #word #snippet

**Author:** [Sriniketh Jayasendil](https://twitter.com/srini047/)

**Description:** This notebook get world definition and translation from English using PyDictionary.

## Input

### Import libraries


```python
try:
    from PyDictionary import PyDictionary
except ModuleNotFoundError:
    !pip install PyDictionary
    from PyDictionary import PyDictionary
```

### Setup Variables
- Enter the word in English
- Check the available language codes [here](https://developers.google.com/admin-sdk/directory/v1/languages)


```python
word = input('-> Enter the word to get all details: ')   # Get the word from the user
lang_code = input('-> Enter the language that needs to be translated: ')  # Available language codes: https://developers.google.com/admin-sdk/directory/v1/languages [EN, FR, ES]
```

## Model

### Set PyDictionary


```python
dictionary = PyDictionary()   # Create a PyDictionary Model
```

## Output

### Meaning


```python
print(word + " : ", end="")
meaning = dictionary.meaning(word)
print(meaning)  # replace "earth" with any other word of your choice
```

### Translate


```python
print(word + " => " + dictionary.translate(word, lang_code))
```
