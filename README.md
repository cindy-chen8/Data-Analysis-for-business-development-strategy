# Data-Analysis-for-business-development-strategy
Project for business development using a raw dataset of 200,000+ merchants scraped from Google Maps in a major Australian city, turning this messy and unorganised dataset into concrete and actionable strategy for Kpay 's BD Team.

---

## Project Context

**Company:** KPay — a fintech company offering $0 EFTPOS terminals, 1.4% flat transaction fee, same-day settlement, and multilingual support for Asian payment methods (UnionPay, Alipay, WeChat Pay, JCB). Launched in Australia October 2025.

**Dataset:** 199,999 merchant records scraped from Google Maps across NSW, Australia. Raw, messy, and unstructured — not usable for BD outreach in its original form.

---

## The Problem

KPay's BD team was handed a raw Google Maps scrape with no cleaning, no prioritisation, and no contact quality assessment. Without structure:

- BD reps would waste time calling fake or placeholder phone numbers
- Overseas records and irrelevant merchants pollute the outreach list
- No way to know which suburbs or sectors to visit first
- 52.5% of records had null suburbs — geo-based targeting was impossible

---

## What This Project Delivers

| Deliverable | Description |
|---|---|
| **Cleaned AU dataset** | 188,789 AU-only, KPay-relevant merchant records |
| **Tier 1 leads** | 23,700 high-priority merchants scored ≥ 80 KMF points |
| **Tier 2 leads** | 68,447 medium-priority merchants (score 50–79) |
| **Tier 3 leads** | 96,642 lower-priority merchants (score < 50) |
| **BD Canvassing Strategy** | Territory zones, pitch angles, 4-week sprint plan |
| **Dashboard Blueprint** | 5 KPIs for the sales manager to track the initiative |

---

## Project Structure

```
KPay_caseStudy/
│
├── data/
│   ├── Case Study - Raw Data.csv            # Raw input dataset (199,999 records)
│   ├── Case Study - Result Data.csv         # Raw dataset exploration first -- draft(199,999 records)
│   ├── Cleaned_AU_Data.csv                  # Final cleaned + scored dataset (188,789 records)
│   ├── Out_of_AU_Data.csv                   # Non-AU records filtered out (8,819 records)
│   └── Merchant priority/
│       ├── Tier_1_Merchants.csv             # 23,700 records — score ≥ 80
│       ├── Tier_2_Merchants.csv             # 68,447 records — score 50–79
│       └── Tier_3_Merchants.csv             # 96,642 records — score < 50
│
├── Case-Study-Clearning-Enrichment.ipynb    # Main improvement analysis notebook
├── Case-Study-Analysis-Exploration.ipynb    # Experience 1 draft analysis notebook
│
└── README.md
```

---

## Notebook Structure

The notebook (`Case-Study-Clearning-Enrichment.ipynb`) is organised into 7 sections:

| Section | Description |
|---|---|
| **1. Load & Inspect** | Load raw CSV, inspect shape, dtypes, and column distributions |
| **2. Data Quality Check** | Missing values, duplicates, phone patterns, state/sector distributions |
| **3. Business Impact Summary** | Quantify the BD cost of each data quality issue before cleaning |
| **4. Data Cleaning & Enrichment** | 6 cleaning steps + 6 enrichment columns (see below) |
| **5. Cleaning Funnel** | Row-count audit at each cleaning step |
| **6. KMF Scoring** | Score every record 0–100 across 4 components, assign tiers |

Plus three additional cells appended after the main flow:

- **Fix 3** — Smart country detection (address-based AU/non-AU classification)
- **Fix 2** — Suburb extraction from address column (95.3% fill rate)
- **Final re-score** — Re-run KMF on the fully cleaned dataset

| **Export** | Export cleaned CSV files |

---

## Data Cleaning — What Was Fixed

### Fix 1 — Sector Remapping
`sector_level_1` contained 146 unique label variations representing only 5 actual categories. Labels like `"Restaurants"`, `"Food & Drink"`, `"Food & Beverage"` all meant F&B but scored as unknown (6 pts instead of 35 pts).

**Approach:** Two-pass remapping — keyword matching on `sector_level_1` label variations, then keyword rescue of `Others`/null records using `sector_level_3` content. Fuzzy matching (`rapidfuzz`) handled edge cases.

**Result:** 59,091 records rescued from `Others`/null into correct categories. Zero unmapped labels remaining.

### Fix 2 — Suburb Extraction from Address
52.5% of records had null `suburb`. Without suburbs, geo-based scoring assigned minimum points to all of these records.

**Approach:** Regex extraction from the `address` column — find the state code, extract tokens before it, skip postcodes and street-type words, cross-validate candidates against an official 16,220-entry AU suburb reference list.

**Result:** 91,646 null suburbs filled (95.3% fill rate). Geographic data coverage increased from 47.5% to 93.3%.

### Fix 3 — Country Detection
The `state` column contained AU states, US state codes (NY, CA, FL etc.), full state names in inconsistent capitalisation, and garbage values. Simple filtering would have incorrectly removed 138 legitimate AU records.

**Approach:** Multi-signal classifier — check state value first, then interrogate the `address` column for AU/overseas signals (state codes, city names, country names, postcode patterns). Two output files produced.

