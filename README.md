# Power Plants Data Pipeline

A self-contained Jupyter notebook implementing a data ingestion and aggregation pipeline for power plant volume data across France and Great Britain.

---

## Overview

The pipeline ingests three source CSV files — `gas_fr_plants.csv`, `gas_plants.csv`, and `wind_plants.csv` — cleans and validates each one, and loads the results into a local `database.csv` file. From there, the data can be queried and aggregated in two ways: quarterly statistics per plant, and total volume by country and technology type.

All logic lives in a single `PowerPlants` class with six public methods, keeping the notebook entirely self-contained as required.

---

## Project Structure

```
power_plants_task.ipynb   # Main notebook — run top to bottom
gas_fr_plants.csv         # Source data: French gas plants
gas_plants.csv            # Source data: GB gas plants
wind_plants.csv           # Source data: GB wind plants
database.csv              # Output: generated on first run
README.md                 # This file
```

---

## How to Run

**Requirements:** Python 3.x with `pandas` installed.

```bash
pip install pandas
```

Open `power_plants_task.ipynb` in Jupyter and run all cells from top to bottom. The notebook is structured in four sections:

1. **Explore** — diagnostic summary of each source file (shape, nulls, duplicates, sample rows)
2. **Cleaning decisions** — documented rationale for every data quality choice made
3. **Implementation** — the `PowerPlants` class
4. **Results** — pipeline execution and output of all three aggregation methods

---

## Data Cleaning Approach

The source files had several quality issues identified during exploration. The decisions taken were:

| Issue | Action |
|---|---|
| Missing `Volume` | Fill with `0` (per spec) |
| Non-numeric `Volume` | Coerce to NaN, then fill with `0` |
| Missing or invalid `date` | Coerce to NaT, drop the row (no key = unusable) |
| Dates in `dd/mm/yyyy` format | Parse with `dayfirst=True` to avoid US-format misreads |
| Duplicate `(date, SiteName)` | Keep the row with the latest `updatetime` |
| Country as 2-letter code | Map to full name (`FR` → `France`, `GB` → `Great Britain`) |
| Missing `updatedby` | Fill with `'petroineos'` |
| Missing `updatetime` | Fill with current timestamp |
| `Technology` column absent | Inferred from filename (`wind_*.csv` → `Wind`, else `Gas`) |
| Column name casing inconsistency | Normalised to lowercase, then renamed to schema convention |

The database uses an **upsert pattern** keyed on `(date, SiteName)`: re-running the pipeline updates existing rows rather than duplicating them, making the process idempotent.

---

## PowerPlants Class — Method Summary

| Method | Description |
|---|---|
| `analyse_plant_data(file_path)` | Loads and cleans a source CSV, returning a validated DataFrame |
| `load_new_data_from_file(input_data)` | Validates schema and normalises dates ready for saving |
| `save_new_data(input_data)` | Upserts data into `database.csv` keyed on `(date, SiteName)` |
| `get_data_from_database()` | Returns the full daily history, deduplicated to one row per `(date, SiteName)` |
| `aggregate_data_to_quarterly()` | Wide-format quarterly Mean, Median and Std per plant |
| `aggregate_data_to_country()` | Total Volume grouped by country and technology type |

---

## Use of AI Assistance

**Claude Sonnet 4.6** was used for **code review and debugging** 

Specifically, Claude was used to:

- Review the `analyse_plant_data` method for edge cases after the initial implementation was written
- Identify that the `dd/mm/yyyy` date format required `dayfirst=True` when the pipeline was silently dropping hundreds of rows
- Cross-check the final notebook against the spec before submission to catch any remaining inconsistencies

The data exploration, cleaning strategy, class design, method logic, and all analytical decisions were my own. AI served as a second pair of eyes.
