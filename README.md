# Car Sales Performance Analysis

> A hands-on Python EDA of 23,906 car sales transactions from January 2022 through December 2023. The dataset has some genuinely messy corners -- a Gender column with 13 different spellings of two words, a UTF-8 encoding corruption in the Engine field, and a phantom duplicate region caused by a trailing space. Working through each issue methodically is what this project is really about.

---
## 📋 Table of Contents

- [What this project covers](#what-this-project-covers)
- [Dataset at a glance](#dataset-at-a-glance)
- [Project structure](#project-structure)
- [How the pipeline works](#how-the-pipeline-works)
- [What we found](#what-we-found)
- [Requirements](#requirements)
- [Getting started](#getting-started)
- [Notebook walkthrough](#notebook-walkthrough)
- [Design decisions](#design-decisions)
- [Business takeaways](#business-takeaways)

---

## What this project covers

1. Loading and auditing raw data, including the kind of issues that `.isna()` will not catch
2. Nine targeted cleaning steps, each with a before/after check or assertion
3. Feature engineering: time dimensions, income bands, price tiers
4. Exploratory analysis across brand, region, body style, gender, colour, and time
5. Ten visualisations covering every major dimension
6. A summary dashboard and practical business recommendations

---

## Dataset at a glance

| Property | Value |
|---|---|
| File | `car_sales.csv` |
| Rows | 23,906 transactions |
| Columns | 17 raw, 16 after dropping constant Qty column |
| Period | January 2022 to December 2023 |
| Scope | Multi-dealer car sales across 7 US regions |

### Column reference

| Column | Raw type | Clean type | Notes |
|---|---|---|---|
| Car_id | str | str | Unique transaction ID |
| Date | str | datetime | M/D/YYYY format, parsed on clean |
| Customer Name | str | str | One true NaN filled with Unknown |
| Gender | str | str | 13 dirty variants collapsed to Male / Female |
| Annual Income | int64 | int64 | Customer annual income in USD |
| Dealer_Name | str | str | Dealership name |
| Company | str | str | Car manufacturer |
| Model | str | str | Car model name |
| Engine | str | str | UTF-8 encoding artefact repaired |
| Transmission | str | str | Auto or Manual |
| Color | str | str | 9 raw variants cleaned to 3 canonical values |
| Price | int64 | int64 | Sale price in USD, renamed from Price ($) |
| Dealer_No | str | str | Trailing space removed from raw column name |
| Body Style | str | str | SUV / Hatchback / Sedan / Passenger / Hardtop |
| Phone | int64 | int64 | Customer phone number |
| Dealer_Region | str | str | Trailing space on Pasco collapsed phantom duplicate |
| Qty | int64 | dropped | Every row is 1, zero variance |

### Data quality issues found

| Column | Problem | Fix |
|---|---|---|
| Column names | Dealer_No had trailing space; Price ($) had special characters | df.rename() |
| Customer Name | 1 true NaN | Filled with Unknown |
| Gender | 13 dirty variants of Male and Female | Normalise by first character of stripped lowercase value |
| Color | 9 variants for 3 colours: typos, trailing backslash, double space | Strip artefacts then apply correction map |
| Dealer_Region | Trailing space on Pasco created a phantom 8th region | str.strip() |
| Engine | UTF-8 mis-encoding of a non-breaking space | Replace the corrupt byte sequence |
| Date | Stored as string in M/D/YYYY format | pd.to_datetime with explicit format |
| Qty | Every row is 1, zero variance | Drop the column |

---

## Project structure

```
car-sales-eda/
+-- car_sales.csv                  # Raw data (not in version control)
+-- car_sales_analysis.ipynb       # Main notebook
+-- car_sales_analysis.png         # Composite output chart
+-- README.md                      # This file
+-- requirements.txt               # Dependencies
```

Raw data is not committed to version control. Place `car_sales.csv` in the project root before running the notebook.

---

## How the pipeline works

```
Raw CSV
  |
  +-- Stage 1: Load and audit
  |       Shape, dtypes, null counts, unique value scan per column
  |       Focused deep-dive on the four dirtiest columns
  |
  +-- Stage 2: Clean (9 steps)
  |       Rename columns with trailing spaces and special characters
  |       Fill the one NaN in Customer Name
  |       Normalise Gender: 13 variants -> Male / Female
  |       Fix Color: 9 variants -> 3 canonical values
  |       Strip Dealer_Region whitespace: 8 apparent -> 7 real regions
  |       Repair Engine UTF-8 encoding artefact
  |       Parse Date string (M/D/YYYY -> datetime)
  |       Drop constant Qty column
  |       Assert: zero nulls, exact colour set, exact region count
  |
  +-- Stage 3: Feature engineering
  |       Year, Month, Month Name, Quarter, Day of Week
  |       Income Band (pd.cut, 5 tiers)
  |       Price Band (pd.cut, 5 tiers)
  |
  +-- Stage 4: EDA aggregations
  |       Revenue by brand, body style, region, gender,
  |       colour, transmission, day of week
  |       Monthly YoY comparison
  |       Brand x Body Style cross-table
  |
  +-- Stage 5: Visualisations (10 charts)
  |
  +-- Stage 6: KPI summary
  |
  +-- Stage 7: Business takeaways
```

---

## What we found

| Metric | Value |
|---|---|
| Total Revenue | $671.53M |
| Total Units Sold | 23,906 |
| Avg Sale Price | $28,090 |
| Median Sale Price | $23,000 |
| Top Brand | Chevrolet ($47.66M) |
| Top Dealer Region | Austin ($117.19M, 17.4% of total) |
| Top Body Style | SUV |
| Most Popular Colour | Pale White (11,256 units, 47.1%) |
| Male Revenue Share | 78.5% |
| Female Revenue Share | 21.5% |
| Auto Transmission | 52.9% of revenue |
| Best Month | December |

Four patterns worth calling out: Chevrolet, Dodge and Ford are essentially tied at the top -- no single brand dominates. Austin generates 17.4% of all revenue despite being one of seven equal regions. December spikes sharply in both years. And Engine type and Transmission are perfectly correlated -- they are one variable pretending to be two.

---

## Requirements

| Library | Version |
|---|---|
| pandas | 2.0+ |
| numpy | 1.25+ |
| matplotlib | 3.7+ |
| seaborn | 0.13+ |
| jupyter | 1.0+ |

---

## Getting started

```bash
git clone https://github.com/SyusufWaliyyi/car_sales_analysis.git
cd car_sales_analysis

python -m venv .venv
source .venv/bin/activate

pip install -r requirements.txt

jupyter lab car_sales_analysis.ipynb
```

**requirements.txt**

```
pandas>=2.0
numpy>=1.25
matplotlib>=3.7
seaborn>=0.13
jupyter>=1.0
```

---

## Notebook walkthrough

| Stage | Cells | What happens |
|---|---|---|
| 0 Setup | 1 | Imports, colour palette, matplotlib defaults |
| 1 Load and audit | 5 | Read CSV, inspect types, null count, per-column unique values, focused dirty-column review |
| 2 Cleaning | 9 | One cell per issue, each with evidence of before and after state, plus a final assertion block |
| 3 Feature engineering | 1 | Date parts, Income Band, Price Band |
| 4 EDA | 5 | All groupby aggregations, printed and inspectable before any chart is drawn |
| 5 Visualisations | 10 | One chart per cell: brand, body style, YoY trend, region, gender, transmission, colour, boxplot, day of week, heatmap |
| 6 Summary | 1 | KPI dictionary printed as a formatted table |
| 7 Takeaways | Markdown | Six numbered findings with plain-English interpretation |

---

## Design decisions

### Normalising Gender without a lookup table

The Gender column has 13 distinct dirty variants of just two words. A lookup table approach would need to explicitly list every variant and would fail silently any time a new typo appeared in future data.

The cleaner approach classifies by the first character of the stripped, lowercased value:

```python
def normalise_gender(val: str) -> str:
    v = str(val).strip().lower()
    if v.startswith('m'):
        return 'Male'
    if v.startswith('f'):
        return 'Female'
    return 'Unknown'
```

Every real-world variant of Male starts with m, and every real-world variant of Female starts with f. The Unknown fallback ensures genuinely ambiguous values surface as a distinct category rather than being silently misassigned.

### What actually caused the Engine corruption

The Engine column contains a two-character artefact embedded in the string. This is a Windows-1252 to UTF-8 charset mis-conversion: a non-breaking space from Windows-1252 was mis-interpreted as the start of a multi-byte UTF-8 sequence, producing a corrupted output string. The fix targets the exact byte sequence with `regex=False` so the replacement is treated as a literal rather than a regex pattern.

### The Dealer_Region phantom duplicate

Calling `value_counts()` on Dealer_Region shows eight regions even though there should be seven. The issue is a single trailing space: the string with the space is distinct from the string without it in pandas, so all rows with that value are silently split into a separate group. One `str.strip()` call merges them. The assertion that follows enforces that exactly seven regions remain.

### Why we drop Qty rather than keep it

Every row in the Qty column contains the value 1. A column with zero variance contains no information -- it cannot distinguish between rows, cannot be used in any meaningful aggregation, and will only add noise to downstream modelling. Dropping it is the right call.

### Operation order matters for Color cleaning

The Color column has three overlapping issues: trailing backslashes, extra whitespace, and typos. The operations need to run in this specific order:

1. `str.strip()` removes leading and trailing whitespace
2. `str.rstrip()` with a backslash argument removes any trailing backslash
3. `str.strip()` again clears whitespace exposed by the backslash removal
4. `.replace(color_map)` fixes the remaining typos

Putting the map replacement before the stripping would cause lookups to fail because the keys would not match strings that still have whitespace attached.

### Assertions as a regression safety net

The cleaning pipeline ends with five assert statements that check post-conditions. If the source data is ever refreshed and introduces new dirty patterns, the pipeline fails loudly at the assertion rather than silently carrying bad values through into the aggregations and charts.

---

## Business takeaways

**Austin is an outlier and that is worth understanding.** Generating 17.4% of total revenue from one of seven regions is a meaningful gap. Before attributing it to luck, it is worth examining whether Austin dealers have a different product mix, pricing strategy, or marketing approach that can be replicated elsewhere.

**The gender gap is the largest commercial opportunity in the dataset.** Male customers account for 78.5% of revenue. Whether that reflects who is being targeted, who is being served well in the showroom, or simply the product catalogue on offer, narrowing it by even a few percentage points would move meaningful revenue.

**December is predictable -- plan around it.** Both years show the same sharp end-of-year spike. Inventory and staffing decisions made in October and November will determine how much of that demand actually gets captured.

**Pale White dominates, but that creates risk.** Nearly half of all units sold are Pale White. If supplier availability on that colour were to tighten, the business would feel it quickly.

**Chevrolet, Dodge and Ford being this close together is interesting.** The near-parity means small changes in how each brand is positioned could have material effects on mix and margin -- more room to manoeuvre than a market with a clear dominant brand.

**Treat Engine and Transmission as one feature.** They carry identical information. Any future modelling work should use one or the other, not both.

---
