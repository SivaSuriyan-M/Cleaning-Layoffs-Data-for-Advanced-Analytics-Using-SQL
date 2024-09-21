# SQL Data Cleaning Project

## Overview

This project focuses on cleaning and preparing the `world_layoffs.layoffs` dataset for analysis. The dataset contains information on company layoffs, and the data cleaning process ensures it is free from duplicates, standardized, and ready for further exploratory data analysis (EDA).

The dataset used can be found [here on Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022).

### Key Cleaning Steps:

1. Removing duplicates
2. Standardizing data
3. Handling null values
4. Removing unnecessary columns and rows

---

## Table of Contents

- [Staging Table Creation](#staging-table-creation)
- [Remove Duplicates](#remove-duplicates)
- [Standardize Data](#standardize-data)
  - [Handle Null or Empty Industry Values](#handle-null-or-empty-industry-values)
  - [Standardize Industry Naming](#standardize-industry-naming)
  - [Standardize Country Names](#standardize-country-names)
  - [Standardize Date Format](#standardize-date-format)
- [Handle Null Values](#handle-null-values)
- [Remove Unnecessary Rows](#remove-unnecessary-rows)
- [Remove Unnecessary Columns](#remove-unnecessary-columns)
- [Final Dataset](#final-dataset)

---

## Staging Table Creation

To ensure the raw data is preserved, a staging table is created where the data cleaning process is performed.

```sql
CREATE TABLE world_layoffs.layoffs_staging 
LIKE world_layoffs.layoffs;

INSERT INTO world_layoffs.layoffs_staging 
SELECT * FROM world_layoffs.layoffs;
```

---

## Remove Duplicates

First, duplicates are identified by assigning a `ROW_NUMBER()` to rows partitioned by key columns (e.g., `company`, `location`, `industry`, etc.).

```sql
SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
    ROW_NUMBER() OVER (
        PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
    ) AS row_num
FROM world_layoffs.layoffs_staging;
```

After identifying duplicates, rows where `row_num > 1` are removed:

```sql
DELETE FROM world_layoffs.layoffs_staging
WHERE row_num > 1;
```

---

## Standardize Data

### Handle Null or Empty Industry Values

To handle null or empty values in the `industry` column, a few steps are taken:

1. Empty strings in the `industry` column are set to `NULL`.

    ```sql
    UPDATE world_layoffs.layoffs_staging2
    SET industry = NULL
    WHERE industry = '';
    ```

2. If multiple rows exist for the same company, the null `industry` values are populated based on non-null values from other rows.

    ```sql
    UPDATE layoffs_staging2 t1
    JOIN layoffs_staging2 t2
    ON t1.company = t2.company
    SET t1.industry = t2.industry
    WHERE t1.industry IS NULL
    AND t2.industry IS NOT NULL;
    ```

### Standardize Industry Naming

Inconsistent naming for certain industries is corrected. For example, "Crypto Currency" and "CryptoCurrency" are unified as "Crypto."

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
```

### Standardize Country Names

Country names are cleaned by removing extra punctuation, such as a trailing period in "United States."

```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);
```

### Standardize Date Format

Dates in the dataset are cleaned and standardized by converting the `date` field from string to the `DATE` format.

```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

---

## Handle Null Values

The null values in the `total_laid_off`, `percentage_laid_off`, and `funds_raised_millions` columns are left as they are, as these nulls can be useful for analysis and calculations during the EDA phase.

```sql
SELECT * FROM world_layoffs.layoffs_staging2 WHERE total_laid_off IS NULL;
```

---

## Remove Unnecessary Rows

Rows where both `total_laid_off` and `percentage_laid_off` are `NULL` are considered uninformative and are removed.

```sql
DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

## Remove Unnecessary Columns

The `row_num` column, used for duplicate detection, is no longer needed and is dropped from the staging table.

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## Final Dataset

The cleaned dataset is now stored in `world_layoffs.layoffs_staging2` and is ready for analysis.

```sql
SELECT * 
FROM world_layoffs.layoffs_staging2;
```

---

## Summary

- **Staging Table:** Created for working with the raw data without altering the original dataset.
- **Duplicate Removal:** Duplicates are identified and removed using `ROW_NUMBER()`.
- **Data Standardization:** Industry names, country names, and date formats are standardized.
- **Handling Null Values:** Null values are preserved for specific columns, while rows with both `total_laid_off` and `percentage_laid_off` as null are removed.
- **Clean Table:** The cleaned table `layoffs_staging2` is finalized and ready for analysis.

---

### License

This project is open-sourced and available for public use. Feel free to contribute!

---

### Dataset Source

The dataset used in this project can be found on [Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022).
