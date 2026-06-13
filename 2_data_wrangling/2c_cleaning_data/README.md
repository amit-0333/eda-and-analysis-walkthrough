# 🧼 Step 2c: Cleaning Data

Once data quality and tidiness issues have been identified in **2b: Assessing Data**, this step is about systematically fixing them using a structured, repeatable approach.

---

## 🎯 Goal of This Step

- Resolve each issue identified during assessment
- Convert messy (untidy) data into tidy data
- Resolve dirty data (quality) issues: duplicates, missing values, inaccurate/inconsistent entries
- Keep a clear record of what was changed and why
- Produce a clean, analysis-ready dataset

---

## 🔁 The Define–Code–Test Framework

A structured approach for each issue:

1. **Define** – describe the issue and the plan to fix it
2. **Code** – write the code that implements the fix
3. **Test** – verify the fix worked as intended

Always work on a **copy** of the original dataframes so the raw data remains untouched:

```python
patients_df = patients.copy()
treatments_df = treatments.copy()
treatments_cut_df = treatments_cut.copy()
adverse_reactions_df = adverse_reactions.copy()
```

---

## 🧹 Cleaning `patients_df`

### Issue 1: Misspelled name "Dsvid" (Accuracy)
```python
# Define: patient_id = 9 has 'Dsvid' instead of 'David'
# Code:
patients_df.loc[patients_df['patient_id'] == 9, 'given_name'] = 'David'

# Test:
patients_df.loc[patients_df['patient_id'] == 9, 'given_name']
```

### Issue 2: Inconsistent `state` values (Consistency)
```python
# Define: 'state' has a mix of full names and abbreviations
# Code:
state_abbreviations = {
    'California': 'CA', 'Texas': 'TX', 'New York': 'NY', 'Massachusetts': 'MA',
    'Pennsylvania': 'PA', 'Georgia': 'GA', 'Illinois': 'IL', 'Ohio': 'OH',
    'Florida': 'FL', 'Michigan': 'MI', 'Oklahoma': 'OK', 'Louisiana': 'LA',
    'New Jersey': 'NJ', 'Virginia': 'VA', 'Mississippi': 'MS', 'Wisconsin': 'WI',
    'Indiana': 'IN', 'Minnesota': 'MN', 'Tennessee': 'TN', 'Alabama': 'AL',
    'North Carolina': 'NC', 'Kentucky': 'KY', 'Washington': 'WA', 'Missouri': 'MO',
    'Idaho': 'ID', 'Kansas': 'KS', 'Nevada': 'NV', 'South Carolina': 'SC',
    'Iowa': 'IA', 'Connecticut': 'CT', 'Maine': 'ME', 'North Dakota': 'ND',
    'Nebraska': 'NE', 'Rhode Island': 'RI', 'Arkansas': 'AR', 'Colorado': 'CO',
    'Arizona': 'AZ', 'Maryland': 'MD', 'Delaware': 'DE', 'West Virginia': 'WV',
    'Oregon': 'OR', 'South Dakota': 'SD', 'Montana': 'MT', 'Vermont': 'VT',
    'District of Columbia': 'DC', 'Alaska': 'AK', 'Wyoming': 'WY',
    'New Hampshire': 'NH', 'New Mexico': 'NM', 'No data': 'Unknown'
}

patients_df['state'] = patients_df['state'].apply(lambda x: state_abbreviations.get(x, x))

# Test:
print(patients_df['state'].value_counts())
```

### Issue 3: Zip code in decimal/string format (Validity)
```python
# Define: zip_code stored as decimal float string, needs 5-digit zero-padded string
# Code:
def clean_zip_code(zip_code):
    if zip_code == 'No data':
        return 'NA'
    zip_code = int(float(zip_code))   # remove decimal part
    return str(zip_code).zfill(5)     # pad with leading zeros

patients_df['zip_code'] = patients_df['zip_code'].apply(clean_zip_code)

# Test:
patients_df['zip_code'].apply(str).str.len().value_counts()
```

### Issue 4: Missing values for 12 patients (Completeness)
```python
# Define: 12 patients missing address, city, state, zip_code, country, contact
# Code:
patients_df[['address', 'city', 'state', 'zip_code', 'country', 'phone']] = \
    patients_df[['address', 'city', 'state', 'zip_code', 'country', 'phone']].fillna('Unknown')

# Also fill remaining NaNs more broadly if needed:
patients_df.fillna('No data', inplace=True)

# Test:
patients_df[patients_df.isnull().any(axis=1)]
patients_df.info()
```

