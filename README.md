# car\_sales\_analysis



\# Car Sales Performance Analysis



A hands-on Python EDA of 23,906 car sales transactions from January 2022 through December 2023. The dataset has some genuinely messy corners -- a Gender column with 13 different spellings of two words, a UTF-8 encoding corruption in the Engine field, and a phantom duplicate region caused by a trailing space. Working through each issue methodically is what this project is really about.



---



\## Table of contents



\- \[What this project covers](#what-this-project-covers)

\- \[Dataset at a glance](#dataset-at-a-glance)

\- \[Project structure](#project-structure)

\- \[How the pipeline works](#how-the-pipeline-works)

\- \[What we found](#what-we-found)

\- \[Requirements](#requirements)

\- \[Getting started](#getting-started)

\- \[Notebook walkthrough](#notebook-walkthrough)

\- \[Design decisions](#design-decisions)

\- \[Business takeaways](#business-takeaways)



---



\## What this project covers



1\. Loading and auditing raw data -- including the kind of issues that `.isnull()` will not catch

2\. Nine targeted cleaning steps, each with a before/after check or assertion

3\. Feature engineering -- time dimensions, income bands, price tiers

4\. Exploratory analysis across brand, region, body style, gender, colour, and time

5\. Ten visualisations covering every major dimension

6\. A summary dashboard and practical business recommendations



---



\## Dataset at a glance



| Property | Value |

|---|---|

| File | `car\_sales.csv` |

| Rows | 23,906 transactions |

| Columns | 17 raw, 16 after dropping constant Qty column |

| Period | January 2022 to December 2023 |

| Scope | Multi-dealer car sales across 7 US regions |



\### Column reference



| Column | Raw type | Clean type | Notes |

|---|---|---|---|

| Car\_id | str | str | Unique transaction ID |

| Date | str | datetime | M/D/YYYY format -- parsed on clean |

| Customer Name | str | str | One true NaN filled with Unknown |

| Gender | str | str | 13 dirty variants collapsed to Male / Female |

| Annual Income | int64 | int64 | Customer annual income in USD |

| Dealer\_Name | str | str | Dealership name |

| Company | str | str | Car manufacturer |

| Model | str | str | Car model name |

| Engine | str | str | UTF-8 encoding artefact repaired |

| Transmission | str | str | Auto or Manual |

| Color | str | str | 9 raw variants cleaned to 3 canonical values |

| Price ($) | int64 | int64 renamed Price | Sale price in USD |

| Dealer\_No | str | str | Trailing space removed from column name |

| Body Style | str | str | SUV / Hatchback / Sedan / Passenger / Hardtop |

| Phone | int64 | int64 | Customer phone number |

| Dealer\_Region | str | str | Trailing space on Pasco collapsed phantom duplicate |

| Qty | int64 | dropped | Every row is 1 -- zero variance |



\### Data quality issues found



| Column | What was wrong | How we fixed it |

|---|---|---|

| Column names | `Dealer\_No ` trailing space; `Price ($)` special chars | `df.rename()` |

| Customer Name | 1 true NaN | Filled with Unknown |

| Gender | 13 dirty variants: `Male `, `Maleee`, `fFemale`, `M`, `Fem` etc. | Normalise by first character of stripped lowercase value |

| Color | 9 variants for 3 colours -- typos, trailing backslash, double space | Strip then apply correction map |

| Dealer\_Region | Trailing space on Pasco created an 8th phantom region | `str.strip()` |

| Engine | UTF-8 mis-encoding of a non-breaking space | Replace the exact corrupt byte sequence |

| Date | String in M/D/YYYY -- not parsed on load | `pd.to\_datetime` with explicit format |

| Qty | Every single row is 1 | Drop the column |



---



\## Project structure



```

car-sales-eda/

+-- car\_sales.csv                  # Raw data (not in version control -- see below)

+-- car\_sales\_analysis.ipynb       # Main notebook

+-- car\_sales\_analysis.png         # Composite output chart

+-- README.md                      # This file

+-- requirements.txt               # Dependencies

```



> Raw data is not committed to version control. Place `car\_sales.csv` in the project root before running the notebook.



---



\## How the pipeline works



```

Raw CSV

&nbsp; |

&nbsp; +-- Stage 1: Load and audit

&nbsp; |       Shape, dtypes, null counts, unique value scan per column

&nbsp; |       Focused deep-dive on the four dirtiest columns

&nbsp; |

&nbsp; +-- Stage 2: Clean (9 steps)

&nbsp; |       Rename columns with trailing spaces / special characters

&nbsp; |       Fill the one NaN in Customer Name

&nbsp; |       Normalise Gender: 13 variants -> Male / Female

&nbsp; |       Fix Color: 9 variants -> 3 canonical values

&nbsp; |       Strip Dealer\_Region whitespace: 8 apparent -> 7 real regions

&nbsp; |       Repair Engine UTF-8 encoding artefact

&nbsp; |       Parse Date string (M/D/YYYY -> datetime)

&nbsp; |       Drop constant Qty column

&nbsp; |       Assert: zero nulls, exact colour set, exact region count

&nbsp; |

&nbsp; +-- Stage 3: Feature engineering

&nbsp; |       Year, Month, Month Name, Quarter, Day of Week

&nbsp; |       Income Band (pd.cut, 5 tiers)

&nbsp; |       Price Band (pd.cut, 5 tiers)

&nbsp; |

&nbsp; +-- Stage 4: EDA aggregations

&nbsp; |       Revenue by brand, body style, region, gender,

&nbsp; |       colour, transmission, day of week

&nbsp; |       Monthly YoY comparison

&nbsp; |       Brand x Body Style cross-table

&nbsp; |

&nbsp; +-- Stage 5: Visualisations (10 charts)

&nbsp; |

&nbsp; +-- Stage 6: KPI summary

&nbsp; |

&nbsp; +-- Stage 7: Business takeaways

```



---



\## What we found



| Metric | Value |

|---|---|

| Total Revenue | $671.53M |

| Total Units Sold | 23,906 |

| Avg Sale Price | $28,090 |

| Median Sale Price | $23,000 |

| Top Brand | Chevrolet ($47.66M) |

| Top Dealer Region | Austin ($117.19M -- 17.4% of total) |

| Top Body Style | SUV |

| Most Popular Colour | Pale White (11,256 units -- 47.1%) |

| Male Revenue Share | 78.5% |

| Female Revenue Share | 21.5% |

| Auto Transmission | 52.9% of revenue |

| Best Month | December |



\*\*Four patterns worth calling out:\*\*



Chevrolet, Dodge and Ford are essentially tied at the top of the revenue table -- no single brand dominates. Austin generates 17.4% of all revenue despite being one of seven equal regions, which is a gap large enough to warrant a proper root-cause investigation. December spikes sharply in both years, but 2023 showed stronger mid-year performance between May and September compared to 2022. And Engine type and Transmission are perfectly correlated -- they are one variable pretending to be two.



---



\## Requirements



| Library | Version |

|---|---|

| pandas | 2.0+ |

| numpy | 1.25+ |

| matplotlib | 3.7+ |

| seaborn | 0.13+ |

| jupyter | 1.0+ |



---



\## Getting started



```bash

\# Clone

git clone https://github.com/SyusufWaliyyi/car\_sales\_analysis.git

cd car\_sales\_analysis



\# Virtual environment

python -m venv .venv

source .venv/bin/activate      # macOS / Linux

.venv\\Scripts\\activate         # Windows



\# Install

pip install -r requirements.txt



\# Run

jupyter lab car\_sales\_analysis.ipynb

```



---



\## Notebook walkthrough



| Stage | Cells | What happens |

|---|---|---|

| 0 -- Setup | 1 | Imports, colour palette, matplotlib defaults |

| 1 -- Load and audit | 5 | Read CSV, inspect types, null count, per-column unique values, focused dirty-column review |

| 2 -- Cleaning | 9 | One cell per issue, each with evidence of before and after state, plus a final assertion block |

| 3 -- Feature engineering | 1 | Date parts, Income Band, Price Band |

| 4 -- EDA | 5 | All groupby aggregations, printed and inspectable before any chart is drawn |

| 5 -- Visualisations | 10 | One chart per cell -- brand, body style, YoY trend, region, gender, transmission, colour, boxplot, day of week, heatmap |

| 6 -- Summary | 1 | KPI dictionary printed as a formatted table |

| 7 -- Takeaways | Markdown | Six numbered findings with plain-English interpretation |



---



\## Design decisions



\### Normalising Gender without a lookup table



The Gender column has 13 distinct dirty variants of just two words. A lookup table approach would need to explicitly list every variant and would fail silently any time a new typo appeared in future data.



The cleaner approach is to classify by the first character of the stripped, lowercased value:



```python

def normalise\_gender(val: str) -> str:

&nbsp;   v = str(val).strip().lower()

&nbsp;   if v.startswith('m'):

&nbsp;       return 'Male'

&nbsp;   if v.startswith('f'):

&nbsp;       return 'Female'

&nbsp;   return 'Unknown'

```



Every real-world variant of Male starts with m, and every real-world variant of Female starts with f. The Unknown fallback ensures genuinely ambiguous values surface as a distinct category rather than being silently misassigned.



\### What actually caused the Engine corruption



The Engine column contains a two-character artefact that should not be there. This is a Windows-1252 to UTF-8 charset mis-conversion. The non-breaking space character from Windows-1252 (byte value 0xA0) was mis-interpreted as the start of a multi-byte UTF-8 sequence, producing a capital A-with-circumflex character followed by 0xA0 in the output.



The fix targets the exact byte sequence:



```python

df\['Engine'] = df\['Engine'].str.replace(artefact\_sequence, ' ', regex=False).str.strip()

```



Using `regex=False` is important here. Without it, the backslash in the byte reference would be treated as a regex escape and the replacement would fail or match the wrong thing.



\### The Dealer\_Region phantom duplicate



Calling `value\_counts()` on Dealer\_Region shows eight regions even though there should be seven. The issue is a single trailing space: the string `Pasco ` is distinct from `Pasco` in pandas, so all three of its records are silently counted separately. One `str.strip()` call merges them back together. The assertion that follows enforces that exactly seven regions remain.



\### Why we drop Qty rather than keep it



Every row in the Qty column contains the value 1. A column with zero variance contains no information -- it cannot distinguish between rows, cannot be used in any meaningful aggregation, and will only add noise to any downstream modelling. Dropping it is the right call.



\### Operation order matters for Color cleaning



The Color column has three overlapping issues: trailing backslashes, extra whitespace, and outright typos. The operations need to run in this specific order:



1\. `str.strip()` -- clears leading and trailing whitespace

2\. `str.rstrip()` with a backslash argument -- removes any trailing backslash

3\. `str.strip()` -- clears whitespace that the backslash removal may have exposed

4\. `.replace(color\_map)` -- fixes the remaining typos



Putting the map replacement before the stripping would cause the lookups to fail because the keys would not match strings that still have extra whitespace attached.



\### Assertions as a regression safety net



The cleaning pipeline ends with five assert statements:



```python

assert df.isnull().sum().sum() == 0

assert set(df\['Gender'].unique()) <= {'Male', 'Female', 'Unknown'}

assert set(df\['Color'].unique()) == {'Black', 'Red', 'Pale White'}

assert df\['Engine'].str.contains('encoding\_artefact').sum() == 0

assert df\['Dealer\_Region'].nunique() == 7

```



These are not just sanity checks -- they are a regression test. If the source data is ever refreshed and introduces new dirty patterns, the pipeline fails loudly at the assertion rather than silently carrying bad values through into the aggregations and charts.



---



\## Business takeaways



\*\*Austin is an outlier and that is worth understanding.\*\* Generating 17.4% of total revenue from one of seven regions is a meaningful gap. Before attributing it to luck, it is worth looking at whether Austin dealers have a different product mix, pricing strategy, or marketing approach that can be replicated elsewhere.



\*\*The gender gap is the largest commercial opportunity in the dataset.\*\* Male customers account for 78.5% of revenue. Whether that gap reflects who is being targeted, who is being served well in the showroom, or simply the product catalogue on offer, narrowing it by even a few percentage points would move meaningful revenue.



\*\*December is predictable -- plan around it.\*\* Both years show the same sharp end-of-year spike. Inventory and staffing decisions made in October and November will determine how much of that demand actually gets captured.



\*\*Pale White dominates, but that creates risk.\*\* Nearly half of all units sold are Pale White. If supplier availability on that colour were to tighten, the business would feel it quickly. Having a clearer picture of how interchangeable the three colours are for buyers would be useful.



\*\*Chevrolet, Dodge and Ford being this close together is actually interesting.\*\* In a dataset where one brand clearly dominated, the inventory and marketing strategy would be obvious. Here the near-parity means small changes in how each brand is positioned could have material effects on the mix.



\*\*Treat Engine and Transmission as one feature.\*\* They carry identical information. Any future modelling work should use one or the other, not both.



---





