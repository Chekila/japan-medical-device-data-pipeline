# Japan Medical Device Certification Data Cleaning and EDA

## 1. Project Introduction

This project cleans and analyzes Japanese medical device certification data obtained from the PMDA website.

The goal of this project is to convert the raw Excel dataset into a clean, standardized, and analysis-ready format. The workflow includes column name translation, missing value review, date formatting, duplicate detection, SQLite database export, and exploratory data analysis (EDA).

The cleaned dataset supports downstream analysis, including regulatory data review, certification year trend analysis, certification body analysis, major vendor analysis.

---

## 2. Data Source

The raw data was downloaded from the Pharmaceuticals and Medical Devices Agency (PMDA) website in Japan.

| Item | Description |
|---|---|
| Source organization | Pharmaceuticals and Medical Devices Agency (PMDA), Japan |
| Source URL | `https://www.pmda.go.jp/review-services/drug-reviews/about-reviews/devices/0026.html?` |
| Source file | `000280216.xlsx` |
| Data format | Excel file |
| Date accessed | `2026-05-12` |

The raw dataset contains Japanese medical device certification records, including certification body codes, certification numbers, certification dates, sales names, general product categories, certified vendors, corporate identifiers, succession-related information, certification summary dates, and certification revocation dates.

---

## 3. Data Extraction

The raw Excel file was downloaded from the PMDA website and loaded into Python using `pandas.read_excel()`.

Example:

```python
import pandas as pd

data1 = pd.read_excel("../data/000280216.xlsx")
```

---

## 4. Cleaning Workflow

### 4.1 Load and inspect raw data

The raw Excel file was loaded using `pandas.read_excel()`. Initial checks included dataset shape, column names, data types, missing values, and sample records to understand the raw structure before cleaning.

---

### 4.2 Standardize column names and add the data source

Japanese column names were translated into standardized English snake_case format to improve readability and support downstream analysis. Source metadata (`source_file`, `source_url`) was added to preserve traceability to the original dataset.

---

### 4.3 Review missing values and preserve business-meaningful nulls

Missing values were reviewed across all columns. Null values were preserved when representing meaningful business states rather than data quality issues (e.g., non-revoked certifications or absent succession events).

---

### 4.4 Standardize date columns

Date-related columns were converted using `pd.to_datetime()` and standardized into a consistent format for downstream analysis and SQLite export.

Target columns include:
- `certification_date`
- `date_of_succession`
- `certification_summary_date`
- `certification_revocation_date`

---
### 4.5 Identify and deduplicate records

The deduplication workflow was:

1. Define duplicate key columns  
2. Flag duplicate groups using `duplicate_flag`  
3. Review flagged duplicate records  
4. Remove confirmed duplicates while keeping the first occurrence  
5. Recheck the dataset for remaining duplicates  

First, duplicate detection was based on a composite business key:

```python
duplicate_keys = [
    "certification_number",
    "sales_name",
    "vendor_name_certified_person"
]
```

These columns were selected because a true duplicate record should refer to the same certification number, the same product sales name, and the same certified vendor.

Then, duplicate groups were flagged using:

```python
data1["duplicate_flag"] = data1.duplicated(
    subset=duplicate_keys,
    keep=False
)
```

`keep=False` was used to mark all rows within each duplicate group, allowing the duplicate records to be reviewed before removal.

Initial duplicate detection identified:

```text
1294 rows flagged as belonging to duplicate groups
```

After reviewing the flagged duplicate groups, confirmed duplicate records were removed while keeping the first occurrence:

```python
data1 = data1.drop_duplicates(
    subset=duplicate_keys,
    keep="first"
)
```

Dataset size before and after deduplication:

| Stage | Row count |
|------|-----------:|
| Before deduplication | 26,630 |
| After deduplication | 25,849 |
| Removed rows | 781 |

This indicates that multiple duplicate groups existed, and removing duplicates while preserving the first occurrence reduced the dataset by 781 records.

Finally, the dataset was checked again to confirm that no duplicate records remained under the same duplicate key.