### Issue 5: Incorrect data types (Validity)
```python
# Define: sex, zip_code, birthdate have incorrect data types
# Code:
patients_df['birthdate'] = pd.to_datetime(patients_df['birthdate'], errors='coerce')
patients_df['sex'] = patients_df['sex'].astype('category')
patients_df['zip_code'] = patients_df['zip_code'].astype('str')

# Test:
patients_df.info()
```

### Issue 6: Duplicate "John Doe" entries (Accuracy)
```python
# Define: duplicate patient entries identified by name
# Code:
patients_df_tmp = patients_df.drop_duplicates(subset=['given_name', 'surname'])

# Test:
patients_df_tmp[patients_df_tmp.duplicated(subset=['given_name', 'surname'])]
```

### Issue 7 & 8: Implausible weight (48) and height (27) values (Accuracy)
```python
# Define: height = 27 inches and weight = 48 are clearly erroneous values
# Code:
# Reverse-engineer the correct height using known weight (lb) and BMI
weight_lb = 192.3
bmi = 26.1

weight_kg = weight_lb * 0.453592
height_m = (weight_kg / bmi) ** 0.5
height_inches = height_m * 39.3701   # ≈ 71.97

patients_df.loc[patients_df['height'] == 27, 'height'] = 71.97

# Test:
patients_df.loc[(patients_df['height'] == 27)]
```

### Splitting `contact` into Phone & Email (Tidiness)
```python
# Define: 'contact' column combines phone number and email address into one string
# Code:
import re

def find_contact_details(text: str) -> tuple:
    if pd.isna(text):
        return np.nan

    phone_number_pattern = re.compile(
        r"(\+[\d]{1,3}\s)?(\(?[\d]{3}\)?\s?-?[\d]{3}\s?-?[\d]{4})"
    )
    phone_number = re.findall(phone_number_pattern, text)

    if len(phone_number) <= 0:
        phone_number = np.nan
    elif len(phone_number) >= 2:
        phone_number = phone_number[1]
    else:
        phone_number = phone_number[0]

    possible_email_add = re.sub(phone_number_pattern, "", text).strip()

    return phone_number, possible_email_add

patients_df['phone'] = patients_df["contact"].apply(lambda x: find_contact_details(x)).apply(lambda x: x[0])
patients_df['email'] = patients_df["contact"].apply(lambda x: find_contact_details(x)).apply(lambda x: x[1])

patients_df.drop(columns='contact', inplace=True)

# Test:
patients_df
```

---

## 🧹 Cleaning `treatments_df` & `treatments_cut_df`

### Issue 9: `given_name`/`surname` all lowercase (Consistency)
```python
# Define: names are in all lowercase, should be title case
# Code:
treatments_df['given_name'] = treatments_df['given_name'].str.title()
treatments_df['surname'] = treatments_df['surname'].str.title()

treatments_cut_df['given_name'] = treatments_cut_df['given_name'].str.title()
treatments_cut_df['surname'] = treatments_cut_df['surname'].str.title()
```

### Issue 10 & 11 & 15: Dosage range with units, `-` placeholder, medication as column headers (Messy/Tidiness)
```python
# Define: 'auralin' and 'novodra' columns contain dosage ranges with 'u' units
# and '-' placeholders. Medication type is stored as column headers instead of values.

# Code (for treatments_df via melt):
treatments_df = pd.concat([treatments_df, treatments_cut_df])

treatments_df = treatments_df.melt(
    id_vars=['given_name', 'surname', 'hba1c_start', 'hba1c_end', 'hba1c_change'],
    var_name='type',
    value_name='dosage_range'
)

# Remove rows where dosage was '-'
treatments_df = treatments_df[treatments_df['dosage_range'] != '-']

# Split dosage range into start/end
treatments_df['dosage_start'] = treatments_df['dosage_range'].str.split('-').str.get(0)
treatments_df['dosage_end'] = treatments_df['dosage_range'].str.split('-').str.get(1)

treatments_df.drop(columns='dosage_range', inplace=True)

# Remove 'u' units and convert to int
treatments_df['dosage_start'] = treatments_df['dosage_start'].str.replace('u', '').astype('int')
treatments_df['dosage_end'] = treatments_df['dosage_end'].str.replace('u', '').astype('int')

# Test:
treatments_df
```

