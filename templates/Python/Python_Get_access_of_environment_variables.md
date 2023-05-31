<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Python/Python_Get_access_of_environment_variables.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Python+-+Get+access+of+environment+variables:+Error+short+description">Bug report</a> | <a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Naas/Naas_Start_data_product.ipynb" target="_parent">Generate Data Product</a>

**Tags:** #python #environment #variables #os #environ #object

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

**Description:** This notebook explains how to use of `os.environ` to get access of environment variables. Environment variables are useful in several ways:
- Portability: Environment variables are independent of the code and can be easily ported across systems. This means that you can define an environment variable once, and use it in multiple scripts, applications, or systems without having to hardcode the variable's value.
- Security: Environment variables can be used to store sensitive information like passwords or API keys, which can be accessed by the code without exposing them in the script. This is particularly useful when you need to deploy your code to a server or share it with others.
- Flexibility: Environment variables allow you to change the behavior of your code without modifying the code itself. This is useful when you need to modify the behavior of your code based on different scenarios, such as development, testing, or production environments.
- Configuration: Environment variables can be used to configure your code, by providing default values for variables that can be overridden by environment variables. This makes your code more flexible and easier to customize.

Overall, environment variables are a useful tool for managing configuration settings and sensitive information in your code, and can make your code more portable, secure, and flexible.

**References:**
- [Python os.environ object](https://www.geeksforgeeks.org/python-os-environ-object/)
- [Python os.environ documentation](https://docs.python.org/3/library/os.html#os.environ)

## Input

### Import libraries


```python
import os
import pprint
```

## Model

### Get the list of user's environment variables


```python
# Get the list of user's environment variables
env_var = os.environ
```

## Output

### Display result


```python
# Print the list of user's
# environment variables
print("User's Environment variable:")
pprint.pprint(dict(env_var), width = 1)
```

 