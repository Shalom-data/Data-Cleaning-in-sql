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
- Remove Outliers in SalePrice
- Standardize Text Formats
- Add Price per Square Foot Metric
- **Drop Unused Columns:** Remove unnecessary columns as per stakeholder instructions.
