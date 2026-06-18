
# 📊 Step 3: Exploratory Data Analysis (EDA)

Once data is gathered, assessed, and cleaned (Step 2), EDA is where you actually explore the dataset — understanding distributions, relationships, and patterns, and engineering new features that make the data more useful for analysis.

> EDA is iterative — you rarely go through it once. Insights from one chart often send you back to clean or engineer another feature.

---

## 🎯 Goal of This Step

- Understand the distribution of individual variables (univariate analysis)
- Explore relationships between variables (bivariate / multivariate analysis)
- Detect outliers, skewness, and missing data patterns visually
- Engineer new features that better capture information for analysis
- Build intuition before drawing conclusions (Step 4)

---

## 🔁 The EDA Workflow

```
1. Univariate Analysis   → look at one variable at a time
2. Bivariate Analysis    → look at relationships between two variables
3. Multivariate Analysis → look at relationships across 3+ variables
4. Feature Engineering   → create new, more meaningful features
5. Repeat                → new features often need their own EDA
```

---

## 1️⃣ Univariate Analysis

Looking at one column at a time — its distribution, spread, and quality.

### Numeric Columns

```python
df['Age'].describe()

df['Age'].plot(kind='hist', bins=20)   # distribution shape
df['Age'].plot(kind='kde')             # smoothed distribution
df['Age'].plot(kind='box')             # outliers & spread

df['Age'].skew()                       # skewness check
```

**Worked example — `Age` column (Titanic):**
- Checked summary stats with `.describe()`
- Plotted histogram (bins=20) and KDE to see distribution shape
- Checked skewness with `.skew()`
- Used a box plot to spot outliers
- Investigated outliers directly: `df[df['Age'] > 65]`
- Checked missing value proportion: `df['Age'].isnull().sum() / len(df['Age'])`

The same process was repeated for `Fare`:
```python
df['Fare'].describe()
df['Fare'].plot(kind='hist')
df['Fare'].plot(kind='kde')
df['Fare'].skew()
df['Fare'].plot(kind='box')
df[df['Fare'] > 250]          # investigate extreme fares
df['Fare'].isnull().sum()
```

### Categorical Columns

```python
df['Survived'].value_counts()
df['Embarked'].value_counts().plot(kind='bar')
df['Survived'].value_counts().plot(kind='pie', autopct='%0.1f%%')
df['Survived'].isnull().sum()
```

---

## 2️⃣ Bivariate / Multivariate Analysis

Looking at relationships *between* variables — this is where the real insights tend to show up.

### Categorical vs Categorical — Crosstabs & Heatmaps

```python
# Survival rate by passenger class
sns.heatmap(pd.crosstab(df['Survived'], df['Pclass'], normalize='columns') * 100)

# Survival rate by sex
pd.crosstab(df['Survived'], df['Sex'], normalize='columns') * 100

# Survival rate by embarkation point
pd.crosstab(df['Survived'], df['Embarked'], normalize='columns') * 100

# Sex distribution by embarkation point
pd.crosstab(df['Sex'], df['Embarked'], normalize='columns') * 100

# Class distribution by embarkation point
pd.crosstab(df['Pclass'], df['Embarked'], normalize='columns') * 100
```

`normalize='columns'` converts raw counts into percentages within each column, making the comparison easier to read than raw counts.

### Numeric vs Categorical — Overlaid KDE Plots

```python
df[df['Survived'] == 1]['Age'].plot(kind='kde', label='Survived')
df[df['Survived'] == 0]['Age'].plot(kind='kde', label='Not Survived')

plt.legend()
plt.show()
```

This compares the age distribution between survivors and non-survivors directly on one chart.

```python
# Average age within a specific class
df[df['Pclass'] == 1]['Age'].mean()
```

### Correlation Heatmap (Numeric Features)

```python
sns.heatmap(df.corr())
```

### Pairplot (Multivariate Overview)

```python
sns.pairplot(df1)
```

Useful for a quick visual scan of relationships across all numeric features at once.

---

## 3️⃣ Feature Engineering

Creating new features that capture information more usefully than the raw columns alone.

### Combining Train & Test Sets (when engineering features consistently across both)

```python
df1 = pd.read_csv('test.csv')
df = pd.concat([df, df1])
```

### `individual_fare` — Normalizing Fare by Family Size

The raw `Fare` column reflects the **total** ticket price for a group, not the cost per person — so it needs to be normalized:

```python
df['individual_fare'] = df['Fare'] / (df['SibSp'] + df['Parch'] + 1)

df['individual_fare'].plot(kind='box')
df[['individual_fare', 'Fare']].describe()
```

### `family_size` and `family_type` — Grouping Family Size into Categories

```python
df['family_size'] = df['SibSp'] + df['Parch'] + 1

# family_type buckets:
# 1     -> alone
# 2-4   -> small
# 5+    -> large

def transform_family_size(num):
    if num == 1:
        return 'alone'
    elif num > 1 and num < 5:
        return "small"
    else:
        return "large"

df['family_type'] = df['family_size'].apply(transform_family_size)

pd.crosstab(df['Survived'], df['family_type'], normalize='columns') * 100
```

This turns a numeric feature (`family_size`) into a more interpretable categorical feature (`family_type`), and reveals whether traveling alone/small/large family affected survival.

### `surname` and `title` — Extracting Information from the `Name` Column

```python
df['surname'] = df['Name'].str.split(',').str.get(0)

df['title'] = df['Name'].str.split(',').str.get(1).str.strip().str.split(' ').str.get(0)

# Inspect common titles
temp_df = df[df['title'].isin(['Mr.', 'Miss.', 'Mrs.', 'Master.'])]
pd.crosstab(temp_df['Survived'], temp_df['title'], normalize='columns') * 100
```