Post-deduplication validation result:

```text
Remaining duplicate records: 0
```

This confirms that the deduplication process successfully removed confirmed duplicate records while preserving one representative record for each duplicate group.

After deduplication and standardization, the cleaned dataset was exported as:

```text
cleaned_japan.xlsx
```

This file serves as the analysis-ready dataset and was used as the input for subsequent exploratory data analysis (EDA) and visualization workflows.

---

### 4.6 Export cleaned dataset to SQLite

The cleaned dataset was exported into SQLite format for structured storage, SQL querying, and future integration with dashboards or downstream analytical workflows.

Output:

```text
Japan_medical_devices.sqlite
```
---
## 5. Database Schema

The following schema describes the cleaned Japan medical device certification dataset.

| Column Name | Description | Example Values | Data Type |
|---|---|---|---|
| certification_body_code | Certification body code assigned to the record. | AB, AI, AG | object |
| certification_number | Certification number of the medical device. | 217ABBZX00001000 | object |
| certification_date | Date when certification was issued. | 2005-08-15 | datetime64[ns] |
| sales_name | Sales or marketed name of the medical device. | ビデオプロセッサＥＰＫ－１０００ | object |
| general_name | General category/name of the medical device. | 胚移植用カテーテル | object |
| vendor_name_certified_person | Name of the certified vendor or holder. | ペンタックス株式会社 | object |
| corporate_number | Corporate identification number of the vendor. | 5080101000000 | float64 |
| company_name_designated_foreign_<wbr>manufactured_medical_device_<wbr>manufacturer_and_distributor | Name of the designated foreign manufacturer/distributor, if applicable. | ＡＪＭＤ株式会社 | object |
| corporate_number_2 | Corporate number associated with the foreign manufacturer/distributor. | 5010001141738 | float64 |
| transition_from_authorization_to_certification | Indicates transition from authorization to certification. | ○ | object |
| succession_item | Indicates whether succession occurred. | 2025/3/31  0:00:00 | object |
| date_of_succession | Date of succession, if applicable. | 2018-04-01 | datetime64[ns] |
| change_of_certification_body_at_time_of_succession | Indicates whether certification body changed during succession. | ○ | object |
| certification_summary_date | Date when certification information was summarized. | 2011-04-07 | datetime64[ns] |
| certification_revocation_date | Date when certification was revoked | 2024/4/24  0:00:00 | datetime64[ns] |
| source_file | Original source file name added during cleaning. | 000280216.xlsx | object |
| source_url | Original source URL added during cleaning. | https://www.pmda.go.jp/review-services/drug-reviews/about-reviews/devices/0026.html? | object |
| duplicate_flag | Indicates whether the record belongs to a duplicate group. | True / False | bool |


---
## 6. Exploratory Data Analysis (EDA)

EDA was performed using the cleaned dataset:

```text
cleaned_japan.xlsx
```

The purpose of the analysis was to understand the overall data structure, certification trends, certification body distribution, and major vendor composition.

---

### 6.1 Dataset Overview

The cleaned dataset was first inspected to understand its overall structure and quality.

Initial checks included:

- dataset shape
- column names
- data types
- non-null counts
- missing value counts
- missing value ratios

The dataset size changed after the deduplication process:

| Stage | Record count |
|------|--------------:|
| Raw dataset | 26,630 |
| Cleaned dataset | 25,849 |
| Removed duplicates | 781 |

Duplicate status distribution in the cleaned dataset:

```text
False    25336
True       513
```

Overall, the cleaned dataset provides a standardized foundation for subsequent EDA.

---

### 6.2 Certification Year Trend Analysis

Certification records were grouped by certification year to examine temporal trends in medical device certifications.

The analysis included:

1. Calculating yearly certification counts
2. Generating a line chart to visualize changes over time

Certification Year Trend:

The yearly certification count increased rapidly from 2005 and peaked in 2008 (2,525 certifications), followed by a gradual decline over subsequent years. And certification counts remained relatively stable between 2017–2025, generally ranging from 700–1,000 records per year.



