# 📱 Smartprix Mobile Phone Data — Web Scraping, Cleaning, EDA & Feature Engineering

A complete end-to-end mini project: scraping smartphone listings from Smartprix using **Selenium + BeautifulSoup**, cleaning the raw scraped data, and then performing EDA and feature engineering on the cleaned dataset.

---

## 🎯 Project Goal

- Scrape live smartphone listing data from Smartprix (price, specs, brand, ratings, etc.)
- Clean the raw, messy scraped data into a structured, analysis-ready dataset
- Explore the cleaned dataset through univariate, bivariate, and multivariate analysis
- Engineer/select features relevant to predicting or understanding phone `price`

---

## 🕸️ Part 1: Web Scraping (Selenium + BeautifulSoup)

Smartprix's listing pages are JavaScript-rendered, so a simple `requests` call isn't enough — **Selenium** is used to load the page (and handle "load more" / pagination), and **BeautifulSoup** is used to parse the rendered HTML and extract structured fields.

### General Approach

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import time
import pandas as pd

driver = webdriver.Chrome()
driver.get('https://www.smartprix.com/mobiles')

# Scroll / click "Load More" to load additional listings
for _ in range(20):
    try:
        load_more = driver.find_element(By.CLASS_NAME, 'sm-load-more')
        load_more.click()
        time.sleep(2)
    except Exception:
        break

soup = BeautifulSoup(driver.page_source, 'html.parser')
driver.quit()
```

### Extracting Listing Cards

```python
listings = soup.find_all('div', class_='sm-product')

data = []
for item in listings:
    name = item.find('h2').text.strip() if item.find('h2') else None
    price = item.find('span', class_='price').text.strip() if item.find('span', class_='price') else None
    rating = item.find('div', class_='score').text.strip() if item.find('div', class_='score') else None
    specs = item.find('ul', class_='sm-feat-list')
    spec_list = [li.text.strip() for li in specs.find_all('li')] if specs else []

    data.append({
        'name': name,
        'price': price,
        'rating': rating,
        'specs': spec_list
    })

df_raw = pd.DataFrame(data)
df_raw.to_csv('smartphone_raw.csv', index=False)
```

**Things documented during scraping:**
- Source URL: Smartprix mobiles listing page
- Date scraped
- robots.txt / terms of use checked before scraping
- Page structure noted (class names used) in case the site layout changes later

---

## 🧹 Part 2: Cleaning the Raw Scraped Data

Raw scraped data is typically messy: prices as strings with currency symbols, specs combined into a single free-text field, missing values where listings have incomplete information, and inconsistent formatting across brands.

### Common Cleaning Steps Applied

```python
df = df_raw.copy()

# Clean price: remove currency symbols/commas, convert to numeric
df['price'] = df['price'].str.replace('₹', '').str.replace(',', '').astype(float)

# Clean rating: extract numeric rating
df['rating'] = df['rating'].str.extract(r'([\d.]+)').astype(float)

# Split combined 'name' into brand_name and model
df['brand_name'] = df['name'].str.split(' ').str.get(0).str.lower()
df['model'] = df['name'].str.split(' ').str.slice(1).str.join(' ')

# Parse the free-text specs list into individual structured columns
def extract_spec(spec_list, keyword):
    for spec in spec_list:
        if keyword.lower() in spec.lower():
            return spec
    return None

df['ram_capacity'] = df['specs'].apply(lambda x: extract_spec(x, 'RAM'))
df['battery'] = df['specs'].apply(lambda x: extract_spec(x, 'mAh'))
df['processor_brand'] = df['specs'].apply(lambda x: extract_spec(x, 'Processor'))

# Convert boolean-style features
df['has_5g'] = df['specs'].apply(lambda x: any('5G' in s for s in x))
df['has_nfc'] = df['specs'].apply(lambda x: any('NFC' in s for s in x))
df['has_ir_blaster'] = df['specs'].apply(lambda x: any('IR Blaster' in s for s in x))

# Drop the now-redundant raw 'name' and 'specs' columns
df.drop(columns=['name', 'specs'], inplace=True)

# Handle missing values
df.isnull().sum()
df.dropna(subset=['price'], inplace=True)   # price is essential, drop if missing
```

The cleaned dataset (`smartphone_cleaned_v5.csv`) was saved once all fields were standardized and structured properly — this became the input for the EDA phase below.

---

## 📊 Part 3: Exploratory Data Analysis

### First Look

```python
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

df = pd.read_csv('smartphone_cleaned_v5.csv')

df.shape
df.head()
df.info()
df.isnull().sum()
```

### Univariate Analysis — Categorical Features

```python
# Top brands
df['brand_name'].value_counts().head(10).plot(kind='bar')

df['brand_name'].value_counts().plot(kind='pie', autopct='%0.1f%%', figsize=(8,8))
plt.ylabel('')
plt.show()

