# Pokémon Data Processing Pipeline (PySpark & Databricks)

## 📌 Dataset Description & Source
* **Source:** Kaggle Pokémon Dataset (`pokedex.csv`)
* **Description:** Contains statistical and biographical information for Pokémon, including IDs, elemental types, battle stats (HP, Attack, Defense, Speed), physical attributes (height, weight), and descriptions.

---

## 🛠️ Step-by-Step Pipeline Overview

1. **Data Ingestion:** Loaded CSV data using PySpark with permissive mode to catch malformed rows.
2. **Schema Enforcement:** Applied an explicit `StructType` schema to enforce column data types and track corrupt records.
3. **Data Quality Checks:** Verified data integrity using `_corrupt_record`.
4. **Transformations & Cleaning:**
   * Renamed key attributes (`id` $\rightarrow$ `pokemon_id`, `type` $\rightarrow$ `primary_type`).
   * Handled missing values by filling null `primary_type` entries with `"Unknown"` and dropping rows missing `pokemon_name`.
   * Removed duplicate entries (`dropDuplicates()`).
5. **Data Storage:** Exported cleaned and deduplicated dataset to optimized Parquet format.

---

## 📸 Key Output Screenshots

### 1. Initial Schema & Raw Data
![Schema](![alt text](sc-1.png))

### 2. Corrupt Record Check
![Corrupt Check](![alt text](sc-1-1.png))

### 3. Deduplication & Final Output
![Output](![alt text](sc-7.png))

---

## 💡 Technical Decisions & Justification

* **Read Mode (`PERMISSIVE`):** Chosen to ensure bad/corrupted records don't crash the ingestion job; instead, bad rows are routed to `_corrupt_record` for auditing.
* **Schema Definition:** Explicitly defining `StructType` improves read performance compared to schema inference and prevents type misalignment.
* **Null Handling Strategy:** 
  * `primary_type`: Replaced nulls with `"Unknown"` to preserve secondary stats without losing rows.
  * `pokemon_name`: Dropped rows with missing names, as a record without a identity key is non-actionable.
* **Parquet Format:** Selected for final storage due to its column-oriented structure, high compression efficiency, and optimal read performance for downstream queries.

---

## ⚠️ Challenges Faced & Resolutions

1. **Handling Malformed Data:**
   * *Challenge:* Risk of CSV parsing failures due to irregular row formatting.
   * *Resolution:* Implemented `columnNameOfCorruptRecord` alongside `PERMISSIVE` mode to safely capture non-conforming rows without failing the batch execution.
2. **Schema Validation:**
   * *Challenge:* Inferring schemas on large datasets adds unnecessary compute overhead.
   * *Resolution:* Enforced a custom `StructType` schema upfront to speed up read times.