---

### 6.3 Certification Body Analysis

The original dataset only provided certification body codes (e.g., AA, AB, AC). 
The corresponding certification bodies were identified using publicly available PMDA information and related regulatory sources：https://www.mhlw.go.jp/stf/seisakunitsuite/bunya/kenkou_iryou/iyakuhin/touroku/index.html

To improve interpretability, a reference table was added containing:

| Field | Description |
|------|------|
| certification_body_code | Original code from dataset |
| certification_body_name | Full certification body name |
| certification_body_short_name | Simplified name for reporting |
| status | Active or abolished |
| abolished_date | Abolishment date (if applicable) |


Several certification bodies were found to have been abolished in previous years and were explicitly labeled in the reference table.

A certification body distribution visualization was then generated to compare certification volumes across certification bodies.


Certification Body Distribution:

Certification activities were concentrated among a few major certification bodies. Cosmos, TÜV SÜD, JET, SGS Japan, and BSI Japan accounted for the highest certification volumes, suggesting these organizations play dominant roles in the Japanese medical device certification system. 

---

### 6.4 Major Vendor Analysis

Major vendor analysis was conducted using:

```text
vendor_name_certified_person
```

The workflow included:

1. Identifying the Top 10 vendors by certification count and translating Japanese vendor names into English
3. Generating a pie chart visualization based on vendor proportions

The visualization provides major vendors include:

- FUJIFILM Corporation
- Olympus Medical Systems Corporation
- GE Healthcare Japan Corporation
- Canon Medical Systems Corporation
- OMRON Healthcare Co., Ltd.

---

## 7. Known Limitations, Edge Cases, and Quality Checks

### Known Limitations

- Some fields contain missing values because they are only applicable to specific records, such as succession-related fields and certification revocation dates.
- Several columns contain Japanese text, including product names and vendor names. English translations were only added where needed for reporting and visualization.
- Certification body codes were mapped to certification body names based on official reference information, but the original raw dataset only contains codes.

### Edge Cases

- Records with the same certification number may still represent different products or vendors, so deduplication was not based on certification number alone.
- Some certification bodies were abolished in previous years and were marked separately in the certification body reference table.

### Quality Checks Performed

The following checks were performed during cleaning and EDA:

- Checked dataset shape, column names, data types, and missing values.
- Reviewed missing values and preserved business-meaningful nulls.
- Identified duplicate groups using a composite key.
- Rechecked the dataset after deduplication and confirmed that no duplicate records remained.

---

## 8. Reproduction Steps

This project contains both the data cleaning workflow and the EDA workflow.

### 1. Set up the environment

Install project dependencies using `uv`:

```bash
uv sync
```

If dependencies need to be added manually:

```bash
uv add pandas numpy matplotlib openpyxl jupyter ipykernel
```

### 2. Run the cleaning workflow

Open and run the cleaning notebook:

```text
japan_cleaning.ipynb
```

This notebook uses `000280216.xlsx` as input and performs standardizes column names, handles missing values, formats date columns, performs deduplication, and exports the cleaned dataset.

Expected cleaned output:

```text
cleaned_japan.xlsx
```

Expected SQLite output:

```text
Japan_medical_devices.sqlite
```

### 3. Run the EDA workflow

Open and run the EDA notebook:

```text
japan_eda.ipynb
```

This notebook uses `cleaned_japan.xlsx` as input and performs dataset overview, certification year trend analysis, certification body distribution analysis, and major vendor analysis.

### 6. Review outputs

The main outputs include:

```text
data/processed/cleaned_japan.xlsx
database/Japan_medical_devices.sqlite
reference/certification_body_mapping.xlsx

outputs/certification_year_trend.png
outputs/certification_body_distribution.png
outputs/vendor_distribution.png
```

These outputs include the cleaned dataset, SQLite database, certification body mapping table, and EDA visualizations for certification trends, certification body distribution, and major vendor distribution.
