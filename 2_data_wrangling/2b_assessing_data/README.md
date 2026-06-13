# 🔍 Step 2b: Assessing Data

> *"You have to become Sherlock."* — before fixing data, understand what's wrong with it.

In this step, the goal is **not to fix anything yet**. The goal is to deeply understand the dataset — its structure, content, and quality — so that cleaning decisions made later are informed and justified.

---

## 🎯 Goal of This Step

- Understand what the data is about
- Identify *what* is wrong with the data (not yet *how* to fix it)
- Distinguish between **dirty data** (quality issues) and **messy data** (structural/tidiness issues)
- Build a documented list of issues to address in the cleaning step

---

## 🧹 Two Kinds of "Unclean" Data

### 1. Dirty Data (Quality Issues)
Dirty data, also known as **low quality data**, has problems with its *content*.

- **Duplicated data** – same record appears more than once
- **Missing data** – nulls, blanks, placeholders (e.g. `?`, `-`, `No data`)
- **Corrupt data** – garbled values, encoding errors, broken entries
- **Inaccurate data** – wrong values (e.g. impossible height/weight, typos in names)

### 2. Messy Data (Tidiness Issues)
Messy data, also known as **untidy data**, has problems with its *structure*.

A tidy dataset should have:

- Each **variable** forms a column
- Each **observation** forms a row
- Each **observational unit** forms a table

---

## 🔎 Methods of Assessment

### Visual Assessment
Looking directly at the data to get a feel for it.

```python
patients.head()
treatments.head()
adverse_reactions.head()
treatments_cut.shape
```

You can also export to Excel for manual review across multiple sheets:

```python
with pd.ExcelWriter('clinical_trials.xlsx') as writer:
    patients.to_excel(writer, sheet_name='patients')
    treatments.to_excel(writer, sheet_name='treatments')
    treatments_cut.to_excel(writer, sheet_name='treatment_cut')
    adverse_reactions.to_excel(writer, sheet_name='adverse_reactions')
```

### Programmatic Assessment
Using code to systematically inspect the dataset.

```python
df.info()
df.describe()
df.dtypes
df.isnull().sum()
df.duplicated().sum()
df['column'].value_counts()
```

---

## 📋 Worked Example: Clinical Trials Dataset

Below are the actual issues identified across the `patients`, `treatments`, `treatments_cut`, and `adverse_reactions` datasets, with the assessment code used to find them.

### Missing Data
```python
patients[patients['address'].isnull()]
patients[patients.isnull().any(axis=1)]
```
→ 12 patients are missing `address`, `city`, `state`, `zip_code`, `country`, and `contact`.

### Duplicates
```python
treatments[treatments.duplicated()]
treatments[treatments.duplicated(subset=['given_name', 'surname'])]
treatments_cut[treatments_cut.duplicated(subset=['given_name', 'surname'])]
adverse_reactions.duplicated().sum()
patients[patients.duplicated(subset=['given_name', 'surname'])]
```
→ Found duplicate "John Doe" style entries in `patients`, and a duplicate `Joseph Day` entry in `treatments`.

### Out-of-Range / Inaccurate Values
```python
patients.describe()
patients[patients['height'] == 27]          # height of 27 inches – clearly wrong
patients.loc[(patients['weight'] == 48)]     # check unusually low weight
treatments.sort_values('hba1c_change', na_position='first')
```
→ One patient has `height = 27` (likely a data entry error). One `hba1c_change` value of `9` looks inconsistent with expected range.

### Inconsistent Categories
```python
patients.state.value_counts()
```
→ `state` column has a mix of full state names ("California") and abbreviations ("CA") — inconsistent representation of the same information.

### Incorrect / Suspicious Format
```python
patients['zip_code'].apply(str).str.len().value_counts()
patients.zip_code.sample(5)
```
→ `zip_code` is stored as a decimal/float string (e.g. `90210.0`) instead of a clean 5-digit string.

### Spelling / Naming Issues
```python
patients_df.loc[patients_df['patient_id'] == 9, 'given_name']
```
→ Patient `id = 9` has the name misspelled as `Dsvid` instead of `David`.

### Data Type Issues
```python
patients_df.info()
treatments_df.info()
treatments_cut_df.info()
```
→ `sex`, `zip_code`, and `birthdate` have incorrect data types and need conversion.

### Messy / Untidy Structure
```python
treatments.head()
treatments_cut.describe()
```
→ `auralin` and `novodra` columns store dosage *ranges* (e.g. `41u - 48u`) as strings — these are really **two variables** (`dosage_start`, `dosage_end`) combined into one column, with units (`u`) mixed in as text. Medication type is encoded as **column headers** instead of values — a classic tidiness issue (should be melted into a `type` column).

### Combined Fields (Contact Column)
```python
patients.head()
```
→ The `contact` column combines **phone number** and **email address** into a single string — these are two separate variables.

---

## 📝 Summary of Issues to Fix

| # | Issue | Type | Column(s) Affected | Notes |
|---|-------|------|---------------------|-------|
| 1 | Misspelled name "Dsvid" | Dirty – Accuracy | `given_name` | patient_id = 9 |
| 2 | Inconsistent state values | Dirty – Consistency | `state` | Mix of full names and abbreviations |
| 3 | Zip code in decimal/string format | Dirty – Validity | `zip_code` | Needs zero-padded 5-digit string |
| 4 | Missing values for 12 patients | Dirty – Completeness | `address`, `city`, `state`, `zip_code`, `country`, `contact` | |
| 5 | Incorrect data types | Dirty – Validity | `sex`, `zip_code`, `birthdate` | |
| 6 | Duplicate "John Doe" entries | Dirty – Accuracy | `patients` (whole row) | |
| 7 | Implausible weight value | Dirty – Accuracy | `weight` | weight = 48 |
| 8 | Implausible height value | Dirty – Accuracy | `height` | height = 27 inches |
| 9 | `given_name`/`surname` all lowercase | Dirty – Consistency | `treatments`, `treatments_cut` | |
| 10 | Dosage stored as range with units | Messy – Tidiness | `auralin`, `novodra` | Should split into `dosage_start`/`dosage_end` |
| 11 | `-` placeholder treated as data | Dirty – Validity | `auralin`, `novodra` | Should be NaN |
| 12 | Missing `hba1c_change` values | Dirty – Completeness | `hba1c_change` (in `treatments_cut`) | |
| 13 | Duplicate "Joseph Day" entry | Dirty – Accuracy | `treatments` | |
| 14 | Incorrect `hba1c_change` value (9 instead of 4) | Dirty – Accuracy | `hba1c_change` | |
| 15 | Medication type stored as column headers | Messy – Tidiness | `auralin`, `novodra` | Needs melting into `type` column |
| 16 | `contact` combines phone + email | Messy – Tidiness | `contact` | Should be split into `phone`, `email` |

---

## ➡️ Next Step

Proceed to **2c: Cleaning Data**, where each of the issues above is addressed using the define–code–test framework.