### Consolidating Rare Titles into "Other"

Rare titles (Reverend, Doctor, Colonel, Major, Captain, nobility, etc.) are grouped together since they're too sparse to analyze individually:

```python
df['title'] = df['title'].str.replace('Rev.', 'other')
df['title'] = df['title'].str.replace('Dr.', 'other')
df['title'] = df['title'].str.replace('Col.', 'other')
df['title'] = df['title'].str.replace('Major.', 'other')
df['title'] = df['title'].str.replace('Capt.', 'other')
df['title'] = df['title'].str.replace('the', 'other')
df['title'] = df['title'].str.replace('Jonkheer.', 'other')
```

### `deck` — Extracting Deck Letter from `Cabin`

`Cabin` has a high proportion of missing values, but the **first letter** (deck) is still informative:

```python
df['Cabin'].isnull().sum() / len(df['Cabin'])   # check missing proportion

df['Cabin'].fillna('M', inplace=True)            # 'M' = Missing/Unknown deck

df['deck'] = df['Cabin'].str[0]

df['deck'].value_counts()

pd.crosstab(df['deck'], df['Pclass'])

pd.crosstab(df['deck'], df['Survived'], normalize='index').plot(kind='bar', stacked=True)
```

---

## 📋 Feature Engineering Summary

| New Feature | Derived From | Purpose |
|---|---|---|
| `individual_fare` | `Fare`, `SibSp`, `Parch` | Normalize group ticket price to per-person cost |
| `family_size` | `SibSp`, `Parch` | Total family members aboard |
| `family_type` | `family_size` | Bucket into alone / small / large for easier analysis |
| `surname` | `Name` | Extract family surname (useful for grouping families) |
| `title` | `Name` | Extract social title (Mr, Mrs, Miss, Master, Other) |
| `deck` | `Cabin` | Extract deck letter, usable despite high missingness in `Cabin` |

---

## 🔍 Key Patterns Observed

- Survival rate varies notably by `Pclass` and `Sex` (seen via normalized crosstabs)
- Age distributions differ between survivors and non-survivors (overlaid KDE plot)
- `family_type` (alone/small/large) shows differing survival rates — worth digging deeper in Step 4
- `title` (extracted from name) correlates with survival, capturing social status not directly in other columns
- `deck` (extracted from `Cabin`) shows visible relationship with both `Pclass` and `Survived`, despite missing data

---

---

# 🧰 Generic EDA Template (Reusable for Any Dataset)

A standard checklist/template to follow for EDA on any new dataset, regardless of domain.

## A. First Look

```python
df.shape
df.head()
df.info()
df.describe(include='all')
```

## B. Univariate Analysis

**Numeric columns:**
```python
for col in df.select_dtypes(include='number').columns:
    print(col)
    print(df[col].describe())
    df[col].plot(kind='hist', bins=20, title=col)
    plt.show()
    df[col].plot(kind='box', title=col)
    plt.show()
    print('Skew:', df[col].skew())
    print('Missing %:', df[col].isnull().mean() * 100)
```

**Categorical columns:**
```python
for col in df.select_dtypes(include='object').columns:
    print(col)
    print(df[col].value_counts())
    df[col].value_counts().plot(kind='bar', title=col)
    plt.show()
```

## C. Bivariate / Multivariate Analysis

```python
# Numeric vs Numeric
sns.heatmap(df.corr(numeric_only=True), annot=True, cmap='coolwarm')
sns.pairplot(df)

# Numeric vs Categorical
sns.boxplot(x='categorical_col', y='numeric_col', data=df)
df.groupby('categorical_col')['numeric_col'].mean().plot(kind='bar')

# Categorical vs Categorical
pd.crosstab(df['col1'], df['col2'], normalize='columns') * 100
sns.heatmap(pd.crosstab(df['col1'], df['col2'], normalize='columns') * 100, annot=True)

# Target variable vs each feature
for col in df.columns:
    if col != target:
        pd.crosstab(df[target], df[col], normalize='columns').plot(kind='bar', stacked=True)
        plt.show()
```

## D. Outlier Detection

```python
# IQR method
Q1 = df['col'].quantile(0.25)
Q3 = df['col'].quantile(0.75)
IQR = Q3 - Q1
lower = Q1 - 1.5 * IQR
upper = Q3 + 1.5 * IQR

outliers = df[(df['col'] < lower) | (df['col'] > upper)]
```

## E. Missing Value Patterns

```python
df.isnull().sum()
df.isnull().mean() * 100

# Visualize missingness
sns.heatmap(df.isnull(), cbar=False)
```

## F. Feature Engineering Checklist

- Can any numeric columns be **binned** into categories? (e.g. age → age groups)
- Can any columns be **combined/ratio'd** into a more meaningful feature? (e.g. fare per person)
- Can any **text columns** be parsed for extra signal? (e.g. extracting titles from names)
- Are there **high-cardinality categorical** columns that need grouping into "Other"?
- Can **date columns** be split into year/month/day/weekday if relevant?
- Should any features be **scaled/encoded** before further analysis or modeling?

## G. Summary Table to Fill In

| Feature | Type | Key Observation | Action Taken |
|---|---|---|---|
| | Numeric / Categorical | | |
| | | | |

---

## ➡️ Next Step

Proceed to **Step 4: Drawing Conclusions** to interpret these patterns, validate them statistically, and connect them back to the original questions from Step 1.