```python
# Alternative approach for treatments_cut_df (row-wise function):
def clean_dosage(dosage):
    return dosage.replace('u', '').strip()

def parse_dosage(dosage):
    if pd.isna(dosage) or dosage == '-':
        return np.nan, np.nan
    try:
        start, end = dosage.split(' - ')
        return float(start), float(end)
    except ValueError:
        return np.nan, np.nan

def determine_medication(row):
    if row['novodra'] == '-':
        row['med_type'] = 'auralin'
        row['start_dosage'], row['end_dosage'] = parse_dosage(row['auralin'])
    else:
        row['med_type'] = 'novodra'
        row['start_dosage'], row['end_dosage'] = parse_dosage(row['novodra'])
    return row

treatments_cut_df['med_type'] = np.nan
treatments_cut_df['start_dosage'] = np.nan
treatments_cut_df['end_dosage'] = np.nan

treatments_cut_df['auralin'] = treatments_cut_df['auralin'].apply(clean_dosage)
treatments_cut_df['novodra'] = treatments_cut_df['novodra'].apply(clean_dosage)

treatments_cut_df = treatments_cut_df.apply(determine_medication, axis=1)

# Test:
treatments_cut_df.info()
```

### Issue 12: Missing `hba1c_change` values (Completeness)
```python
# Define: hba1c_change has missing values, can be derived from start - end
# Code:
treatments_cut_df['hba1c_change'] = treatments_cut_df['hba1c_change'].fillna(
    treatments_cut_df['hba1c_start'] - treatments_cut_df['hba1c_end']
)

# Test:
treatments_cut_df['hba1c_change'].isna().sum()
```

### Issue 13: Duplicate "Joseph Day" entry (Accuracy)
```python
# Define: duplicate entry for Joseph Day exists in treatments_df
# Code:
treatments_df = treatments_df.drop_duplicates(subset=['given_name', 'surname'], keep='first')

# Test:
treatments_df[treatments_df['given_name'] == 'Joseph']
```

### Issue 14: Incorrect `hba1c_change` value (9 instead of 4)
```python
# Define: a hba1c_change value of 9 is inconsistent and was already corrected
# Test:
treatments_df[treatments_df.hba1c_change == 9]   # should return empty after fix
```

---

## 🧹 Cleaning `adverse_reactions_df`

### Naming Consistency
```python
# Define: given_name and surname need consistent title case
# Code:
adverse_reactions_df['given_name'] = adverse_reactions_df['given_name'].str.title()
adverse_reactions_df['surname'] = adverse_reactions_df['surname'].str.title()

# Test:
adverse_reactions_df.sample(10)
```

---

## 🔗 Combining the Datasets

### Merge treatments with adverse reactions
```python
treatments_df = treatments_df.merge(
    adverse_reactions_df, how='left', on=['given_name', 'surname']
)

treatments_df
```

### Align `treatments_cut_df` columns with `treatments_df`
```python
# Rename columns to match
treatments_cut_df.rename(columns={
    'med_type': 'type',
    'start_dosage': 'dosage_start',
    'end_dosage': 'dosage_end'
}, inplace=True)

columns_to_keep = [
    'given_name', 'surname', 'hba1c_start', 'hba1c_end', 'hba1c_change',
    'type', 'dosage_start', 'dosage_end'
]

# Check overlap between the two treatment datasets
common_rows_df = pd.merge(
    treatments_df[columns_to_keep],
    treatments_cut_df[columns_to_keep],
    on=['given_name', 'surname'],
    how='inner'
)

print(common_rows_df.info())
```

**Note:** `adverse_reactions_df` (34 rows) are already represented in `treatments_df` after the merge — no separate handling needed.

---

## ✅ Final Checks

```python
patients_df.info()
treatments_df.info()    # 349 rows
treatments_cut_df.info()  # 70 rows
```

---

## 💾 Saving Cleaned Data

```python
patients_df.to_csv('data/processed/patients_clean.csv', index=False)
treatments_df.to_csv('data/processed/treatments_clean.csv', index=False)
```

---

## ➡️ Next Step

Proceed to **Step 3: EDA (Exploratory Data Analysis)** to explore patterns, relationships, and trends in the cleaned dataset.
