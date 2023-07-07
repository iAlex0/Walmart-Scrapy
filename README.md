# Walmart-Scraper
A Python Scrapy spider that scrapes product data from [Walmart.com](https://www.walmart.com/). 

These scrapers extract the following fields from Walmart product pages:

- Product Type
- Product Name
- Brand
- Average Rating
- Manufacturer Name
- Description
- Image Url
- Price
- Currency
- Etc.


## ScrapeOps Proxy
This Walmart spider uses [ScrapeOps Proxy](https://scrapeops.io/proxy-aggregator/) as the proxy solution. ScrapeOps has a free plan that allows you to make up to 1,000 requests per month which makes it ideal for the development phase, but can be easily scaled up to millions of pages per month if needs be.

You can [sign up for a free API key here](https://scrapeops.io/app/register/main).



## Getting Started
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

1. Clone this project: `git clone https://github.com/iAlex0/Walmart-Scrapy.git`
2. Create a Python Virtual Environment: `python3 -m venv venv`
3. Activate the Python Virtual Environment: `venv/Scripts/activate`
4. Install the project requirements: `pip install -r requirements.txt`
5. Change working directory to the project folder: `cd walmart-python-scrapy-scraper`
6. Add your API key to the settings.py file (https://www.scrapeops.io): 

    - SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

7. Listing the scrapy projects `scrapy list` 
8. Running the scrapy project: `scrapy crawl walmart` (or `scrapy crawl walmart -O results.json` for a JSON output)



### Speeding Up The Crawl
The spiders are set to only use 1 concurrent thread in the settings.py file as the ScrapeOps Free Proxy Plan only gives you 1 concurrent thread.

However, if you upgrade to a paid ScrapeOps Proxy plan you will have more concurrent threads. Then you can increase the concurrency limit in your scraper by updating the `CONCURRENT_REQUESTS` value in your ``settings.py`` file.

```python
# settings.py

CONCURRENT_REQUESTS = 32

```


----------------------------------------------------------------------------------------------------------------------------


## Customizing The Walmart Scraper
The following are instructions on how to modify the Walmart scraper for your particular use case.


### Configuring Walmart Product Search
To change the query parameters for the product search just change the keywords and locations in the `keyword_list` lists in the spider.

For example:

```python

def start_requests(self):
    keyword_list = ['laptop', 'ipad', '']
    for keyword in keyword_list:
        payload = {'q': keyword, 'sort': 'best_seller', 'page': 1, 'affinityOverride': 'default'}
        walmart_search_url = 'https://www.walmart.com/search?' + urlencode(payload)
        yield scrapy.Request(url=walmart_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})

```

You can also change the sorting algorithm to one of: ``best_seller``, `best_match`, `price_low` or `price_high`.

### Extract More/Different Data
The JSON blobs the spiders extract the product data from are pretty big so the spiders are configured to only parse some of the data. 

You can expand or change the data that gets extract by changing the yield statements:

```python

yield {
        'keyword': response.meta['keyword'],
        'page': response.meta['page'],
        'position': response.meta['position'],
        'id':  raw_product_data.get('id'),
        'type':  raw_product_data.get('type'),
        'name':  raw_product_data.get('name'),
        'brand':  raw_product_data.get('brand'),
        'averageRating':  raw_product_data.get('averageRating'),
        'manufacturerName':  raw_product_data.get('manufacturerName'),
        'shortDescription':  raw_product_data.get('shortDescription'),
        'thumbnailUrl':  raw_product_data['imageInfo'].get('thumbnailUrl'),
        'price':  raw_product_data['priceInfo']['currentPrice'].get('price'), 
        'currencyUnit':  raw_product_data['priceInfo']['currentPrice'].get('currencyUnit'),  
    }

```


### Storing Data
The spiders are set to save the scraped data into a CSV file and store it in a data folder using [Scrapy's Feed Export functionality](https://docs.scrapy.org/en/latest/topics/feed-exports.html).

```python

custom_settings = {
        'FEEDS': { 'data/%(name)s_%(time)s.csv': { 'format': 'csv',}}
        }

```

## ScrapeOps Monitoring
To monitor our scraper, this spider uses the [ScrapeOps Monitor](https://scrapeops.io/monitoring-scheduling/), a free monitoring tool specifically designed for web scraping. 

**Live demo here:** [ScrapeOps Demo](https://scrapeops.io/app/login/demo) 


## License
MIT © iAlex0
