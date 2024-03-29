#  Simple ETL-Pipeline
the purpose of this project is to make a simple ETL pipeline that can extract data in various form (DB, CSV, WEB) and transform it into a ready to use data in a database
## Preparation
1. installing docker from https://docs.docker.com/desktop/
2. installing ubuntu for cron scheduling
   1. Step 1: Install WSL.
  In the Windows Search bar, type “turn Windows features on or off” and then open the corresponding app.
  Scroll down to check the boxes in front of Virtual Machine Platform and Windows Subsystem for Linux. Then, click OK.
  2. Install ubuntu
     open windows powershell and copy this line
      ```
     wsl --install -d ubuntu
      ```
3. installing python (will be used as primary coding languange here)
## Initial leads
### 1. Sales team
Sales team have data stored in a postgresql database with some missing values and incorect formats
### 2. marketing team
marketing team have data stored in CSV but needs celaning
### 3. data science
data science team need data scraping from a web for NLP
## requirement gathering and proposed solution
### problems
1. inital leads from sales and marketing teams need to be transformed into a clean and usable data
2. data science team have no data or way to find data for NLP
### solutions
1. Creating web scraping methods for data science team to gather their data
2. make ETL Pipeline to futher extracing data, transform the existing data into clean and usable data frame, and load it into new or existing DB
3. scheduling to keep db udpated
## ETL Design
![Untitled-2024-03-10-2054](https://github.com/frhnkl/ETL-Pipeline/assets/125452431/6dc6cbfb-5ac1-4e2f-8d5b-5dc39a088251)
We ae using base **Extract - tranform - load** concept of data engineering. The ide is that we will extract the data fron various sources, transform it into a usable data, and load it into a new or existing database.
1. **Extracting** - data from Database, CSV  will be extracted using pandas and luigi. Meanwhile, the needed data for data science team will be scrapped from amazon using python. All data from those 3 source will be extracted into a dataframe using pandas and luigi.
2. **validation** - the extracted data will undergo validation process where we will search for missing values and wrong table formats
3. **Transform** - we will use pandas to clean the data from missing values, bad tables, and other things
4. **Load** - the datframes will be loaded into new postgresql db
## ETL Implementation
### Web Scraping
in this case, we will use it to scrape "laptop" in amazon, so we will scrape anything with "laptop" on amazon with this code:
```
import requests
from bs4 import BeautifulSoup
import pandas as pd

def scrape_amazon(keyword, page):
    url = f"https://www.amazon.com/s?k={keyword}&page={page}"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36"
    }

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, "html.parser")
        product_titles = [title.text.strip() for title in soup.find_all("span", {"class": "a-size-medium a-color-base a-text-normal"})]
        product_prices = [price.text.strip() for price in soup.find_all("span", {"class": "a-price-whole"})]
        product_urls = ["https://www.amazon.com" + link.get("href") for link in soup.find_all("a", {"class": "a-link-normal a-text-normal"}, href=True)]

        df = pd.DataFrame({"Title": product_titles, "Price": product_prices, "URL": product_urls})
        return df
    else:
        print("Failed to fetch the page.")
        return None

keyword = "laptop"
page = 8
scraped_df = scrape_amazon(keyword, page)

if scraped_df is not None:
    folder_path = "d:/project/etl/data/raw/"
    file_path = folder_path + "etl_scrape.csv"
    scraped_df.to_csv(file_path, index=False)
    print(f"Scraped data saved to: {file_path}")


```
### extracting csv
we will use this code to extract csv into a dataframe
```
import pandas as pd

def extract_data(file_path):
    try:
        df = pd.read_csv(file_path)
        return df
    except FileNotFoundError:
        print(f"File {file_path} not found.")
        return None

file_path = "d:/project/etl/data/raw/products_data.csv"
extracted_data = extract_data(file_path)

if extracted_data is not None:
    print("Data extracted successfully:")
    print(extracted_data.head())  # Display the first few rows of the extracted data
```
