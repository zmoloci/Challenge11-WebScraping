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

### Step 1: Visit the Website
</br>
Load all dependencies, activate the browser and navigate to the [Mars weather data](https://static.bc-edx.com/data/web/mars_facts/temperature.html) website:

```

from splinter import Browser
from bs4 import BeautifulSoup as soup
import matplotlib.pyplot as plt
import pandas as pd
from pprint import pprint

browser = Browser('chrome')

url = 'https://static.bc-edx.com/data/web/mars_facts/temperature.html'
browser.visit(url)

```

### Step 2: Scrape the Table
</br>
Create a [Beautiful Soup](https://beautiful-soup-4.readthedocs.io/en/latest/) object and then use it to scrape all of the data in the HTML table. Display all of the scraped rows from the 'tr' tags with class = 'data-row'.


```
html = browser.html

mars_soup = soup(html, 'html.parser')

table_elem = mars_soup.find_all('tr', class_='data-row')
pprint(table_elem)

```


### Step 3: Store the Data
</br>
Store the data into a Pandas DataFrame ensuring that the column headings match those on the site.
</br>
The column headings should be:

- **id**: the identification number of a single transmission from the Curiosity rover
- **terrestrial_date**: the date on Earth
- **sol**: the number of elapsed sols (Martian days) since Curiosity landed on Mars
- **ls**: the solar longitude
- **month**: the Martian month
- **min_temp**: the minimum temperature, in Celsius, of a single Martian day (sol)
- **pressure**: The atmospheric pressure at Curiosity's location
</br>

To do this, first create an empty list and then use 'for loops' to populate the list with lists of the values for each row in the table:

```
rows = []

for i in range(len(table_elem)):
    try:
        row_it = []
        for j in range(len(table_elem[i].find_all('td'))):
            row_it.append((table_elem[i].find_all('td')[j]).text)
        rows.append(row_it)    
    except:
        print(f'Did not store row {i}')
print(rows)


```

</br>
Then create a dataframe from this list:

```
columns = ['id','terrestrial_date','sol','ls','month','min_temp','pressure']

table_df = pd.DataFrame(rows, columns=columns)


```
</br>
...and display it:

```
print(table_df)

```

### Step 4: Prepare Data for Analysis
</br>
Check data types for each column and cast (or convert) to datetime, int, or float where necessary.

```
table_df.dtypes

table_df.id = table_df.id.astype(int)
table_df['terrestrial_date'] = pd.to_datetime(table_df['terrestrial_date'], format='%Y-%m-%d')
table_df.sol = table_df.sol.astype(int)
table_df.ls = table_df.ls.astype(int)
table_df.month = table_df.month.astype(int)
table_df.min_temp = table_df.min_temp.astype(float)
table_df.pressure = table_df.pressure.astype(float)

table_df.dtypes
```
### Step 5: Analyze the Data
</br>
Using [Pandas](https://pandas.pydata.org/pandas-docs/stable/) functions the following questions are answered:

1. How many months exist on Mars?
```
table_df['month'].max()
```
2. How many Martian (and not Earth) days worth of data exist in the scraped dataset?
```
table_df['sol'].count()
```
3. What are the coldest and the warmest months on Mars (at the location of Curiosity)?</br> To answer this question:
- Find the average of the minimum daily temperature for all of the months.
```
avg_min_temp_df = table_df.groupby(['month']).agg(mean_min_temp = ('min_temp','mean'))
avg_min_temp_df
```
OUTPUT:
</br>
![fig.1_mean_min_temp_by_month](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.1_mean_min_temp_by_month.png)

- Plot the results as a bar chart.</br>
```
ax = avg_min_temp_df.plot.bar(y='mean_min_temp')
ax.set_ylabel('mean min_temp')
ax.set_title('Mean Mars Minimum Temp by Month')
```
OUTPUT:
</br>
![fig.2_mean_min_temp_by_month_plot](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.2_mean_min_temp_by_month_plot.png)

- Identify the coldest and hottest months in Curiosity's location:
```
mintemp=avg_min_temp_df['mean_min_temp'].min()
maxtemp=avg_min_temp_df['mean_min_temp'].max()

min_temp_df = avg_min_temp_df[avg_min_temp_df['mean_min_temp']==mintemp]
max_temp_df = avg_min_temp_df[avg_min_temp_df['mean_min_temp']==maxtemp]
print('Minimum Mean Temp Month:')
print(min_temp_df)
print('------------------------')
print('Maximum Mean Temp Month:')
print(max_temp_df)
```
OUTPUT:
</br>
![fig.3_hot_cold_mean_temp_month](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.3_hot_cold_mean_temp_month.png)

4. Which months have the lowest and the highest atmospheric pressure on Mars?</br> To answer this question:
- Find the average the daily atmospheric pressure of all the months.
```
avg_pressure_df = table_df.groupby(['month']).agg(mean_pressure = ('pressure','mean'))
avg_pressure_df
```
OUTPUT:
</br>
![fig.5_mean_pressure_by_month](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.5_mean_pressure_by_month.png)


- Plot the results as a bar chart.

```
ax = avg_pressure_df.plot.bar(y='mean_pressure')
ax.set_ylabel('mean pressure')
ax.set_title('Mean Mars Pressure by Month')
```
OUTPUT:
</br>
![fig.4_mean_pressure_by_month_plot](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.4_mean_pressure_by_month_plot.png)


5. About how many terrestrial (Earth) days exist in a Martian year? </br>To answer this question:
- Consider how many days elapse on Earth in the time that Mars circles the Sun once.
```

for i in range(len(table_df)):
    if table_df.iloc[i,4] == 1:
        terr_day_one = table_df.iloc[i,1]
        mars_day_one = i
        break

for j in range(mars_day_one,len(table_df)):
    if table_df.iloc[j,4] == 12:
        mars_12mo_day = j
        break    
        
for k in range(mars_12mo_day,len(table_df)):
    if table_df.iloc[k,4] == 1:
        terr_day_x = table_df.iloc[k,1]
        break
    
terr_days_in_mar_yr = terr_day_x - terr_day_one
terr_days_in_mar_yr.days


print(f'Approximately {terr_days_in_mar_yr} elapse on Earth in 1 year on Mars')

```
- Visually estimate the result by plotting the daily minimum temperature.
```
table_df_one_mar_yr = table_df.loc[i:k]

x = table_df_one_mar_yr['sol']
y = table_df_one_mar_yr['min_temp']

ax=plt.subplot(1,1,1)
ax.plot(x,y)
ax.set_ylabel('daily min_temp')
ax.set_xlabel('sol')
ax.set_title('Mars Daily Minimum Temp for 1 Year')

ax.set_xticks(x[::25])
ax.set_xticklabels(x[::25],rotation=90)


```
OUTPUT:
</br>
![fig.6_min_temp_1_year.png](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/Figures/fig.6_min_temp_1_year.png)

### Step 6: Save the Data
</br>

Export dataframe to a [CSV file](https://github.com/zmoloci/Challenge11-WebScraping/blob/main/mars_output.csv) and quit the browser.

```
table_df.to_csv('mars_output.csv')

browser.quit()

```
