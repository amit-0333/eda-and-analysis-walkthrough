<div align="center">

```
███████╗██████╗  █████╗ 
██╔════╝██╔══██╗██╔══██╗
█████╗  ██║  ██║███████║
██╔══╝  ██║  ██║██╔══██║
███████╗██████╔╝██║  ██║
╚══════╝╚═════╝ ╚═╝  ╚═╝
```

### 📊 EDA & Data Analysis Walkthrough

> A structured, step-by-step walkthrough of the data analysis process — from asking the right questions to communicating final results — built using Python and Pandas.

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=for-the-badge&logo=python&logoColor=white)
![Selenium](https://img.shields.io/badge/Selenium-43B02A?style=for-the-badge&logo=selenium&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)

</div>

---

## 📌 About

This repository documents my **Data Analysis learning journey**, following the standard 5-step data analysis process:

1. **Ask** – Defining the right questions to ask of the dataset
2. **Data Wrangling** – Gathering, assessing, and cleaning data
3. **EDA** – Exploring patterns, trends, and relationships, and engineering new features
4. **Draw Conclusions** – Interpreting findings and insights
5. **Communicate Results** – Presenting results clearly

Each step is documented in its own folder with notebooks and notes, showing the full workflow from raw data to final insights. Alongside the step-by-step walkthrough, the repo also includes a **hands-on project** applying the full process end-to-end on real scraped data.

---

## 🗺️ Repository Structure

```
eda-and-analysis-walkthrough/
│
├── 📁 1_asking_questions/
│   └── 📄 questions.md                    # Generic question framework for any dataset
│
├── 📁 2_data_wrangling/
│   ├── 📁 2a_gathering_data/
│   │   └── 📄 README.md                   # Importing/exporting data: CSV, JSON, SQL, API, scraping, Excel, HTML
│   ├── 📁 2b_assessing_data/
│   │   └── 📄 README.md                   # Dirty vs messy data, assessment methods + worked example
│   └── 📁 2c_cleaning_data/
│       └── 📄 README.md                   # Define-Code-Test framework + worked example
│
├── 📁 3_eda/
│   └── 📄 README.md                       # Univariate/bivariate/multivariate analysis, feature engineering (Titanic) + generic EDA template
│
├── 📁 4_drawing_conclusions/
│   └── 📄 README.md                       # Interpreting findings, statistical validation, limitations
│
├── 📁 5_communicating_results/
│   └── 📄 README.md                       # Structuring the narrative, choosing visuals, report/dashboard formats
│
├── 📁 web-scraping-and-cleaning/
│   └── 📄 README.md                       # Mini project: Smartprix mobile data — Selenium + BeautifulSoup scraping, cleaning, EDA & feature engineering
│
└── 📄 README.md
```

---

## 🧪 Featured Mini Project

**📱 Smartprix Mobile Phone Data Analysis**
End-to-end project covering the full pipeline on real-world data:
- Scraped live smartphone listings using **Selenium + BeautifulSoup**
- Cleaned messy scraped fields (prices, specs, boolean flags) into a structured dataset
- Performed univariate, bivariate, and multivariate EDA on price, ratings, and specs
- Engineered features and used **KNN imputation** to handle missing numeric specs
- Used one-hot encoding to study correlation between categorical features and price

📂 See [`web-scraping-and-cleaning/`](./web-scraping-and-cleaning/) for the full write-up.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| 🐍 **Python** | Core language |
| 🐼 **Pandas** | Data wrangling and analysis |
| 📓 **Jupyter Notebook** | Step-by-step workflow |
| 📊 **Matplotlib / Seaborn** | Data visualisation |
| 🧭 **Selenium** | Scraping JavaScript-rendered pages |
| 🍲 **BeautifulSoup** | Parsing scraped HTML |
| 🤖 **Scikit-learn** | Missing value imputation (KNNImputer) |

---

## 👨‍💻 Author

**Amit Kumar**

[![GitHub](https://img.shields.io/badge/GitHub-amit--0333-181717?style=flat&logo=github)](https://github.com/amit-0333)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Amit%20Kumar-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/amit-kumar-a62a3640a/)

---

<div align="center">

> 📝 *Built as part of my Data Science and Python learning journey.*

⭐ **Star this repo if you found it useful!**

</div>