**Result:** 191,180 confirmed AU records retained. 8,819 non-AU records separated. 138 records rescued that had null/bad state but clear AU addresses.

---

## Data Enrichment — Columns Created

Six new columns were engineered from the existing data. No external data sources were required.

| Column | Type | Description | BD Decision Unlocked |
|---|---|---|---|
| `phone_type` | string | 7-category classification via `phonenumbers` library: mobile, landline, shared_cost, toll_free, premium_rate, unknown | Phone type drives the reachability score — a mobile scores 30/30, a shared_cost 13xx scores 20/30 |
| `phone_quality` | string (L1/L2/L3) | 3-level quality score from 4 signals: library validity, phone↔lead_key cross-match, frequency analysis (>10 = fake), bad pattern detection | Level 1 = skip entirely. Level 3 = call first. Prevents wasting BD time on fake numbers |
| `reachability_score` | int (0–30) | Numeric score combining phone_quality + phone_type. Feeds KMF model directly | BD rep daily call list sorted by this column descending |
| `has_address` | bool | True if address is not null | Determines if merchant can be visited in person (+3 pts data completeness) |
| `has_sector` | bool | True if sector_level_1 is not null | Determines if pitch angle is known (+3 pts data completeness) |
| `kpay_relevant` | bool | False for sectors structurally unsuitable for EFTPOS (Retirement, NGO, Construction, Religious etc.) | Hard filter — these records never reach the BD team |
| `name_suburb_dup` | bool | True if business_name + suburb duplicates a previous record | Prevents two BD reps calling the same merchant independently |
| `state_clean` | string | Normalised AU state code (NSW, VIC, QLD etc.) | Consistent state field for geo filtering and reporting |
| `suburb` (filled) | string | Suburb extracted from address where previously null | Enables geo scoring for 91,646 additional records |
| `sector_level_1_clean` | string | Standardised sector label mapped to 5 clean categories | Correct sector scoring in KMF model |

---

## KMF Scoring Model

The **KPay Merchant Fit (KMF) Index** scores every merchant 0–100 across four components:

| Component | Max Points | Logic |
|---|---|---|
| **Reachability** | 40 pts | `reachability_score` (0–30) + `has_address` × 10 |
| **Sector Priority** | 35 pts | F&B = 35, Beauty & Wellness = 26, Retail = 26, Professional Services = 12, Retail - Electronics = 12, Others = 9 |
| **Geo Efficiency** | 15 pts | Tier A suburbs (Hurstville, Chatswood etc.) = 15, Tier B suburbs = 10, Sydney area = 5, other = 1 |
| **Data Completeness** | 10 pts | `has_address` × 3 + `has_sector` × 3 + `phone_match_score` × 4 |

**Tier thresholds:**

| Tier | Score Range | Count | % of Dataset | Outreach Method |
|---|---|---|---|---|
| Tier 1 | ≥ 80 | 23,700 | 12.6% | In-person visits, Week 1 |
| Tier 2 | 50–79 | 68,447 | 36.3% | Phone outreach, Weeks 2–3 |
| Tier 3 | < 50 | 96,642 | 51.2% | Email / digital, Month 2+ |

---

## Key Insight

**8 of the top 10 Tier 1 suburbs are Asian-Australian community centres** — Chatswood, Hurstville, Haymarket, Cabramatta, Bankstown, Auburn, Burwood, Eastwood, Ashfield, and Campsie.

This is not coincidence. KPay's multilingual support (Mandarin, Cantonese, Vietnamese) and native UnionPay/Alipay/WeChat Pay acceptance creates a direct competitive moat in these suburbs that no other EFTPOS provider in Australia can match. The territory design exploits this advantage by prioritising Zone A (Asian Community Core) suburbs in Week 1 of the BD sprint.

---

## Tech Stack

| Tool | Usage |
|---|---|
| Python 3.11 | Core analysis language |
| pandas | Data manipulation and cleaning |
| phonenumbers | Phone number parsing, validation, and type classification |
| rapidfuzz | Fuzzy string matching for sector label remapping |
| re | Regex for address parsing and phone pattern detection |
| urllib.request | Loading AU suburb reference list from GitHub |
| matplotlib / seaborn | Visualisation |
| Jupyter Notebook | Interactive analysis environment |

---

## How to Run

```bash

# 1. Install dependencies
pip install pandas phonenumbers rapidfuzz matplotlib seaborn

# 2. Place the raw dataset
# Copy "Case Study - Raw Data.csv" into the data/ folder

# 3. Open and run the notebook top to bottom
jupyter notebook Case-Study-Clearning-Enrichment.ipynb
```

---

## Output Files

| File | Rows | Description |
|---|---|---|
| `Cleaned_AU_Data.csv` | 188,789 | Master cleaned dataset with all 30 columns |
| `Out_of_AU_Data.csv` | 8,819 | Non-AU records for reference |
| `Tier_1_Merchants.csv` | 23,700 | KMF score ≥ 80 — BD priority this week |
| `Tier_2_Merchants.csv` | 68,447 | KMF score 50–79 — second wave |
| `Tier_3_Merchants.csv` | 96,642 | KMF score < 50 — long tail |

---

## Author

**Cindy Chen**
KPay Data Analytics Intern Case Study — March 2026