df['model'].nunique()
```

Boolean/spec-based features were each explored with pie charts to see their overall split across the dataset:

```python
df['has_5g'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['has_nfc'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['has_ir_blaster'].value_counts().plot(kind='pie', autopct='%0.1f%%')

df['processor_brand'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['num_cores'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['fast_charging_available'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['ram_capacity'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['internal_memory'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['refresh_rate'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['os'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['extended_memory_available'].value_counts().plot(kind='pie', autopct='%0.1f%%')
```

**Cross-checking a feature against brand:**
```python
df[df['has_ir_blaster'] == True]['brand_name'].value_counts()
```

### Univariate Analysis — Numeric Features

```python
df['price'].describe()
sns.displot(kind='hist', data=df, x='price', kde=True)
df['price'].skew()
sns.boxplot(df['price'])
df[df['price'] > 250000]          # investigate extreme high-end prices
df['price'].isnull().sum()

df['rating'].describe()
sns.displot(kind='hist', data=df, x='rating', kde=True)
df['rating'].skew()
sns.boxplot(df['rating'])
df['rating'].isnull().sum() / len(df)
```

### Looping Through Multiple Numeric Columns

```python
def plot_graphs(column_name):
    sns.displot(kind='hist', kde=True, data=df, x=column_name, label=column_name)
    sns.catplot(kind='box', data=df, x=column_name)

num_columns = df.select_dtypes(include=['float64', 'int64']).iloc[:, [3,4,6,9,13,14,16]].columns

for col in num_columns:
    plot_graphs(col)
```

### Bivariate Analysis — Categorical vs Price

```python
plt.figure(figsize=(20,10))
sns.barplot(data=df, x='brand_name', y='price', palette='viridis')
plt.xticks(rotation='vertical')
plt.show()

# Focus only on brands with a reasonable sample size (>10 listings)
x = df.groupby('brand_name').count()['model']
temp_df = df[df['brand_name'].isin(x[x > 10].index)]

plt.figure(figsize=(15,8))
sns.barplot(data=temp_df, x='brand_name', y='price')
plt.xticks(rotation='vertical')
plt.show()
```

### Bivariate Analysis — Numeric vs Price

```python
sns.scatterplot(data=df, x='rating', y='price')
sns.scatterplot(data=df, x='processor_speed', y='price')
sns.scatterplot(data=df, x='screen_size', y='price')
```

### Bivariate Analysis — Boolean Specs vs Price

```python
sns.barplot(data=temp_df, x='has_5g', y='price', estimator=np.median)
sns.pointplot(data=temp_df, x='has_nfc', y='price', estimator=np.median)
sns.barplot(data=temp_df, x='has_ir_blaster', y='price', estimator=np.median)
sns.barplot(data=temp_df, x='processor_brand', y='price', estimator=np.median)
plt.xticks(rotation='vertical')

sns.barplot(data=temp_df, x='num_cores', y='price', estimator=np.median)
plt.xticks(rotation='vertical')
```

Using `estimator=np.median` instead of the default mean makes the comparison more robust to outlier-priced flagship phones.

### Cross-tab Between Categorical Features

```python
pd.crosstab(df['num_cores'], df['os'])
```

### Correlation with Target

```python
df.corr(numeric_only=True)['price']
df.corr(numeric_only=True)['rating']
```

---

## 🧪 Part 4: Feature Engineering & Missing Value Imputation

### Combining Camera Counts into a Single Feature

```python
(df['num_rear_cameras'] + df['num_front_cameras']).value_counts().plot(kind='pie', autopct='%0.1f%%')
```

### KNN Imputation for Missing Numeric Values

Rather than dropping rows with missing numeric specs, a **KNN Imputer** was used to estimate missing values based on similar phones:

```python
from sklearn.impute import KNNImputer

x_df = df.select_dtypes(include=['int64', 'float64']).drop(columns='price')

imputer = KNNImputer(n_neighbors=5)
x_df_values = imputer.fit_transform(x_df)

x_df = pd.DataFrame(x_df_values, columns=x_df.columns)
x_df['price'] = df['price']
```

### Validating Imputation Impact on Correlation with Price

```python
a = x_df.corr()['price'].reset_index()
b = df.corr(numeric_only=True)['price'].reset_index()

b.merge(a, on='index')
```

This compares correlation-with-price **before vs after** imputation, to confirm the imputed values didn't distort relationships in the data.

### One-Hot Encoding Categorical Features for Correlation Analysis

```python
pd.get_dummies(df, columns=['brand_name', 'processor_brand', 'os'], drop_first=True).corr(numeric_only=True)['price']
```

This converts key categorical columns into dummy variables so their relationship with `price` can be measured numerically alongside the other features.

---

## 📋 Key Features Used/Engineered

| Feature | Source | Notes |
|---|---|---|
| `brand_name`, `model` | Split from raw `name` | Brand-level price comparison |
| `price` | Cleaned from raw string | Currency symbols/commas removed |
| `rating` | Extracted numeric value | Some missing values |
| `has_5g`, `has_nfc`, `has_ir_blaster` | Parsed from specs text | Boolean flags |
| `processor_brand`, `num_cores` | Parsed from specs text | Strong relationship with price |
| `ram_capacity`, `internal_memory` | Parsed from specs text | Imputed where missing (KNN) |
| `refresh_rate`, `screen_size` | Parsed from specs text | Explored vs price |
| `num_rear_cameras` + `num_front_cameras` | Combined feature | Total camera count |
| One-hot encoded `brand_name`/`processor_brand`/`os` | Engineered | For correlation with price |

---

## 🔍 Key Insights

- `price` is right-skewed with a long tail of high-end flagship phones (confirmed via skew + box plot)
- Brand has a visible effect on price, but only meaningful when restricted to brands with sufficient listings (>10)
- `has_5g`, `processor_brand`, and `num_cores` show noticeable median price differences
- Numeric specs (`processor_speed`, `screen_size`, `rating`) show varying degrees of correlation with `price`
- KNN imputation preserved the overall correlation structure with `price`, validating its use over dropping rows

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Selenium | Rendering and interacting with JS-loaded pages |
| BeautifulSoup | Parsing HTML and extracting listing data |
| Pandas | Cleaning, wrangling, feature engineering |
| Matplotlib / Seaborn | Visualization |
| Scikit-learn (KNNImputer) | Missing value imputation |

---

## ➡️ Possible Next Steps

- Build a price-prediction model using the engineered features
- Expand scraping to track price history over time
- Add more brands/categories for broader comparison
