# Netflix Datasets: Cleaned

Cleaned versions of two Netflix datasets: a catalogue of titles and a customer churn table. Original files are unchanged; all cleaning steps are listed in `cleaning_log.csv`.

## Files

| File | Rows | Columns | Description |
|---|---|---|---|
| `netflix_titles_clean.csv` | 8,807 | 15 | Movies and TV shows on Netflix |
| `netflix_customer_churn_clean.csv` | 5,000 | 16 | Customer subscription and usage records with a churn flag |
| `cleaning_log.csv` | 11 | 4 | Every issue found, rows affected, and action taken |

No rows were removed from either file.

---

## 1. netflix_titles_clean.csv

One row per title. `show_id` is unique.

| Column | Type | Description |
|---|---|---|
| `show_id` | text | Unique ID (s1, s2, ...) |
| `type` | text | `Movie` or `TV Show` |
| `title` | text | Title name |
| `director` | text | Director(s), comma-separated. `Unknown` if missing |
| `cast` | text | Cast, comma-separated. `Unknown` if missing |
| `country` | text | Production country/countries, comma-separated. `Unknown` if missing |
| `date_added` | date | Date added to Netflix, `YYYY-MM-DD`. Blank if missing |
| `release_year` | integer | Original release year |
| `rating` | text | Content rating (TV-MA, PG-13, R, NR, ...). `Unknown` if missing |
| `listed_in` | text | Genre categories, comma-separated |
| `description` | text | Short synopsis |
| `year_added` | integer | **New.** Year from `date_added` |
| `month_added` | text | **New.** Month name from `date_added` |
| `duration_value` | integer | **New.** Minutes (movies) or number of seasons (TV shows) |
| `duration_unit` | text | **New.** `min` or `Seasons` |

**Changed from the original**
- The original `duration` column (e.g. "90 min", "2 Seasons") is replaced by `duration_value` + `duration_unit`. "Season" and "Seasons" are unified as `Seasons`.
- 3 rows had the runtime in `rating` and a blank `duration`; these were moved back.
- Rating `UR` merged into `NR` (both mean unrated).
- 7 `country` values with stray leading/trailing/double commas were fixed.
- Blank `director`, `cast`, `country` and `rating` are filled with `Unknown`. Blank `date_added` (10 rows) is left empty.

**Notes for analysis**
- `director`, `cast`, `country` and `listed_in` hold multiple values per cell. Split on `, ` before counting per person, country or genre.
- `Unknown` is a placeholder, not a real value. Exclude it from rankings (e.g. top countries).
- Filter on `duration_unit` before averaging `duration_value`, since minutes and seasons are not comparable.

---

## 2. netflix_customer_churn_clean.csv

One row per customer. `customer_id` is unique.

| Column | Type | Description |
|---|---|---|
| `customer_id` | text | Unique customer UUID |
| `age` | integer | Customer age (18-70 range in this data) |
| `gender` | text | `Male`, `Female`, `Other` |
| `subscription_type` | text | `Basic`, `Standard`, `Premium` |
| `watch_hours` | float | Total hours watched |
| `last_login_days` | integer | Days since last login (0-60) |
| `region` | text | Africa, Asia, Europe, North America, Oceania, South America |
| `device` | text | Primary device: Mobile, Tablet, Laptop, Desktop, TV |
| `monthly_fee` | float | Monthly fee: Basic 8.99, Standard 13.99, Premium 17.99 |
| `churned` | integer | `1` = churned, `0` = active |
| `payment_method` | text | Credit Card, Debit Card, PayPal, Crypto, Gift Card |
| `number_of_profiles` | integer | Profiles on the account (1-5) |
| `avg_watch_time_per_day` | float | Derived: `watch_hours / (last_login_days + 1)`, rounded to 2 decimals |
| `favorite_genre` | text | Action, Comedy, Documentary, Drama, Horror, Romance, Sci-Fi |
| `avg_watch_time_flag` | text | **New.** `ok` or `impossible (>24 h/day)` |
| `churned_label` | text | **New.** `Active` or `Churned`, readable version of `churned` |

**Changed from the original**
- Added `avg_watch_time_flag` and `churned_label`. No values were edited.
- The original file had no nulls, duplicates, duplicate IDs, fee-vs-plan mismatches or category typos.

**Notes for analysis**
- **10 rows are flagged** because `avg_watch_time_per_day` exceeds 24 hours, which is impossible. This happens when `last_login_days` is 0 or 1, which shrinks the divisor. Exclude these rows (`avg_watch_time_flag == "ok"`) when analysing daily watch time.
- `avg_watch_time_per_day` is a derived field. It divides total hours by days since last login, not by account age, so read it as a rough engagement signal.
- Churn is roughly balanced (about 50% churned), so accuracy is not misleading, unlike a typical churn dataset.
- Categories are near-uniformly distributed, which suggests the data may be synthetic. Be cautious about drawing real-world conclusions.

---

## 3. cleaning_log.csv

| Column | Description |
|---|---|
| `file` | `titles` or `churn` |
| `issue` | What was wrong |
| `rows_affected` | Number of rows touched |
| `action` | What was done |

## Reproducing the cleaning

The cleaning was done in Python (pandas). Key rules: parse `date_added` with format `%B %d, %Y`; split `duration` with regex `^(\d+)\s+(\w+)$`; fill text blanks with `Unknown`; flag rather than delete outliers.
