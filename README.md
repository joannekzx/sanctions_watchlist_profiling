# OFAC SDN Watchlist — Advanced Data Profiling & Analytics
  
**Dataset:** OFAC Specially Designated Nationals (SDN) List  
**Source:** https://sanctionslist.ofac.treas.gov  
**Tools:** Python, pandas, matplotlib, seaborn, networkx, fuzzywuzzy

---

## Objective

This project performs end-to-end data profiling and analysis of the OFAC Specially Designated Nationals (SDN) sanctions watchlist — mirroring real-world compliance data analyst workflows in Anti-Money Laundering (AML) and sanctions screening.

The OFAC SDN list is one of the most widely used watchlists globally. Financial institutions are legally required to screen all transactions against it to prevent dealings with sanctioned individuals, entities, vessels, and aircraft. Poor data quality in the watchlist directly increases false-negative risk — meaning sanctioned actors can evade detection.

---

## Installation

```bash
pip install pandas numpy matplotlib seaborn networkx plotly fuzzywuzzy python-Levenshtein
```

---

## Analysis Overview

### 1. Data Loading & Preprocessing
The raw OFAC SDN CSV has no column headers and uses `-0-` as a proprietary placeholder for missing values. The loading step defines column names, strips whitespace, maps `-0-` entity types to `entity`, and replaces all remaining `-0-` values with `NaN`. Structured fields including nationality, date of birth, place of birth, passport number, and alias count are extracted from the unstructured `remarks` column using regex.

### 2. Data Cleaning
A dedicated cleaning pipeline transforms the raw dataset into an analysis-ready `df_clean`:

- Casing standardisation — entity names converted to Title Case, `sdn_type` lowercased
- Nationality normalisation — adjective forms mapped to country names (e.g. `Iranian` to `Iran`)
- Deduplication — exact duplicate rows removed
- `sdn_type` validation — values outside `individual`, `entity`, `vessel`, `aircraft` tagged as `unknown`
- DOB validation — implausible birth years (before 1900 or after 2015) nullified
- Data quality flagging — each record tagged with specific issues such as `MISSING_DOB`, `MISSING_NATIONALITY`, or `MISSING_PASSPORT` for downstream prioritisation

### 3. Advanced Data Quality Assessment
Field-level completeness is measured and visualised across nine key fields. A four-dimension Data Quality Scorecard assesses Completeness (55.7%), Uniqueness (100%), Consistency (14.3%), and Validity (100%).

The low Consistency score reflects that most program entries do not follow the standard bracket format, requiring normalisation before ingestion into screening systems. The low Completeness score — particularly for identity fields like DOB, nationality, and passport — represents a material screening risk: records missing these fields are harder to match accurately and increase false-negative rates.

### 4. Entity & Program Profiling
Entity type distribution, top sanctions programs by volume, nationality breakdown, and alias density analysis. Key finding: 55%+ of records have at least one alias, requiring fuzzy matching rather than exact string matching in screening systems.

### 5. Name Analysis
Text-based analysis of name characteristics across the watchlist including length distribution, token count, and prevalence of special characters, digits, and all-caps formatting. The high rate of special characters (~10,000 records) and all-caps formatting confirms that raw watchlist data cannot be used directly in screening without normalisation — casing and punctuation variation would cause exact-match systems to miss sanctioned entities.

A corporate keyword frequency analysis on entity names identifies which legal suffixes and business terms are most common, informing how screening rules should handle entity name variants.

### 6. Fuzzy Name Matching & Threshold Sensitivity
A fuzzy screening engine is implemented using `fuzzywuzzy` to simulate real-world name matching. The engine compares query names against the full watchlist using token sort ratio, which handles word order variation.

A threshold sensitivity analysis demonstrates the precision-recall tradeoff: lowering the similarity threshold increases recall (catches more matches) but generates more false positives. Raising the threshold reduces alert volume but risks missing near-matches. In AML screening, false negatives carry greater regulatory risk than false positives — this tradeoff directly informs threshold calibration decisions.

### 7. Time Series Analysis
Birth decade and age distribution of sanctioned individuals reveal that the majority were born in the 1960s and 1970s, with a median age of 55 as of 2025. This is consistent with sanctions predominantly targeting established actors — senior government officials, business figures, and criminal network leaders — rather than low-level operatives.

### 8. Network Graph — Program Co-occurrence
A program co-occurrence network maps which sanctions regimes share sanctioned entities. Two distinct ecosystems emerge: an Iran/terrorism cluster (SDGT, IRAN, NPWMD, IFSR, IRGC) and a Russia/cyber cluster (RUSSIA-EO14024, UKRAINE, CAATSA, CYBER2, CYBER4). SDGT acts as the primary bridge between the two clusters.

Entities appearing in multiple programs represent the highest-risk actors and should be weighted accordingly in risk scoring models.

### 9. Nationality x Program Bipartite Network
A bipartite network links nationalities to their associated sanctions programs, revealing geopolitical targeting patterns. Iranian nationals appear across six or more programs simultaneously, while Mexican nationals appear exclusively under drug trafficking programs (SDNTK). Nationality is a strong predictor of program type and can serve as a feature in downstream risk scoring models.

---

## Key Findings

| Finding | Implication |
|---|---|
| 55.7% average field completeness | High null rates in identity fields increase false-negative risk in automated screening |
| 14.3% formatting consistency | Program field inconsistency requires standardised parsing before ingestion |
| 55%+ of entities have aliases | Exact-match screening is insufficient — fuzzy matching is required |
| Special characters in ~10,000 names | Punctuation normalisation is critical for accurate name matching |
| RUSSIA-EO14024 accounts for 4,159 entities | Largest single sanctions program — reflects post-2022 Ukraine invasion sanctions |
| Two distinct network clusters | Iran/terrorism and Russia/cyber ecosystems overlap minimally — different screening rules may be warranted per cluster |
| Nationality predicts program type | Nationality can be used as a risk feature in entity scoring models |

---

## Business Implications

Watchlist data quality directly impacts sanctions screening accuracy. Missing identity fields, inconsistent formatting, and alias complexity are not cosmetic issues — they translate directly into compliance risk. Financial institutions relying on exact-match or poorly normalised screening systems face elevated exposure to regulatory penalties for missed sanctions hits.

The fuzzy matching threshold analysis demonstrates that there is no universally correct threshold — the optimal value depends on the institution's risk appetite, the specific sanctions program, and the volume of alerts its compliance team can review. A threshold that is too strict will miss sanctioned entities; one that is too loose will generate unmanageable alert volumes.

The network analysis suggests that multi-program entities represent disproportionately high risk and should be flagged with elevated priority scores in any screening or risk management system.

---

## Data Source

OFAC Specially Designated Nationals (SDN) List  
U.S. Department of the Treasury, Office of Foreign Assets Control  
https://sanctionslist.ofac.treas.gov/Home/SdnList  
Data is updated regularly — results will vary depending on download date.