# 🛒 Quantium Retail Strategy — Chips Category Analysis

> End-to-end Python data analysis project identifying customer segments driving chip sales for a major Australian retailer — with actionable recommendations for the category manager.

[![Python](https://img.shields.io/badge/Python-3.12-blue?style=flat&logo=python)](https://python.org)
[![Pandas](https://img.shields.io/badge/Library-Pandas-lightblue?style=flat)]()
[![Seaborn](https://img.shields.io/badge/Library-Seaborn-teal?style=flat)]()
[![Forage](https://img.shields.io/badge/Project-Quantium%20Virtual%20Experience-orange?style=flat)]()

---

## 📁 Quick Access

| Resource | Link |
|----------|------|
| 📓 Full Notebook | [View Notebook](./Quantium_Retail_Analysis.ipynb) |

---

## 🏢 Project Background

This project is based on the **Quantium Data Analytics Virtual Experience** on Forage. Quantium is a leading data analytics firm working with retail clients across Australia.

The task simulates the role of a **Data Analyst** at Quantium — analysing transaction and customer data for a major supermarket chain's chips category. The goal is to understand which customer segments buy the most chips, at what price, and surface strategic recommendations for the category manager.

---

## 🎯 Problem Statement

> *"The chips category manager needs to understand customer purchasing behaviour to inform the next 6 months of category strategy — specifically which segments to target, which brands to promote, and how to plan for seasonal demand."*

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **Python** | Core analysis language |
| **Pandas** | Data loading, cleaning, merging, groupby analysis |
| **NumPy** | Numerical operations |
| **Matplotlib** | Custom chart building and dashboard layout |
| **Seaborn** | Statistical visualisations |
| **Regex** | Brand name and packet size extraction from product strings |
| **Collections.Counter** | Word frequency analysis for product names |

---

## 📋 Analysis Walkthrough — Step by Step

---

### 1. 📦 Data Loading

Two datasets loaded and inspected:
- `QVI_purchase_behaviour.csv` — 72,637 loyalty card holders with LIFESTAGE and PREMIUM_CUSTOMER classification
- `QVI_transaction_data.xlsx` — 264,836 transactions from Jul 2018 to Mar 2019

---

### 2. 🔍 Exploratory Data Analysis

- Checked shape, dtypes, null values for both datasets
- Identified 7 unique lifestage segments and 3 premium customer tiers
- Found DATE column stored as numeric Excel serial — flagged for conversion
- Identified 364 unique dates vs 274 expected — flagged missing date investigation

---

### 3. 🧹 Data Preprocessing

#### Date Fix
```python
transaction_df['DATE'] = pd.to_datetime(
    transaction_df['DATE'], origin='1899-12-30', unit='D'
)
```
Converted numeric Excel serial to proper datetime format.

#### Missing Date Investigation
Built a complete date range and merged to find gaps:
```python
all_dates = pd.date_range(start='2018-07-01', end='2019-03-31')
transactions_by_date = pd.merge(all_dates_df, transactions_by_date,
                                on='DATE', how='left')
```
**Finding:** Only December 25, 2018 had zero transactions — Christmas Day store closure. No data quality issue.

#### Outlier Detection & Removal
Boxplot revealed one customer (card 226000) purchasing 200 units in a single transaction — identified as a non-retail commercial buyer and removed.
```python
transaction_df = transaction_df[transaction_df['LYLTY_CARD_NBR'] != 226000]
```

---

### 4. ⚙️ Feature Engineering

#### Packet Size Extraction (Regex)
```python
transaction_df['packet_size'] = transaction_df['PROD_NAME'].str.extract(r'(\d+)[gG]')
```
Extracted gram weight from product name strings across 114 unique products.

#### Product Name Cleaning
```python
transaction_df['PROD_NAME_CLEAN'] = (
    transaction_df['PROD_NAME']
    .str.replace(r'\d+', '', regex=True)
    .str.replace(r'[^a-zA-Z\s]', '', regex=True)
    .str.strip()
)
```

#### Brand Name Extraction
Extracted first word of cleaned product name as brand:
```python
transaction_df['brand_name'] = transaction_df['PROD_NAME_CLEAN'].str.split().str[0]
```

#### Brand Standardisation
Identified and merged 8 duplicate/abbreviated brand names:
```python
brand_mapping = {
    'Burger' : 'Smiths',
    'Dorito' : 'Doritos',
    'Infzns' : 'Infuzions',
    'NCC'    : 'Natural',
    'Snbts'  : 'Sunbites',
    'WW'     : 'Woolworths',
    'Smith'  : 'Smiths',
    'RRD'    : 'Red Rock Deli',
}
transaction_df['brand_name'] = transaction_df['brand_name'].replace(brand_mapping)
```

#### Salsa Removal
Filtered non-chip products from the dataset:
```python
transaction_df = transaction_df[
    ~transaction_df['PROD_NAME'].str.contains('salsa', case=False)
]
```

---

### 5. 📊 Customer Segment Analysis

After merging transaction data with purchase behaviour data on loyalty card number:

| Analysis | Key Finding |
|----------|------------|
| **Total Sales by Segment** | Older Families (Budget) and Young Singles/Couples (Mainstream) are the top spending segments |
| **Customer Count** | Mainstream Young Singles/Couples is the largest segment by customer count |
| **Avg Spend per Customer** | Older Families — Budget spend highest per customer |
| **Chips per Customer** | Young Families and Older Families buy the most units per customer |
| **Avg Price per Unit** | Mainstream customers pay $3.87/unit vs $3.80 Budget — small but consistent premium gap |
| **Preferred Packet Sizes** | 175g–200g most popular across all segments |

---

### 6. 📋 Executive Dashboard

A full matplotlib dashboard built at the end of the notebook containing:
- 5 KPI cards (Total Revenue, Transactions, Customers, Brands, Avg Price)
- Total Sales by Segment (horizontal bar)
- Brand Market Share (donut chart)
- Avg Spend per Customer by Lifestage (grouped bar — all segments)
- Avg Price per Unit by Customer Type
- Daily Transaction Volume trend

---

## 💡 Key Recommendations

| # | Recommendation | Based On |
|---|---------------|----------|
| 1 | **Target Mainstream Young Singles/Couples & Retirees** — highest total revenue segments | Segment sales analysis |
| 2 | **Introduce premium bundles** — mainstream customers already pay more per unit | Price per unit analysis |
| 3 | **Never stock out 175g–200g packs** — most popular size across all segments | Packet size analysis |
| 4 | **Promote Infuzions & Red Rock Deli** — Kettle/Smiths/Doritos already dominant | Brand market share |
| 5 | **Target Young & Older Families with value multipacks** — high volume, price sensitive | Units per customer analysis |
| 6 | **Stock up by Dec 18** — pre-Christmas spike is the biggest annual demand event | Transaction volume trend |

---

## 📂 Repository Structure

```
📦 Quantium-Retail-Analysis
 ┣ 📄 README.md
 ┣ 📓 Quantium_Retail_Analysis.ipynb
 ┗ 📁 screenshots
    ┣ 🖼️ dashboard.png
    ┣ 🖼️ segment_analysis.png
    ┗ 🖼️ brand_analysis.png
```

---

## 👩‍💻 About Me

Aspiring Data Analyst passionate about turning raw data into business decisions. Skilled in Python, Power BI, SQL, and data storytelling.

📧 *nupursrivastava94@gmail.com*
🔗 *www.linkedin.com/in/nupur-srivastava-5456131a3*
🐙 https://github.com/Nupur-DS

---

*Based on the Quantium Data Analytics Virtual Experience Program on Forage. Dataset used for educational purposes only.*s
