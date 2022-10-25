---
Description: Run your notebook, even when you sleep.
---

# ⏰ Scheduler

### Introduction:
Plan and decide when to run your notebook and automate the process with Naas Scheduler to optimize your time to work smart.

Naas scheduler helps to automatically run the notebook and perform the task as per our convenience. 

Here’s an example video to help you run schedulers with the help of Naas: "https://www.youtube.com/watch?v=ONiILHFItzs&t=48s"

Currently, We are utilising CRON tasks to schedule notebooks. Refer the link to know more about the syntax: [https://crontab.guru/](https://crontab.guru/)


Note: By default the scheduler is set to the CET Timezone. If you are in another timezone, you would need to use the Timezone low-code formula to sync your server with your timezone.

Refer the document below to set the timezone according to your location:
[advanced-usecases.md](../advanced/advanced-usecases.md)


## Add

Refer the below script to send your notebook to production and schedule it to run at 9:00 every day.

```python
naas.scheduler.add(cron="0 9 * * *")
```

### Other Notebook

Optionally, we can also specify a path to the function and the script will deploy the newly specified file instead of the current one.

```python
naas.scheduler.add(path="path/to/my/super/notebook.ipynb", cron="0 9 * * *")
```

### Parameters

`notif_down` : Receive an email when the notebook run fails.

`notif_up` : Receive an email when the notebook runs well.

`next_url` : Url to call when the notebook success, it can be any service even if an Notebook API

```python
params = {"notif_down": "bob@naas.ai", "notif_up": "georges@naas.ai"}

naas.scheduler.add(cron="0 9 * * *", params=params)
```

### Debug

```python
naas.scheduler.add(cron="0 9 * * *", debug=True)
```

## Delete

Run the below snippet to remove any scheduler capability like that, it takes optionally a path:&#x20;

```python
naas.scheduler.delete()
```

### Other Notebook

Refer the below snippet to delete any other notebook in a specified path

```python
naas.scheduler.delete(path="path/to/my/super/notebook.ipynb")
```

### Debug

```python
naas.scheduler.delete(debug=True)
```



## List&#x20;

Execute the below scripts to obtain a list of all version of a file pushed into the production folder:

### Current file

```python
naas.scheduler.list()
```

### Other Notebook&#x20;

```python
naas.scheduler.list(path="path/to/my/super/notebook.ipynb")
```

## Get&#x20;

Execute the below code to get a version of a file pushed into the production:

### Get the last one

```python
naas.scheduler.get()
```

### With a file path

```python
naas.scheduler.get(path="path/to/my/super/notebook.ipynb")
```

### With history id

```python
naas.scheduler.get(histo="20201008101221879662")
```

### Combined

```python
naas.scheduler.get(path="path/to/my/super/notebook.ipynb", histo="20201008101221879662")
```

## Clear

You can clear the previous version of a file pushed into the production:

### One

```python
naas.scheduler.clear(histo="20201008101221879662")
```

### Other Notebook

```python
naas.scheduler.clear(path="path/to/my/super/notebook.ipynb", histo="20201008101221879662")
```

### All

```python
naas.scheduler.clear()
```

### All for filepath

```python
naas.scheduler.clear(path="path/to/my/super/notebook.ipynb")
```

## Get output

Execute the snippet get the output of the production file:

### Get the last one

```python
naas.scheduler.get_output()
```

### With a file path

```python
naas.scheduler.get_output(path="path/to/my/super/notebook.ipynb")
```

## Clear output

Run the script to clear the previous output of a file pushed into the production:

### One

```python
naas.scheduler.clear_output()
```

### Other Notebook

```python
naas.scheduler.clear_output(path="path/to/my/super/notebook.ipynb")
```

## List Schedulers

Don't remember how many Scheduled notebooks you have? Simply run the below script to get the list of current Schedulers.

### Simple

```python
naas.scheduler.currents()
```

### Raw result&#x20;

```python
naas.scheduler.currents(raw=True)
```

