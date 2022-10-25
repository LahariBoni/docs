<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Find_the_stock_with_closest_performance_using_KNN.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Find+the+stock+with+closest+performance+using+KNN:+Error+short+description">🚨 Bug report</a>

**Tags:** #tool #naas_drivers #naas #scheduler #asset #snippet #automation #ai #analytics #yahoo #clustering #stocks

**Author:** [Abhinav Lakhani](https://www.linkedin.com/in/abhinav-lakhani/)

Given a list of stocks data(tickers), find the stock(s) with the closest performance attribute, make a clustering Graph of clusters (with each cluster having a different color) and save the clustered stocks to a .csv as well as the cluster to an image.

Resources / Inspiration:
https://towardsdatascience.com/in-12-minutes-stocks-analysis-with-pandas-and-scikit-learn-a8d8a7b50ee7

## Input

### Import library


```python
from naas_drivers import yahoofinance
import naas
```


```python
from pylab import plot,show
from numpy import vstack,array
from numpy.random import rand
import numpy as np
from scipy.cluster.vq import kmeans,vq
import pandas as pd
from math import sqrt
from sklearn.cluster import KMeans
from matplotlib import pyplot as plt
```

### Variables


```python
# Input
sp500_url = 'https://en.wikipedia.org/wiki/List_of_S%26P_500_companies'
date_from = -3600
date_to = "today"
moving_averages = [20,50]
tickers = []
feature = ["Adj Close"] # which performance attribute do we want to cluster on: Open	High	Low	Close	Adj Close	Volume

# Output
csv_output = "STOCK_CLUSTERS.csv"
img_output = "STOCK_CLUSTERS.png"
```

### Schedule your notebook


```python
# Schedule your notebook everyday at 9 AM
# naas.scheduler.add(cron="0 9 * * *")

#-> Uncomment the line below to remove your scheduler
# naas.scheduler.delete()
```

## Modeling

### Get tickers from Wikipedia


```python
# read in the url and scrape s&p500 tickers
data_table = pd.read_html(sp500_url)
tickers = data_table[0]["Symbol"].tolist()
```

### Get data from yahoo finance


```python
# get stocks info
prices_list = []
for ticker in tickers:
    try:
        prices = yahoofinance.get(ticker,
                      date_from=date_from,
                      date_to=date_to,
                      moving_averages=moving_averages)[feature]
        prices = pd.DataFrame(prices)
        prices.columns = [ticker]
        prices_list.append(prices)
    except:
        pass
    prices_df = pd.concat(prices_list,axis=1)
prices_df.sort_index(inplace=True)
prices_df.head()
```

We can now start to analyse the data and begin our K-Means investigation…

Our first decision is to choose how many clusters do we actually want to separate our stocks into. Rather than make some arbitrary decision, we can use the “Elbow Method” to highlight the relationship between how many clusters we choose, and the Sum of Squared Errors (SSE) resulting from using that number of clusters.

We then plot this relationship to help us identify the optimal number of clusters to use – we would prefer a lower number of clusters, but also would prefer the SSE to be lower – so this trade off needs to be taken into account.

Lets run the code for our Elbow Curve plot.


```python
#Calculate average annual percentage return and volatilities over a theoretical one year period
returns = prices_df.pct_change().mean() * 252
returns = pd.DataFrame(returns)
returns.columns = ['Returns']
returns['Volatility'] = prices_df.pct_change().std() * sqrt(252)
#format the data as a numpy array to feed into the K-Means algorithm
data = np.asarray([np.asarray(returns['Returns']),np.asarray(returns['Volatility'])]).T
```


```python
# KMeans
X = data
distorsions = []
for k in range(2, 20):
    k_means = KMeans(n_clusters=k)
    k_means.fit(X)
    distorsions.append(k_means.inertia_)
fig = plt.figure(figsize=(15, 5))
plt.plot(range(2, 20), distorsions)
plt.grid(True)
plt.title('Elbow curve')
```

So we can sort of see that once the number of clusters reaches 5 (on the bottom axis), the reduction in the SSE begins to slow down for each increase in cluster number. This would lead me to believe that the optimal number of clusters for this exercise lies around the 5 mark – so let’s use 5.


```python
def get_centroid_labels(centroids):
    m, n = centroids.shape
    labels = ["" for _ in range(m)]
    rank_mode = m // 2
    has_even_n_O_centroids = m % 2 == 0
    
    for i, (returns_rank, volatility_rank) in enumerate(zip(np.argsort(np.argsort(centroids[:, 0])), np.argsort(np.argsort(centroids[:, 1])))):
        # assign return labels
        if returns_rank == 0:
            labels[i] += "lowest returns, "
        elif returns_rank == (m-1):
            labels[i] += "highest returns, "
        else:
            if has_even_n_O_centroids:
                labels[i] += (f"low returns({returns_rank}), " if returns_rank <= rank_mode else f"high returns({m-returns_rank-1}), ")
            else:
                labels[i] += (f"low returns({returns_rank}), " if returns_rank < rank_mode else f"high returns({m-returns_rank-1}), " if returns_rank > rank_mode else "average returns, ")
        # assign volatility labels
        if volatility_rank == 0:
            labels[i] += "stable"
        elif volatility_rank == (m-1):
            labels[i] += "extremly volatile"
        else:
            if has_even_n_O_centroids:
                labels[i] += (f"low volatility({volatility_rank})" if volatility_rank <= rank_mode else f"high volatility({m-volatility_rank-1})")
            else:
                labels[i] += (f"low volatility({volatility_rank})" if volatility_rank < rank_mode else f"high volatility({m-volatility_rank-1})" if volatility_rank > rank_mode else "average volatility")
    return labels
```


```python
# computing K-Means with K = 5 (5 clusters)
n_clusters = 5
centroids,_ = kmeans(data,n_clusters)
cluster_names = get_centroid_labels(centroids)
# assign each sample to a cluster
idx,_ = vq(data,centroids)
returns["cluster_id"] = idx
returns["cluster"] = [cluster_names[i] for i in idx]
# plot the clusters
plt.figure(figsize=(10, 8))
for i, cluster in enumerate(cluster_names):
    cluster_i = returns[returns.cluster_id==i]
    plt.scatter(cluster_i.Returns, cluster_i.Volatility, alpha = 0.6, s=5)
    plt.scatter(centroids[i, 0], centroids[i, 1], label=cluster)
plt.xlabel("returns");plt.ylabel("volatility")
plt.legend()
plt.show()
```

it looks like we have an outlier in the data which is skewing the results and making it difficult to actually see what is going on for all the other stocks. Let’s take the easy route and just drop the outlier from our data set and run this again.


```python
outliers = np.sqrt(returns.Returns**2 + returns.Volatility**2).argsort()[-2:]
outliers = returns.iloc[outliers, :]
outliers
```


```python
#drop the relevant stock from our data
returns.drop(outliers.index,inplace=True)
#recreate data to feed into the algorithm
data = np.asarray([np.asarray(returns['Returns']),np.asarray(returns['Volatility'])]).T
```


```python
# computing K-Means with K = 5 (5 clusters)
n_clusters = 5
centroids,_ = kmeans(data,n_clusters)
cluster_names = get_centroid_labels(centroids)
# assign each sample to a cluster
idx,_ = vq(data,centroids)
returns["cluster_id"] = idx
returns["cluster"] = [cluster_names[i] for i in idx]
# plot the clusters
plt.figure(figsize=(10, 8))
for i, cluster in enumerate(cluster_names):
    cluster_i = returns[returns.cluster_id==i]
    plt.scatter(cluster_i.Returns, cluster_i.Volatility, alpha = 0.6, s=5)
    plt.scatter(centroids[i, 0], centroids[i, 1], label=cluster)
plt.xlabel("returns");plt.ylabel("volatility")
plt.legend()
plt.show()
```

SO there we go, we now have a list of each of the stocks which are close together/similar, along with which one of 5 clusters they belong to with the clusters being defined by their return and volatility characteristics. We also have a visual representation of the clusters in chart format.

## Output


```python
stock_clusters = pd.DataFrame(zip(returns.index,idx), columns=["Ticker", "Group ID"])
stock_clusters
```

### Save result in csv


```python
stock_clusters.to_csv(csv_output, index=False)
```

### Save the cluster plot


```python
fig
```


```python
plt.savefig(img_output)
```

### Share your output with naas


```python
naas.asset.add(csv_output)

#-> Uncomment the line below to remove your asset
# naas.asset.delete()
```


```python

```
