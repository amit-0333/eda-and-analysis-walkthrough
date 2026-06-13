# 📥 Step 2a: Gathering Data

The first part of data wrangling is gathering data from various sources. This notebook/file documents how to collect data from common sources used in real-world data analysis.

---

## 🎯 Goal of This Step

- Identify where the data is coming from
- Load data into a usable format (mainly Pandas DataFrames)
- Document source, format, and any access requirements
- Save a raw copy of the data before any modification (never overwrite raw data)

---

## 📂 1. Reading from CSV / Text Files

```python
import pandas as pd

# Basic read
df = pd.read_csv('data.csv')

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

## 📂 2. Reading JSON Data

```python
import pandas as pd
import json

# From a JSON file
df = pd.read_json('data.json')

# Nested JSON -> flatten
with open('data.json') as f:
    raw = json.load(f)

df = pd.json_normalize(raw, record_path=['key_to_records'])
```

---

## 📂 3. Reading from SQL Databases

```python
import pandas as pd
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

## 📂 4. Fetching Data from an API

```python
import requests
import pandas as pd

response = requests.get('https://api.example.com/data')
data = response.json()

df = pd.json_normalize(data)
```

**Things to document:**
- API endpoint and authentication method (API key, OAuth, etc.)
- Rate limits
- Pagination handling (looping over pages)

---

## 📂 5. Web Scraping to DataFrame

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

url = 'https://example.com/table-page'
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# Example: extracting an HTML table directly
tables = pd.read_html(response.text)
df = tables[0]
```

**Things to document:**
- Source URL and date scraped
- robots.txt / terms of use check
- Structure of the HTML being parsed (in case site changes)

---

## 📂 6. Reading Excel and Other Formats via `pandas.io`

```python
# Excel
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Multiple sheets
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)  # dict of DataFrames

# Parquet
df = pd.read_parquet('data.parquet')

# HTML tables
df_list = pd.read_html('https://example.com/page')

# Clipboard (quick paste-in)
df = pd.read_clipboard()
```

---

## 🗂️ Documenting Each Data Source

For every dataset gathered, record:

| Field | Details |
|---|---|
| **Source name** | |
| **Source type** | CSV / JSON / SQL / API / Web scrape / Excel |
| **URL / Path** | |
| **Date accessed** | |
| **Access method / credentials needed** | |
| **Rows x Columns (raw)** | |
| **Known issues at this stage** | |

---

## 💾 Saving Raw Data

Always save an untouched copy of the gathered data before moving to assessment/cleaning:

```python
df.to_csv('data/raw/dataset_raw.csv', index=False)
```

---

## ➡️ Next Step

Proceed to **2b: Assessing Data** to evaluate data quality, structure, and identify issues to clean.
