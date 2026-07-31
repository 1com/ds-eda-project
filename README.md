# Housing EDA Project — Bonnie Brown (Seller)

Exploratory data analysis on King County housing sales data, done for **Bonnie Brown**, a seller who already owns a house in a middle-class neighborhood and wants to move soon while maximizing her sale price. See [`00_assignment.md`](00_assignment.md) for the original assignment brief and client list.

The analysis answers three questions for her:

1. **Is her neighborhood actually middle class?**
2. **When should she sell** to get the best price?
3. **What can she do** (condition, renovation, etc.) to maximize profit?

## The Data

- Source: King County home sales dataset, loaded from the `eda` schema of the course database (see [`00_assignment.md`](00_assignment.md)).
- Raw file: `data/eda.csv` — not tracked in GitHub (per the assignment instructions), generated locally by connecting to the database.
- Column descriptions: [`column_names.md`](column_names.md).

## Repository Structure

Notebooks are numbered in the order they should be run/read:

| Notebook | Step | Description |
|---|---|---|
| [01_cleanfinal.ipynb](01_cleanfinal.ipynb) | 1. Cleaning | Cleans `data/eda.csv`: corrects a systemic ×10 error in `yr_renovated`, imputes missing `waterfront`/`view` values, derives `sqft_basement` where missing. Saves the result to `data/housing_cleaned.pkl`. |
| [02_middle_class_neighbourhoods.ipynb](02_middle_class_neighbourhoods.ipynb) | 2. Middle-class neighborhoods | Defines "middle class" for a zipcode with no income data available, using **average price per square foot per zipcode** as a proxy for the local price/location premium (rather than raw average price, which conflates house size with location). Middle-class zipcodes = the middle tercile (33rd–67th percentile) of that measure. |
| [03_hypothesis_1.ipynb](03_hypothesis_1.ipynb) | 3. When to sell | **Hypothesis 1: sale price varies through the year.** Defines its own middle-class zipcode segment (raw average price tercile, computed independently of notebook 02) and compares monthly average price within that segment against the full market. |
| [04_hypothesis_2.ipynb](04_hypothesis_2.ipynb) | 4. How to maximize profit | **Hypothesis 2: condition/grade/renovation affect sale price** (written in German by teammate Sonson). Defines its own middle-class zipcode segment (zipcode **median**-price tercile via `pd.qcut`, also independent of notebooks 02 and 03) and looks at grade/condition/renovation within it. |

> **Note:** notebooks 02, 03, and 04 each define "middle-class zipcode" independently and land on different zipcode sets — see [Assumptions & Limitations](#assumptions--limitations).

## Key Findings

### 1. Middle-class neighborhoods (notebook 02)
Zipcodes are ranked by average **price per square foot**, and the middle tercile (24 zipcodes) is taken as "middle class." Price/sqft was chosen over raw average price because it isolates the location premium from house size — a zipcode full of large houses can look expensive on average without commanding any real location premium, and vice versa for a zipcode of unusually small houses.

### 2. When to sell (timing) — notebook 03
Within notebook 03's own middle-class segment, average sale price peaks in **April** (~$541.9k) and is weakest in **September** (~$497.6k) — a swing of about **9%**. Full-market-wide, monthly average price and monthly sales volume move together strongly (correlation 0.86): higher-volume months also tend to have higher average prices.

**Recommendation:** list in spring, targeting an April close if possible.

> **Note:** the dataset only spans ~13 months (2014-05 to 2015-05), so this is described as "this year's pattern," not confirmed multi-year seasonality.

### 3. Maximizing profit (condition, grade, renovation) — notebook 04
Within notebook 04's own middle-class segment (zipcode median-price tercile — 22 zipcodes, ~6,791 homes, ~31% of the market):
- `grade` remains a strong driver of price (correlation 0.65, vs. 0.67 market-wide) — not just an artifact of some other factor.
- `condition` alone barely matters market-wide (correlation 0.04), but within this segment, `condition == 3` homes sell for about **10% less per sqft** than better-rated homes ($249 vs. $276/sqft).
- **Renovation:** among `condition == 3` homes in this segment, the 201 already-renovated ones (~4.5% of that group) sold for a **+36.7% higher median price** (+21.7% higher price/sqft) than non-renovated ones (correlation 0.14). The renovated sample is small and not randomly assigned — owners who renovate may also maintain their homes better in ways the data doesn't capture — so treat this as directional, not proof of causation.

**Recommendation:** grade/overall quality matters more than renovation history for pricing power in general. But if Bonnie's home is `condition == 3`, this data leans toward renovating before selling being worthwhile — provided her actual renovation costs stay well under the observed premium (median ~$170,500). This should be checked against a real cost estimate, since the dataset has no renovation-cost data.

## Assumptions & Limitations

- No income data exists for King County zipcodes, so **price/sqft (or, in notebooks 03/04, raw/median price) is used as a proxy for neighborhood class** — an explicit assumption, not a fact from the data.
- **Middle-class zipcode definitions are not reconciled across notebooks.** Notebooks 02, 03, and 04 each compute their own zipcode tercile independently (price/sqft, raw average price, and median price, respectively) and land on different zipcode sets. A prior notebook (`hypothesis_1_2.ipynb`) reran 03's and 04's analyses on notebook 02's final price/sqft segment and found the timing conclusion held up, but the renovation premium reversed to roughly **-5.6%/-3.2%** (i.e. no longer a benefit). That notebook has since been removed from the repo, so that reconciled result is no longer reproducible here — the renovation finding in Key Finding #3 above should be read as specific to notebook 04's own segment, not confirmed under notebook 02's definition.
- `03_hypothesis_1.ipynb` loads `data/eda.csv` directly rather than the cleaned pickle from notebook 01. This doesn't affect its result (it only uses `price`, `zipcode`, and `date`, none of which notebook 01 changes), but it means that notebook doesn't go through the same cleaning pipeline as 02/04.
- The ~13-month date range can't distinguish a one-year price trend from true seasonality.
- Sample sizes for the renovation analysis (~150–250 renovated homes) are on the smaller side; treat that finding as directional rather than definitive.
