## Report: `00_setup_check.ipynb` (Setup & Data Check)

### Purpose

The notebook prepares the raw Amazon dataset for downstream analysis by validating the load, running sanity checks, standardizing schema, cleaning key numeric fields, and reshaping multi-valued fields into a tidy (row-level) format. It then writes the cleaned result to a processed CSV.

---

## 1) Data load

**Action**

* Loads `../data/raw/amazon.csv` into a pandas DataFrame.

**Reasoning**

* Establishes a reproducible starting point (raw, unmodified source) and enables automated checks/cleaning steps.

---

## 2) Initial sanity checks / profiling

**Action**

* Displays `df.head()`.
* Runs a custom `summarize_df()` to print:

  * shape (rows/columns),
  * column list,
  * dtypes,
  * missing values (top 10),
  * duplicate row count,
  * plus `describe(include="all")` (preview) and `df.info()`.

**Reasoning**

* Confirms the dataset loaded correctly and highlights issues early (unexpected dtypes, missingness hotspots, duplicates) before cleaning and reshaping.

---

## 3) Column name normalization

**Action**

* Standardizes column names by:

  * trimming whitespace,
  * converting to lowercase,
  * replacing spaces with underscores.

**Reasoning**

* Prevents downstream bugs and inconsistent referencing (e.g., `"Discounted Price"` vs `"discounted_price"`), and enables cleaner, more reliable code.

---

## 4) Numeric and currency cleaning

**Action**

* Cleans and converts these columns to numeric using a helper:

  * `discounted_price`
  * `actual_price`
  * `discount_percentage`

The helper strips non-numeric characters (currency symbols, `%`, text) and converts using `pd.to_numeric(..., errors="coerce")`.

**Reasoning**

* Prices and percentages often arrive as formatted strings (currency signs, commas, percent symbols). Converting to numeric is required for aggregations, comparisons, and modeling. Coercion to `NaN` ensures invalid values don’t silently corrupt calculations.

---

## 5) Fixing `rating` and `rating_count` types

**Actions**

* Investigates unique values in `rating` and `rating_count`.
* Detects malformed `rating` entries containing the `|` character.
* Applies fixes:

  * `rating`: replaces the literal value `'|'` with `NaN`, then converts to numeric.
  * `rating_count`: string-cleans:

    * replaces `"nan"` with `"0"`,
    * removes thousands separators (commas),
    * converts to numeric with integer downcasting.

**Reasoning**

* Ensures both fields are usable as numeric measures:

  * `rating` must be numeric for averages/distributions.
  * `rating_count` must be numeric for weighting, popularity ranking, and validity checks.
* Explicitly handling malformed tokens (like `'|'`) avoids hard failures or misleading parsing.

---

## 6) Separating multi-valued categories (tidy reshape)

**Action**

* Splits `category` on `"|"` into a list, then `explode()`s it so **each row contains exactly one category**.

**Reasoning**

* Multi-valued fields break standard groupby/EDA logic. Exploding makes category-level analysis straightforward (counts, mean rating by category, etc.).
* Note: this increases the number of rows when products have multiple categories.

---

## 7) Normalizing review/user fields into row-level records

**Action**
For these columns:

* `user_id`, `user_name`, `review_id`, `review_title`, `review_content`

The notebook:

1. Splits each string by commas into lists.
2. Trims whitespace inside list items.
3. Checks that list lengths match across the five fields (alignment check).
4. Zips aligned lists into tuples per row.
5. Explodes into multiple rows (one row per review/user tuple).
6. Expands tuple elements back into the five columns.

**Reasoning**

* These fields represent *multiple reviews packed into a single row* (a nested/denormalized structure).
* The zip + explode pipeline converts them into a tidy structure where each row corresponds to a single coherent review entity.
* The length-alignment check is critical: it guards against mis-pairing (e.g., review titles shifting onto the wrong review IDs).

---

## 8) Output

**Action**

* Saves the processed dataset to: `../data/processed/amazon.csv` (no index).

**Reasoning**

* Creates a stable, reusable “processed layer” so later notebooks (e.g., EDA) don’t re-run heavy cleaning logic and always operate on the same standardized data.

---

# Final data state (after all transformations)

### Storage

* **File written:** `../data/processed/amazon.csv`

### Schema standardization

* Column names are normalized: `lowercase_with_underscores`.

### Key type guarantees (intended)

* `discounted_price`: numeric (float/nullable, depending on coercions)
* `actual_price`: numeric (float/nullable)
* `discount_percentage`: numeric (float/nullable)
* `rating`: numeric (float/nullable; malformed `'|'` becomes `NaN`)
* `rating_count`: numeric integer (downcasted where possible; missing treated as 0 via string replacement)

### Structural (row-level) guarantees

* `category` is **single-valued per row** (exploded from `|` lists).
* Review/user information is **single-valued per row** (exploded from comma-separated lists).
* As a result, **row count increases** relative to the raw file when:

  * a record has multiple categories and/or multiple reviews.

### Data quality notes introduced by design

* Any non-parsable numeric content becomes `NaN` (safe failure mode).
* `rating_count` missing values are coerced to **0** (this is a modeling/analysis choice; it treats “unknown” as “none”, which may or may not be desirable depending on downstream interpretation).

---
