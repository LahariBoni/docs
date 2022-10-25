<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/YahooFinance/YahooFinance_Cryptocurrencies_heatmap_correlation_graph.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=YahooFinance+-+Cryptocurrencies+heatmap+correlation+graph:+Error+short+description">🚨 Bug report</a>

**Tags:** #yahoofinance #cryptocurrency #eth #btc #heatmap #finance #trading #investors #snippet #matplotlib

**Author:** [Carlo Occhiena](https://www.linkedin.com/in/carloocchiena/)

Get live data from the web and compute data viz and analysis about different cryptocurrencies.<br> 

## Input

### Install and import libraries


```python
import datetime as dt
import matplotlib.pyplot as plt
try:
    import seaborn as sns
except:
    !pip install seaborn
    import seaborn as sns
try:
    import yfinance as yfin
except:
    !pip install yfinance
    import yfinance as yfin
```

### Setup your variables


```python
# user settings (modify accordingly to Yahoo Finance parameters)
currency = "USD"
metric = "Close"

# Date
start = dt.datetime(2018,1,1)
end = dt.datetime.now()
```

👉 Insert a range of cryptocurrencies; here you'll have correlation and heatmap


```python
# pick your favorite list of cryptocurrencies
crypto = ['BTC', 'ETH', 'LTC', 'XRP', 'DASH', 'SC']
```

## Model

### Get combined data


```python
yfin.pdr_override()

colnames = []

first = True

for ticker in crypto:
    data = yfin.download(f"{ticker}-{currency}", start, end)
    if first:
        combined = data[[metric]].copy()
        colnames.append(ticker)
        combined.columns = colnames
        first = False
    else:
        combined = combined.join(data[metric])
        colnames.append(ticker)
        combined.columns = colnames
        
combined
```

## Output

### Show heatmap and correlation


```python
plt.yscale('log') # first show linear
for ticker in crypto:
    plt.plot(combined[ticker], label=ticker)
    
plt.tick_params(axis="x", width = 2)
plt.xticks(rotation = "vertical", )
plt.margins(0.01)
plt.subplots_adjust(bottom = 0.15)
plt.legend(loc='lower center', bbox_to_anchor=(0.5, 1.05),
          ncol=6, fancybox=True, shadow=False)
plt.show()

# Correlation Heat Map
combined = combined.pct_change().corr(method='pearson')

sns.heatmap(combined, annot=True, cmap="coolwarm")
plt.show()
print(combined)
```
