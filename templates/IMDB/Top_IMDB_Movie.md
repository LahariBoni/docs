<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/IMDB/Top_IMDB_Movie.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=IMDB+-+Top++Movie:+Error+short+description">🚨 Bug report</a>

**Tags:** #imdb #python #webscraping #imdb #analytics #operations #csv

**Author:** [Oketunji Oludolapo](https://www.linkedin.com/in/oludolapo-oketunji/)

This notebook will help you in getting the top movies on IMDB by genre

## Input

### Import libraries


```python
try:
    import scrapy
except:
    !pip install scrapy
    import scrapy
from scrapy.crawler import CrawlerProcess
from scrapy.crawler import CrawlerRunner
try:
    from crochet import setup, wait_for
except:
    !pip install crochet
    from crochet import setup, wait_for
setup()
```

## Model


```python
class IMDB(scrapy.Spider):
    """The scraping class"""
    name="movies"
    # Writing the output to a csv file and saving it as sample.csv
    custom_settings = {'FEEDS': {'sample.csv': {"format": 'csv','overwrite': True}}}
    
    def start_requests(self):
        """The start requests method that holds the url and processes it then send it to the parse method"""
        urls=["https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=action",
        "https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=adventure",
        "https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=animation",
        "https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=fantasy",
        "https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=romance"]
        for url in urls:
            yield scrapy.Request(url=url,callback=self.parse)
            
    def parse(self, response):
        """The method that is used for subseting and scraping the websites into acceptable formats""" 
        movies=response.css("div.lister-item-content")
        for movie in movies:
            items={
            "title" :movie.css("h3.lister-item-header").css("a::text").get(),
            "year":movie.css("span.lister-item-year.text-muted.unbold::text").get().replace("(","").replace(")","").replace("I",""),
            "rating":movie.css("span.certificate::text").get(),
            "duration":movie.css("span.runtime::text").get(),
            "genre":movie.css("span.genre::text").get().strip(),
            "Total vote rating":movie.css("div.inline-block.ratings-imdb-rating>strong::text").get(),
            "Number of votes":movie.css("p.sort-num_votes-visible>span:nth-of-type(2)::text").get(),
            "Director":movie.css("p:nth-of-type(3)>a:nth-of-type(1)::text").get()
            }
            yield items
            #You can delete the next 3 lines if you need just the first page and not all pages.
        next_page=response.css("div.desc>a::attr(href)").get()
        if next_page is not None:
            yield response.follow(next_page,callback=self.parse)
@wait_for(10) # To avoid reactor time error
def run_spider():
    """run spider"""
    crawler = CrawlerRunner()
    result = crawler.crawl(IMDB)
    return result
```

## Output


```python
run_spider()
```
