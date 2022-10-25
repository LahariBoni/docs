<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/LinkedIn/LinkedIn_Extract_content_world_cloud.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=LinkedIn+-+Extract+content+world+cloud:+Error+short+description">🚨 Bug report</a>

**Tags:** #linkedin #worldcloud #content #analytics #dependency

**Author:** [Florent Ravenel](https://www.linkedin.com/in/florent-ravenel/)

This notebook extracts the most used world in your LinkedIn posts.

<div class="alert alert-info" role="info" style="margin: 10px">
<b>Requirements:</b><br>
To run this notebook, you must have already run <b>LinkedIn_Get_profile_posts_stats.ipynb</b> or <b>LinkedIn_Get_company_posts_stats.ipynb</b> to get your post stats in CSV.<br>
</div>

## Input

### Import libraries


```python
try:
    from wordcloud import WordCloud
except:
    !pip install wordcloud --user
    from wordcloud import WordCloud
import matplotlib.pyplot as plt
import pandas as pd
```

### Setup Variables


```python
# Input
csv_input = f"LINKEDIN_PROFILE_POSTS.csv" # CSV path with your posts stats generated with 'LinkedIn_Get_profile_posts_stats.ipynb' or 'LinkedIn_Get_company_posts_stats.ipynb'

# Outputs
name_output = "LINKEDIN_CONTENT_WORLD_CLOUD"
image_output = f"{name_output}.png"
```

### Setup Naas dependency


```python
naas.dependency.add()

#-> Uncomment the line below to remove your dependency
# naas.dependency.delete()
```

## Model

### Get your posts
Get posts feed from CSV stored in your local (Returns empty if CSV does not exist)


```python
def read_csv(file_path):
    try:
        df = pd.read_csv(file_path)
    except FileNotFoundError as e:
        # Empty dataframe returned
        return pd.DataFrame()
    return df

df_posts = read_csv(csv_input)
print("✅ Posts fetched:", len(df_posts))
df_posts.head(1)
```

### Create trend dataframe


```python
#Creating the text variable
text = " ".join(text for text in df_posts.astype(str).TEXT)
```


```python
# Creating word_cloud with text as argument in .generate() method
word_cloud = WordCloud(collocations=False,
                       background_color='white',
                       width=1200,
                       height=600).generate(text)
```


```python
%matplotlib inline

# Display the generated Word Cloud

plt.imshow(word_cloud, interpolation='bilinear')

plt.axis("off")

plt.show()
```

## Output

### Save and share your graph in image



```python
# Save your image in PNG
word_cloud.to_file(image_output)

# Share output with naas
naas.asset.add(image_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete(image_output)
```
