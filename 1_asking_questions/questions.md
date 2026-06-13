# 📝 Step 1: Asking Questions

Before jumping into data wrangling and EDA, an experienced data analyst/scientist spends time framing the right questions. These questions shape what data is needed, how it should be cleaned, which features matter, and what kind of analysis/model is appropriate.

This is a **generic template** — usable for any dataset.

---

## 1️⃣ Understanding the Problem & Goal

- What is the business/research problem I'm trying to solve?
- What is the main objective of this analysis — description, prediction, or inference?
- Who is the end user of this analysis, and what decision will it inform?
- What does "success" look like for this analysis?
- Is this a one-time analysis or something that needs to be repeatable/automated?

---

## 2️⃣ Understanding the Data

- What does each row represent? (a person, transaction, event, etc.)
- What is the time period / scope / geography covered by this data?
- How was this data collected? (survey, sensor, transactional system, scraped, API, etc.)
- Is the dataset a sample or the full population?
- How large is the dataset (rows, columns, size)? Will it fit in memory?
- Is there a data dictionary / documentation available for the columns?

---

## 3️⃣ Feature-Level Questions

### Relevance
- What features contribute meaningfully to this analysis?
- Which features are not important / can be dropped (IDs, duplicates, irrelevant metadata)?
- Are there redundant features that capture the same information?
- Are there features with little to no variance (constant columns)?

### Data Types & Structure
- What is the data type of each feature (numerical, categorical, datetime, text, boolean)?
- Are categorical variables nominal or ordinal?
- Are any numeric columns actually categorical (e.g., zip codes, IDs)?
- Are date/time columns in the correct format?

### Quality
- Which columns have missing values, and how much (%)?
- Is missingness random or does it follow a pattern (MCAR, MAR, MNAR)?
- Are there duplicate rows or duplicate records (e.g., same entity counted twice)?
- Are there inconsistent labels/categories (e.g., "NY" vs "New York")?
- Are there outliers or impossible values (e.g., negative age, age = 200)?

---

## 4️⃣ Relationships Between Features

- Which features have strong correlation with each other (multicollinearity)?
- Which features have strong correlation/association with the target variable?
- Are there any non-linear relationships that correlation alone won't capture?
- Are there interaction effects between features (e.g., effect of A depends on B)?
- Do categorical features show meaningful differences in the target across groups?

---

## 5️⃣ Preprocessing Questions

- Do I need preprocessing before analysis/modeling? Why or why not?
- Do numeric features need scaling/normalization (e.g., for distance-based models)?
- Do categorical features need encoding (one-hot, label, ordinal)?
- How should missing values be handled — drop, impute (mean/median/mode), or flag?
- How should outliers be handled — remove, cap, transform, or keep?
- Are there skewed distributions that need transformation (log, sqrt, Box-Cox)?
- Does the data need to be balanced (e.g., class imbalance in target variable)?

---

## 6️⃣ Feature Engineering Questions

- What kind of feature manipulation/engineering is required?
- Can new features be derived from existing ones (e.g., extracting year/month from date, ratios, totals)?
- Should continuous variables be binned into categories (e.g., age groups)?
- Are there text fields that need parsing, tokenizing, or extracting info from?
- Can multiple columns be combined into a single more meaningful feature?
- Are there domain-specific features that could add value (e.g., BMI from height/weight)?

---

## 7️⃣ Trend, Pattern & Segmentation Questions

- Are there trends over time (seasonality, growth, decline)?
- Are there differences across groups/segments (region, category, demographic)?
- Are there natural clusters or segments in the data?
- Do certain subgroups behave very differently from the overall population?

---

## 8️⃣ Hypotheses to Test

- What are my initial hypotheses about the data, based on domain knowledge?
- What do I expect to see, and what would surprise me?
- Are there any A/B comparisons or before/after comparisons to make?

---

## 9️⃣ Output & Communication Questions

- What format should the final output take (report, dashboard, presentation, model)?
- What level of technical detail does the audience need?
- What are the 2–3 key takeaways I want the audience to remember?
- What visualizations best communicate the findings?

---

## ✅ Checklist Before Moving to Step 2 (Data Wrangling)

- [ ] Defined the core question(s) this analysis answers
- [ ] Identified target variable (if predictive analysis)
- [ ] Identified relevant vs irrelevant features
- [ ] Listed known data quality issues to investigate
- [ ] Noted initial hypotheses to validate during EDA

---

## 📌 Notes
- Document assumptions, constraints, or context specific to this dataset here.
