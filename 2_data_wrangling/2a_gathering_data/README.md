# 📥 Step 2a: Gathering Data

The first part of data wrangling is gathering data from various sources. This notebook/file documents how to **import** data from common sources, **export** data into different formats, and **gather data via APIs and web scraping**.

---

## 🎯 Goal of This Step

- Identify where the data is coming from
- Load data into a usable format (mainly Pandas DataFrames)
- Export/save data into different formats as needed
- Document source, format, and any access requirements
- Save a raw copy of the data before any modification (never overwrite raw data)

---

# 📂 Part 1: Importing Data from Various Sources

## 1. CSV / Text Files

```python
import pandas as pd

# Basic read
df = pd.read_csv('data.csv')

# Reading a plain text file (e.g. tab-separated)
df = pd.read_csv('data.txt', sep='\t')

# Advanced / hidden parameters
df = pd.read_csv(
    'data.csv',
    sep=',',              # delimiter
    header=0,             # row to use as column names
    index_col=0,          # column to use as index
    usecols=['col1','col2'],  # load only specific columns
    dtype={'col1': str},      # force data types
    parse_dates=['date_col'], # parse date columns
    na_values=['NA','?','-'], # custom missing value markers
    skiprows=2,           # skip header/footer rows
    nrows=1000,           # limit rows read
    encoding='utf-8',     # handle encoding issues
    chunksize=10000       # read in chunks for large files
)
```

---

## 2. HTML

```python
# Read all <table> elements from a webpage or HTML string into a list of DataFrames
df_list = pd.read_html('https://example.com/page')
df = df_list[0]   # pick the relevant table
```

---

## 3. Excel

```python
# Single sheet
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Multiple sheets -> dict of DataFrames
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)

# Specific columns / skip rows
df = pd.read_excel('data.xlsx', sheet_name='Sheet1', usecols='A:D', skiprows=1)
```

---

## 4. JSON

```python
import json

# From a JSON file (simple/flat structure)
df = pd.read_json('data.json')

# Nested JSON -> flatten
with open('data.json') as f:
    raw = json.load(f)

df = pd.json_normalize(raw, record_path=['key_to_records'])
```

---

## 5. SQL Databases

```python
import sqlite3
# or: from sqlalchemy import create_engine

# Using sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table_name', conn)

# Using SQLAlchemy (for MySQL/Postgres/etc.)
# engine = create_engine('postgresql://user:pass@host:port/dbname')
# df = pd.read_sql('SELECT * FROM table_name', engine)
```

---

# 📤 Part 2: Exporting Data into Different Formats

Once data is loaded, cleaned, or transformed, it's often useful to export it back into different formats for sharing, storage, or further use.

## 1. CSV

```python
df.to_csv('output.csv', index=False)
```

## 2. Excel

```python
df.to_excel('output.xlsx', sheet_name='Sheet1', index=False)

# Writing multiple DataFrames to multiple sheets
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sheet1', index=False)
    df2.to_excel(writer, sheet_name='Sheet2', index=False)
```

## 3. JSON

```python
df.to_json('output.json', orient='records', indent=2)
```

## 4. HTML

```python
df.to_html('output.html', index=False)
```

## 5. SQL

```python
df.to_sql('table_name', conn, if_exists='replace', index=False)
```

---

# 🌐 Part 3: Gathering Data via API

```python
import requests

response = requests.get('https://api.example.com/data')
data = response.json()

df = pd.json_normalize(data)
```

### Handling Pagination
```python
all_data = []
url = 'https://api.example.com/data'
params = {'page': 1, 'per_page': 100}

while True:
    response = requests.get(url, params=params)
    data = response.json()

    if not data['results']:
        break

    all_data.extend(data['results'])
    params['page'] += 1

df = pd.json_normalize(all_data)
```

### Authentication
```python
headers = {'Authorization': 'Bearer YOUR_API_KEY'}
response = requests.get('https://api.example.com/data', headers=headers)
```

**Things to document:**
- API endpoint and authentication method (API key, OAuth, etc.)
- Rate limits
- Pagination handling (looping over pages)

---

# 🕸️ Part 4: Web Scraping to DataFrame

```python
from bs4 import BeautifulSoup

url = 'https://example.com/table-page'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# Example: extracting an HTML table directly
tables = pd.read_html(response.text)
df = tables[0]
```

### Scraping Specific Elements (when not in a `<table>`)
```python
items = soup.find_all('div', class_='item')

rows = []
for item in items:
    name = item.find('h2').text.strip()
    price = item.find('span', class_='price').text.strip()
    rows.append({'name': name, 'price': price})

df = pd.DataFrame(rows)
```

**Things to document:**
- Source URL and date scraped
- robots.txt / terms of use check
- Structure of the HTML being parsed (in case site changes)

---

# 🗂️ Documenting Each Data Source

For every dataset gathered, record:

| Field | Details |
|---|---|
| **Source name** | |
| **Source type** | CSV / TXT / HTML / Excel / JSON / SQL / API / Web scrape |
| **URL / Path** | |
| **Date accessed** | |
| **Access method / credentials needed** | |
| **Rows x Columns (raw)** | |
| **Known issues at this stage** | |

---

# 💾 Saving Raw Data

Always save an untouched copy of the gathered data before moving to assessment/cleaning:

```python
df.to_csv('data/raw/dataset_raw.csv', index=False)
```

---

## ➡️ Next Step

Proceed to **2b: Assessing Data** to evaluate data quality, structure, and identify issues to clean.
