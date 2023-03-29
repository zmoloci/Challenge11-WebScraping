# Web Scraping Challenge
### Module 11 

#### After learning how to identify HTML elements on a page, identify their `id` and `class` attributes, and use that knowledge to extract information via both automated browsing with [Splinter](https://splinter.readthedocs.io/en/latest/) and HTML parsing with [Beautiful Soup](https://beautiful-soup-4.readthedocs.io/en/latest/), this challenge involves scraping the titles and preview text from a list of [Mars news articles](https://static.bc-edx.com/data/web/mars_news/index.html) and then scraping [Mars weather data](https://static.bc-edx.com/data/web/mars_facts/temperature.html) from a table on the [edX](https://www.edx.org/)-maintained website: [The Mars News](https://static.bc-edx.com/data/web/mars_news/index.html).

----

## Deliverable 1: Scrape Titles and Preview Text from Mars News

### Step 1: Visit the Website
</br>
Load all dependencies, activate the browser and navigate to the [The Mars News](https://static.bc-edx.com/data/web/mars_news/index.html) website:

```
from splinter import Browser
from bs4 import BeautifulSoup as soup
from pprint import pprint
import json

browser = Browser('chrome')

url = 'https://static.bc-edx.com/data/web/mars_news/index.html'
browser.visit(url)

```

### Step 2: Scrape the Website
</br>
Create a [Beautiful Soup](https://beautiful-soup-4.readthedocs.io/en/latest/) object, use it to extract text elements from [The Mars News](https://static.bc-edx.com/data/web/mars_news/index.html) website and then extract the article titles and preview text for each article listed:

```
html = browser.html

mars_soup = soup(html, 'html.parser')

text_elem = mars_soup.find_all('div', class_='col-md-8')
pprint(text_elem)

for i in range(len(text_elem)):
    try:
        print(text_elem[i].find('div', class_="content_title").text)
    except:
        print(f'Failed to return content_title text for text_elem[{i}]')


for i in range(len(text_elem)):
    try:
        print(text_elem[i].find('div', class_="article_teaser_body").text)
    except:
        print(f'Failed to return article_teaser_body text for text_elem[{i}]')
```

### Step 3: Store the Results
</br>
Create an empty list, loop through the text elements, extract the title and preview text, store each title/preview pair in a dictionary and add it to the empty list:

```
titles_n_prevs_list = []

for text in text_elem:

    titles = text.find('div', class_='content_title').text
    previews = text.find('div', class_='article_teaser_body').text

    titles_n_prevs = {"title":titles,
                      "preview":previews}
    titles_n_prevs_list.append(titles_n_prevs)


```

Then display the results, close the browser and export the dictionary as a JSON file formatted UTF-8 with endentations:

```
pprint(titles_n_prevs_list)

browser.quit()

with open("mars_titles_n_prevs.json", "w", encoding='utf-8') as outfile: json.dump(titles_n_prevs_list, outfile,ensure_ascii=False,indent=4)
```

## Deliverable 2: Scrape and Analyze Mars Weather Data

