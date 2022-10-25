<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Match_pattern_with_regular_expressions.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Match+pattern+with+regular+expressions:+Error+short+description">🚨 Bug report</a>

**Tags:** #twilio #project #call #mobile #snippet

**Author:** [Sriniketh Jayasendil](https://twitter.com/srini047/)

**Description:**
It can be exceedingly laborious to get data from sources that are not organised. Regular expressions can be used in Python to match patterns in greater depth, similar to the filtering example above. Using this, textual data can be categorised as part of a data processing workflow, and it can also be used to look for specific keywords in user-submitted content.

## Input

### Import librairies


```python
import re
try:
    import phonenumbers
except ModuleNotFoundError:
    !pip install phonenumbers
    import phonenumbers
```

### Setup variables
You can build and test regular expression patterns using this [regex101.com](https://regex101.com/)


```python
email_regex = '''^[a-z0-9]+[\._]?[a-z0-9]+[@]\w+[.]\w{2,3}$'''   # Pattern to match Email Address

phone_number_regex = ''''''   # Pattern to match Phone Number
```

## Model

### Check email function


```python
def check_email(email):
    if (re.fullmatch(email_regex, email)):
        print('Valid Email✅')
    else:
        print('Invalid Email❌')
```

### Check phone number function


```python
def check_phone_number(phone_number):
    phone_number = phonenumbers.parse(phone_number)
    possible = phonenumbers.is_valid_number(phone_number)
    if (possible):
        if (phonenumbers.is_possible_number(phone_number)):
            print('Valid Phone Number✅')
    else:
        print('Invalid Phone Number❌')
```

## Output

### Check email and phone number


```python
input_email = input('Enter the Email-ID: ')
check_email(input_email)

input_phone_number = input('Enter the Phone Number: ')
check_phone_number(input_phone_number)
```
