---
description: Expose assets by URL.
---

# 🖼️ Asset

## Add

Copy in production this current file as an asset \(file\) and allow yourself to get it by calling the returned URL.

```python
naas.asset.add()
```

You will get URL who downloads this current file when you open it.

![screenshot-add-asset](../.gitbook/assets/screenshot-2020-10-07-at-18.34.12.png)

### Other files

If you want to add another file than the current one, give a path:

```python
naas.asset.add("path/to/my/super/data.csv")
```

### Parameters

`inline` : Get a response in your web browser instead of downloading the result.

```python
params = {"inline": True}

naas.asset.add(params=params)
```

### Debug

```python
naas.asset.add(debug=True)
```

## Find

You can find url of a file pushed into the production:

### Current file

```python
url = naas.asset.find()
```

### Other files 

```python
url = naas.asset.find(path="path/to/my/super/notebook.ipynb")
```

## List 

You can list all version of a file pushed into the production:

### Current file

```python
naas.asset.list()
```

### Other files 

```python
naas.asset.list(path="path/to/my/super/data.csv")
```

## Get 

You can get a version of a file pushed into the production:

### Get the last one

```python
naas.asset.get()
```

### With a file path

```python
naas.asset.get(path="path/to/my/super/data.csv")
```

### With history id

```python
naas.asset.get(histo="20201008101221879662")
```

### Combined

```python
naas.asset.get(path="path/to/my/super/data.csv", histo="20201008101221879662")
```

## Clear

You can clear the previous version of a file pushed into the production:

### One

```python
naas.asset.clear(histo="20201008101221879662")
```

### Other Notebook

```python
naas.asset.clear(path="path/to/my/super/data.csv", histo="20201008101221879662")
```

### All

```python
naas.asset.clear()
```

### All for filepath

```python
naas.asset.clear(path="path/to/my/super/data.csv")
```

## Delete

You can remove any scheduler capability like that, it takes optionally a path.

### Current

```python
naas.asset.delete()
```

### Other file

```python
naas.asset.delete(path="path/to/my/super/data.csv")
```

### Debug

```python
naas.asset.delete(debug=True)
```

## List Assets

You don't remember how many Assets notebook you have?

### Simple

```python
naas.asset.currents()
```

### Raw result 

```python
naas.asset.currents(raw=True)
```

