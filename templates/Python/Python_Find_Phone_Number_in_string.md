<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Find_Phone_Number_in_string.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Find+Phone+Number+in+string:+Error+short+description">🚨 Bug report</a>

**Tags:** #python #string #number #naas #operations #snippet

**Author:** [Anas Tazir](https://github.com/anastazir)

This notebook find phone number in string.

## Input

### Setup Variables


```python
# Input
sentence = """My number is +33606060606 """

# Ouptut
phone_number = ""
```

## Model

### Find Phone Number


```python
def find_phone_number(string):
    if len(string) == 10 and string.isdecimal(): # if string is of type XXXXXXXXXX
        return True
    if len(string) == 12 and string.startswith("+") and string[1:].isdecimal(): # if string is of type XXXXXXXXXX
        return True
    if len(string) != 12: # if string is of type XXX-XXX-XXXX
        return False
    if string.isdecimal():
        return True
    for i in range(0, 3):
        if not string[i].isdecimal():
            return False
    if string[3] != '-':
        return False
    for i in range(4, 7):
        if not string[i].isdecimal():
            return False
    if string[7] != '-':
        return False
    for i in range(8, 12):
        if not string[i].isdecimal():
            return False
        return True
```


```python
for i in range(len(sentence)):
    split_12 = sentence[i:i+12]
    split_10 = sentence[i:i+10]
    if find_phone_number(split_10):
        phone_number = split_10
        break
    elif find_phone_number(split_12):
        phone_number = split_12
        break
```

## Output

### Display result


```python
if not phone_number:
    print("Not phone number was found.")
else:
    print("Phone number present in the string is :", phone_number)
```
