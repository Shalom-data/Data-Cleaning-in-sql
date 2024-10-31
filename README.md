# Nashville Housing Data Cleaning Project
This project involves transforming and cleaning a Nashville housing dataset to prepare it for analysis, focusing on improving data quality and organization. Each step of the process is documented to show the applied SQL transformations and methods.

## Objective
- To clean and standardize the Nashville housing dataset for in-depth analysis.
- The project includes handling missing values, parsing addresses, restructuring columns, and transforming data for clarity and usability.

## Tasks
- **Standardize Date Format:** Convert SaleDate to a standard date format and add a new New_date column.
- **Populate Missing Values:** Fill in missing values for PropertyAddress using matching ParcelID entries.
- **Split Columns:**
   - Split PropertyAddress into Property_Address, Property_City, and Property_State.
   - Split OwnerAddress into Owner_Address, Owner_City, and Owner_State.
- **Transform SoldAsVacant Values:** Replace 'Y' and 'N' with 'Yes' and 'No'.
- **Remove Duplicate Records:** Identify and delete duplicate rows.
- **Extract Day, Month, and Year:** Break down the date into day, month, and year for further analysis.
- Standardize Text Formats
- Add Price per Square Foot Metric
- **Drop Unused Columns:** Remove unnecessary columns as per stakeholder instructions.

## Data Cleaning Process
1. #### Standardize Date Format
- Convert the SaleDate column to a uniform date format `YYYY-MM-DD` and store it in a new column, `New_date`, which can be easily queried and analyzed.

```sql
ALTER TABLE Housing
ADD New_date DATE;

UPDATE Housing
SET New_date = CONVERT(date, SaleDate);
```


2. #### Populate Missing Values in PropertyAddress
- Perform a self-join on the ParcelID field to find matching records. For records missing a PropertyAddress, populate them with the address from other rows that share the same ParcelID.

```sql
UPDATE a
SET a.PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM Housing a
JOIN Housing b
    ON a.ParcelID = b.ParcelID
    AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;
```


3. #### Split PropertyAddress into Property_Address, Property_City, and Property_State
- Use string functions to break down PropertyAddress into individual components: Property_Address, Property_City, and Property_State, for more detailed analysis.

```sql
ALTER TABLE Housing
ADD Property_Address NVARCHAR(255),
    Property_City NVARCHAR(255);

UPDATE Housing
SET Property_Address = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1);

UPDATE Housing
SET Property_City = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));
```


4. #### Split OwnerAddress into Owner_Address, Owner_City, and Owner_State
- Split OwnerAddress into Owner_Address, Owner_City, and Owner_State using PARSENAME. First, replace commas with periods so PARSENAME can parse each segment. This creates separate fields for analysis.

```sql
ALTER TABLE Housing
ADD Owner_Address NVARCHAR(255),
    Owner_City NVARCHAR(255),
    Owner_State NVARCHAR(255);

UPDATE Housing
SET Owner_Address = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 3),
    Owner_City = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 2),
    Owner_State = PARSENAME(REPLACE(OwnerAddress, ',', '.'), 1);
```


5. #### Transform SoldAsVacant Column
- To make the SoldAsVacant column more readable, replace values 'Y' and 'N' with 'Yes' and 'No'.

```sql
UPDATE Housing
SET SoldAsVacant = CASE 
    WHEN SoldAsVacant = 'Y' THEN 'YES'
    WHEN SoldAsVacant = 'N' THEN 'NO'
    ELSE SoldAsVacant 
END;
```


6. #### Remove Duplicate Records
- Use a Common Table Expression (CTE) and ROW_NUMBER to identify and remove duplicate records. The CTE partitions data by ParcelID, PropertyAddress, SalePrice, SaleDate, and LegalReference while preserving the first occurrence and removing subsequent duplicates.

```sql
WITH CTE AS (
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY [UniqueID]) AS ROW_NUM
    FROM Housing
)
DELETE FROM CTE
WHERE ROW_NUM > 1;
```


7. #### Split New_date into Day, Month, and Year
- Break down the New_date column into Sale_Day, Sale_Month, and Sale_Year to allow for further time-based analysis.

```sql
ALTER TABLE Housing
ADD Sale_Day INT,
    Sale_Month INT,
    Sale_Year INT;

UPDATE Housing
SET Sale_Day = DAY(New_date),
    Sale_Month = MONTH(New_date),
    Sale_Year = YEAR(New_date);
```


9. #### Standardize Text Formats
- Ensure consistency in casing by converting address columns to title case.

```sql
UPDATE Housing
SET Property_Address = INITCAP(Property_Address),
    Property_State = INITCAP(Property_State),
    Property_City = INITCAP(Property_City),
    Owner_Address = INITCAP(Owner_Address),
    Owner_State = INITCAP(Owner_State),
    Owner_City = INITCAP(Owner_City);
```


10. #### Add Price per Square Foot Metric
- Enhance the housing dataset by introducing a new calculated column, Price_per_Acre, which allows for property size-adjusted price comparisons.

```sql
ALTER TABLE Housing
ADD Price_per_Acre DECIMAL(10, 2);

UPDATE Housing
SET Price_per_Acre = SalePrice / NULLIF(Acreage, 0);
```

11. #### Drop Unused Columns
- Remove columns that are no longer needed for analysis after cleaning, as advised by stakeholders. This reduces data complexity and file size.

```sql
ALTER TABLE Housing
DROP COLUMN PropertyAddress, LegalReference, OwnerAddress, TaxDistrict, SaleDate;
```
