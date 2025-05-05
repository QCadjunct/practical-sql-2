# Chapter 9 - Inspecting and Modifying Data

## Introduction to Dirty Data

Dirty data is a common challenge in data analysis - data with errors, missing values, or poor organization that hinders effective analysis. This chapter explores methods to assess data quality and techniques to modify data and tables to enhance analysis capabilities. These skills allow you to update or add new information, transforming a static database into a living record.

## Importing Data on Meat, Poultry, and Egg Producers

We'll use a directory of U.S. meat, poultry, and egg producers compiled by the Food Safety and Inspection Service (FSIS), an agency within the U.S. Department of Agriculture. This dataset contains information about meat processing plants, slaughterhouses, and farms.

### Creating the Initial Table Structure

#### PostgreSQL Standard Convention (snake_case):

```sql
CREATE TABLE food.meat_poultry_egg_inspect (
    est_number varchar(50) CONSTRAINT food_meat_poultry_egg_inspect_pky PRIMARY KEY,
    company varchar(100),
    street varchar(100),
    city varchar(30),
    st varchar(2),
    zip varchar(5),
    phone varchar(14),
    grant_date date,
    activities text,
    dbas text
);

COPY food.meat_poultry_egg_inspect
FROM '/path/to/MPI_Directory_by_Establishment_Name.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

CREATE INDEX food_meat_poultry_egg_inspect_company_idx 
ON food.meat_poultry_egg_inspect (company);
```

#### Domain Driven Design Convention (PascalCase):

```sql
CREATE TABLE "Food"."MeatPoultryEggInspection" (
    "EstablishmentNumber" varchar(50) CONSTRAINT "PK_Food_MeatPoultryEggInspection" PRIMARY KEY,
    "Company" varchar(100),
    "Street" varchar(100),
    "City" varchar(30),
    "State" varchar(2),
    "ZipCode" char(5),
    "PhoneNumber" varchar(14),
    "GrantDate" date,
    "Activities" text,
    "DoingBusinessAs" text
);

COPY "Food"."MeatPoultryEggInspection"
FROM '/path/to/MPI_Directory_by_Establishment_Name.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

CREATE INDEX "IX_Food_MeatPoultryEggInspection_Company" 
ON "Food"."MeatPoultryEggInspection" ("Company");
```

The code creates a table with 10 columns:

- A primary key constraint is applied to the establishment number column
- Text data type is used for activities and business names columns (allowing up to 1GB of characters)
- An index is created on the company column to speed up searches

Executing a count query reveals the dataset contains 6,287 rows:

```sql
SELECT count(*) FROM food.meat_poultry_egg_inspect;
-- Result: 6,287

-- DDD version
SELECT count(*) FROM "Food"."MeatPoultryEggInspection";
-- Result: 6,287
```

## Interviewing the Data Set

"Interviewing" data means discovering its details: what it contains, what questions it can answer, and how suitable it is for analysis purposes. Aggregate queries are useful tools for this process.

### Finding Multiple Companies at the Same Address

#### PostgreSQL Standard Convention (Example 1):

```sql
SELECT company,
       street,
       city,
       st,
       count(*) AS address_count 
FROM food.meat_poultry_egg_inspect
GROUP BY company, street, city, st 
HAVING count(*) > 1
ORDER BY company, street, city, st;
```

#### Domain Driven Design Convention:

```sql
SELECT "Company",
       "Street",
       "City",
       "State",
       count(*) AS "AddressCount" 
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "Company", "Street", "City", "State" 
HAVING count(*) > 1
ORDER BY "Company", "Street", "City", "State";
```

This query returns 23 rows where the same company is listed multiple times at the same address:

```markdown
Company                  | Street                 | City        | State | AddressCount
-------------------------+------------------------+-------------+-------+--------------
Acre Station Meat Farm   | 17076 Hwy 32 N         | Pinetown    | NC    | 2
Beltex Corporation       | 3801 North Grove Street| Fort Worth  | TX    | 2
Cloverleaf Cold Storage  | 111 Imperial Drive     | Sanford     | NC    | 2
...
```

This reveals potential data entry errors or valid reasons for duplication that require investigation before drawing conclusions.

### Checking for Missing Values

#### PostgreSQL Standard Convention - Checking Missing State Values

```sql
SELECT st,
       count(*) AS st_count
FROM food.meat_poultry_egg_inspect
GROUP BY st
ORDER BY st;
```

#### Domain Driven Design Convention - Checking Missing Values

```sql
SELECT "State",
       count(*) AS "StateCount"
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "State"
ORDER BY "State";
```

The result contains 57 rows (more than the 50 U.S. states) because the data includes Puerto Rico and other U.S. territories. Alaska appears at the top with 17 establishments.

```markdown
State | StateCount
------+-----------
AK    | 17
AL    | 93
AR    | 87
AS    | 1
...
WA    | 139
WI    | 184
WV    | 23
WY    | 3
NULL  | 3
```

The last row shows 3 records with NULL values in the state column.

#### Examining Rows with Missing State Values

#### PostgreSQL Standard Convention - Example 1

```sql
SELECT est_number,
       company,
       city,
       st,
       zip
FROM food.meat_poultry_egg_inspect
WHERE st IS NULL;
```

#### Domain Driven Design Convention - Example 1

```sql
SELECT "EstablishmentNumber",
       "Company",
       "City",
       "State",
       "ZipCode"
FROM "Food"."MeatPoultryEggInspection"
WHERE "State" IS NULL;
```

This query returns three rows that are missing state values:

```markdown
EstablishmentNumber   | Company                           | City    | State | ZipCode
----------------------+-----------------------------------+---------+-------+--------
V18677A               | Atlas Inspection, Inc.            | Blaine  | NULL  | 55449
M45319+P45319         | Hall-Namie Packing Company, Inc   | NULL    | NULL  | 36671
M263A+P263A+V263A     | Jones Dairy Farm                  | NULL    | NULL  | 53538
```

Checking the original data file confirms that these values were indeed missing in the source data.

### Checking for Inconsistent Data Values

#### PostgreSQL Standard Convention - Example 1

```sql
SELECT company,
       count(*) AS company_count
FROM food.meat_poultry_egg_inspect
GROUP BY company
ORDER BY company ASC;
```

#### Domain Driven Design Convention - Example 2

```sql
SELECT "Company",
       count(*) AS "CompanyCount"
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "Company"
ORDER BY "Company" ASC;
```

The results reveal spelling inconsistencies within company names:

```markdown
Company                      | CompanyCount
-----------------------------+--------------
...
Armour - Eckrich Meats, LLC  | 1
Armour-Eckrich Meats LLC     | 3
Armour-Eckrich Meats, Inc.   | 1
Armour-Eckrich Meats, LLC    | 2
...
```

These inconsistencies would affect any aggregation by company name.

### Checking for Malformed Values Using length()

#### PostgreSQL Standard Convention - Example:

```sql
SELECT length(COALESCE(zip, '')),
       count(*) AS length_count
FROM food.meat_poultry_egg_inspect
GROUP BY length(COALESCE(zip, ''))
ORDER BY length(COALESCE(zip, '')) ASC;
```

#### Domain Driven Design Convention - Example 2

```sql
SELECT length("ZipCode") AS "ZipCodeLength",
       count(*) AS "ZipCodeLengthCount"
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "ZipCodeLength"
ORDER BY "ZipCodeLength" ASC;
```

The results reveal formatting errors in ZIP codes:

```markdown
ZipCodeLength | ZipCodeLengthCount
--------------+-------------------
3             | 86
4             | 496
5             | 5705
```

This indicates that 582 ZIP codes have fewer than the expected 5 characters, likely because leading zeros were lost during file conversion.

Let's examine which states have shortened ZIP codes:

#### PostgreSQL Standard Convention - Example 1

```sql
SELECT st,
       count(*) AS st_count
FROM food.meat_poultry_egg_inspect
WHERE length(zip) < 5
GROUP BY st
ORDER BY st ASC;
```

#### Domain Driven Design Convention - Updating Table Structure

```sql
SELECT "State",
       count(*) AS "StateCount"
FROM "Food"."MeatPoultryEggInspection"
WHERE length("ZipCode") < 5
GROUP BY "State"
ORDER BY "State" ASC;
```

The results confirm this issue affects mainly Northeastern states where ZIP codes often start with zero:

```markdown
State | StateCount
------+-----------
CT    | 55
MA    | 101
ME    | 24
NH    | 18
NJ    | 244
PR    | 84
RI    | 27
VI    | 2
VT    | 27
```

## Modifying Tables, Columns, and Data

After identifying data issues, we'll use SQL commands to fix them:

- `ALTER TABLE` - Modify table structure
- `UPDATE` - Change values in columns

### Alter Table Syntax Examples

#### PostgreSQL Standard Convention

```sql
-- Add column
ALTER TABLE food.meat_poultry_egg_inspect ADD COLUMN new_column data_type;

-- Remove column
ALTER TABLE food.meat_poultry_egg_inspect DROP COLUMN column_name;

-- Change data type
ALTER TABLE food.meat_poultry_egg_inspect ALTER COLUMN column_name SET DATA TYPE data_type;

-- Add NOT NULL constraint
ALTER TABLE food.meat_poultry_egg_inspect ALTER COLUMN column_name SET NOT NULL;

-- Remove NOT NULL constraint
ALTER TABLE food.meat_poultry_egg_inspect ALTER COLUMN column_name DROP NOT NULL;
```

#### Domain Driven Design Convention

```sql
-- Add column
ALTER TABLE "Food"."MeatPoultryEggInspection" ADD COLUMN "NewColumn" data_type;

-- Remove column
ALTER TABLE "Food"."MeatPoultryEggInspection" DROP COLUMN "ColumnName";

-- Change data type
ALTER TABLE "Food"."MeatPoultryEggInspection" ALTER COLUMN "ColumnName" SET DATA TYPE data_type;

-- Add NOT NULL constraint
ALTER TABLE "Food"."MeatPoultryEggInspection" ALTER COLUMN "ColumnName" SET NOT NULL;

-- Remove NOT NULL constraint
ALTER TABLE "Food"."MeatPoultryEggInspection" ALTER COLUMN "ColumnName" DROP NOT NULL;
```

### Update Syntax Examples

#### PostgreSQL Standard Convention

```sql
-- Update all rows
UPDATE food.meat_poultry_egg_inspect
SET column_name = value;

-- Update multiple columns
UPDATE food.meat_poultry_egg_inspect
SET column_a = value_a,
    column_b = value_b;

-- Update with condition
UPDATE food.meat_poultry_egg_inspect
SET column_name = value
WHERE criteria;

-- Update with values from another table (ANSI standard with subquery)
UPDATE food.meat_poultry_egg_inspect
SET column_name = (SELECT column_name
                   FROM food.other_table
                   WHERE meat_poultry_egg_inspect.column = other_table.column)
WHERE EXISTS (SELECT column_name
              FROM food.other_table
              WHERE meat_poultry_egg_inspect.column = other_table.column);

-- Update with values from another table (PostgreSQL specific)
UPDATE food.meat_poultry_egg_inspect
SET column_name = other_table.column_name
FROM food.other_table
WHERE meat_poultry_egg_inspect.column = other_table.column;
```

#### Domain Driven Design Convention:

```sql
-- Update all rows
UPDATE "Food"."MeatPoultryEggInspection"
SET "ColumnName" = value;

-- Update multiple columns
UPDATE "Food"."MeatPoultryEggInspection"
SET "ColumnA" = value_a,
    "ColumnB" = value_b;

-- Update with condition
UPDATE "Food"."MeatPoultryEggInspection"
SET "ColumnName" = value
WHERE criteria;

-- Update with values from another table (ANSI standard with subquery)
UPDATE "Food"."MeatPoultryEggInspection"
SET "ColumnName" = (SELECT "ColumnName"
                   FROM "Food"."OtherTable"
                   WHERE "MeatPoultryEggInspection"."Column" = "OtherTable"."Column")
WHERE EXISTS (SELECT "ColumnName"
              FROM "Food"."OtherTable"
              WHERE "MeatPoultryEggInspection"."Column" = "OtherTable"."Column");

-- Update with values from another table (PostgreSQL specific)
UPDATE "Food"."MeatPoultryEggInspection"
SET "ColumnName" = "OtherTable"."ColumnName"
FROM "Food"."OtherTable"
WHERE "MeatPoultryEggInspection"."Column" = "OtherTable"."Column";
```

### Creating Backup Tables

Before modifying a table, it's prudent to create a backup:

#### PostgreSQL Standard Convention:

```sql
CREATE TABLE food.meat_poultry_egg_inspect_backup AS
SELECT * FROM food.meat_poultry_egg_inspect;
```

#### Domain Driven Design Convention:

```sql
CREATE TABLE "Food"."MeatPoultryEggInspection_Backup" AS
SELECT * FROM "Food"."MeatPoultryEggInspection";
```

Verify the backup was successful:

```sql
-- PostgreSQL Standard
SELECT
(SELECT count(*) FROM food.meat_poultry_egg_inspect) AS original, 
(SELECT count(*) FROM food.meat_poultry_egg_inspect_backup) AS backup;

-- Domain Driven Design
SELECT
(SELECT count(*) FROM "Food"."MeatPoultryEggInspection") AS "Original", 
(SELECT count(*) FROM "Food"."MeatPoultryEggInspection_Backup") AS "Backup";
```

Results:

```markdown
Original | Backup
---------+--------
6287     | 6287
```

Note: Indexes are not copied when creating table backups using this method.

### Restoring Missing Column Values

#### Creating a Column Copy

#### PostgreSQL Standard Convention:

```sql
-- Create column copy
ALTER TABLE food.meat_poultry_egg_inspect ADD COLUMN st_copy varchar(2);

-- Fill new column with values from original
UPDATE food.meat_poultry_egg_inspect
SET st_copy = st;
```

#### Domain Driven Design Convention:

```sql
-- Create column copy
ALTER TABLE "Food"."MeatPoultryEggInspection" ADD COLUMN "StateCopy" varchar(2);

-- Fill new column with values from original
UPDATE "Food"."MeatPoultryEggInspection"
SET "StateCopy" = "State";
```

Confirm values were correctly copied:

```sql
-- PostgreSQL Standard snake_case
SELECT st, st_copy
FROM food.meat_poultry_egg_inspect
ORDER BY st;

-- Domain Driven Design
SELECT "State", "StateCopy"
FROM "Food"."MeatPoultryEggInspection"
ORDER BY "State";
```

#### Updating Rows with Missing Values

#### PostgreSQL Standard Convention:

```sql
-- Update missing values one by one
UPDATE food.meat_poultry_egg_inspect
SET st = 'MN'
WHERE est_number = 'V18677A';

UPDATE food.meat_poultry_egg_inspect
SET st = 'AL'
WHERE est_number = 'M45319+P45319';

UPDATE food.meat_poultry_egg_inspect
SET st = 'WI'
WHERE est_number = 'M263A+P263A+V263A';
```

#### Domain Driven Design Convention:

```sql
-- Update missing values one by one
UPDATE "Food"."MeatPoultryEggInspection"
SET "State" = 'MN'
WHERE "EstablishmentNumber" = 'V18677A';

UPDATE "Food"."MeatPoultryEggInspection"
SET "State" = 'AL'
WHERE "EstablishmentNumber" = 'M45319+P45319';

UPDATE "Food"."MeatPoultryEggInspection"
SET "State" = 'WI'
WHERE "EstablishmentNumber" = 'M263A+P263A+V263A';
```

#### Restoring Original Values (If Needed)

#### PostgreSQL Standard Convention:

```sql
-- Restore from column copy
UPDATE food.meat_poultry_egg_inspect
SET st = st_copy;

-- Or restore from backup table
UPDATE food.meat_poultry_egg_inspect original
SET st = backup.st
FROM food.meat_poultry_egg_inspect_backup backup
WHERE original.est_number = backup.est_number;
```

#### Domain Driven Design Convention:

```sql
-- Restore from column copy
UPDATE "Food"."MeatPoultryEggInspection"
SET "State" = "StateCopy";

-- Or restore from backup table
UPDATE "Food"."MeatPoultryEggInspection" "Original"
SET "State" = "Backup"."State"
FROM "Food"."MeatPoultryEggInspection_Backup" "Backup"
WHERE "Original"."EstablishmentNumber" = "Backup"."EstablishmentNumber";
```

### Updating Values for Consistency

#### Creating a Standardized Column

#### PostgreSQL Standard Convention:

```sql
-- Create standardized column
ALTER TABLE food.meat_poultry_egg_inspect ADD COLUMN company_standard varchar(100);

-- Copy original values
UPDATE food.meat_poultry_egg_inspect
SET company_standard = company;
```

#### Domain Driven Design Convention:

```sql
-- Create standardized column
ALTER TABLE "Food"."MeatPoultryEggInspection" ADD COLUMN "CompanyStandardized" varchar(100);

-- Copy original values
UPDATE "Food"."MeatPoultryEggInspection"
SET "CompanyStandardized" = "Company";
```

#### Standardizing Company Names

#### PostgreSQL Standard Convention:

```sql
-- Standardize one company name
UPDATE food.meat_poultry_egg_inspect
SET company_standard = 'Armour-Eckrich Meats'
WHERE company LIKE 'Armour%';

-- Check results
SELECT company, company_standard
FROM food.meat_poultry_egg_inspect
WHERE company LIKE 'Armour%';
```

#### Domain Driven Design Convention:

```sql
-- Standardize one company name
UPDATE "Food"."MeatPoultryEggInspection"
SET "CompanyStandardized" = 'Armour-Eckrich Meats'
WHERE "Company" LIKE 'Armour%';

-- Check results
SELECT "Company", "CompanyStandardized"
FROM "Food"."MeatPoultryEggInspection"
WHERE "Company" LIKE 'Armour%';
```

Results:

```markdown
Company                      | CompanyStandardized
-----------------------------+---------------------
Armour-Eckrich Meats LLC     | Armour-Eckrich Meats
Armour - Eckrich Meats, LLC  | Armour-Eckrich Meats
Armour-Eckrich Meats LLC     | Armour-Eckrich Meats
Armour-Eckrich Meats LLC     | Armour-Eckrich Meats
Armour-Eckrich Meats, Inc.   | Armour-Eckrich Meats
Armour-Eckrich Meats, LLC    | Armour-Eckrich Meats
Armour-Eckrich Meats, LLC    | Armour-Eckrich Meats
```

### Repairing ZIP Codes Using Concatenation

#### Creating a ZIP Code Column Backup

#### PostgreSQL Standard Convention:

```sql
-- Create ZIP code backup column
ALTER TABLE food.meat_poultry_egg_inspect ADD COLUMN zip_copy varchar(5);

-- Copy values
UPDATE food.meat_poultry_egg_inspect
SET zip_copy = zip;
```

#### Domain Driven Design Convention:

```sql
-- Create ZIP code backup column
ALTER TABLE "Food"."MeatPoultryEggInspection" ADD COLUMN "ZipCodeCopy" char(5);

-- Copy values
UPDATE "Food"."MeatPoultryEggInspection"
SET "ZipCodeCopy" = "ZipCode";
```

#### Fixing ZIP Codes with Missing Leading Zeros

#### PostgreSQL Standard Convention:

```sql
-- Fix codes missing two leading zeros (PR and VI)
UPDATE food.meat_poultry_egg_inspect
SET zip = '00' || zip
WHERE st IN('PR','VI') AND length(zip) = 3;

-- Fix codes missing one leading zero (Northeast states)
UPDATE food.meat_poultry_egg_inspect
SET zip = '0' || zip
WHERE st IN('CT','MA','ME','NH','NJ','RI','VT') AND length(zip) = 4;
```

#### Domain Driven Design Convention:

```sql
-- Fix codes missing two leading zeros (PR and VI)
UPDATE "Food"."MeatPoultryEggInspection"
SET "ZipCode" = '00' || "ZipCode"
WHERE "State" IN('PR','VI') AND length("ZipCode") = 3;

-- Fix codes missing one leading zero (Northeast states)
UPDATE "Food"."MeatPoultryEggInspection"
SET "ZipCode" = '0' || "ZipCode"
WHERE "State" IN('CT','MA','ME','NH','NJ','RI','VT') AND length("ZipCode") = 4;
```

Verify the fixes:

```sql
-- PostgreSQL Standard
SELECT length(zip), count(*) AS length_count
FROM food.meat_poultry_egg_inspect
GROUP BY length(zip)
ORDER BY length(zip) ASC;

-- Domain Driven Design
SELECT length("ZipCode") AS "ZipCodeLength", count(*) AS "ZipCodeLengthCount"
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "ZipCodeLength"
ORDER BY "ZipCodeLength" ASC;
```

Results after fixes:

```markdown
ZipCodeLength | ZipCodeLengthCount
--------------+-------------------
5             | 6287
```

### Updating Values Across Tables

Let's add inspection dates to companies based on their geographic regions:

#### PostgreSQL Standard Convention:

```sql
-- Create regions table
CREATE TABLE food.state_regions (
    st varchar(2) CONSTRAINT food_state_regions_pky PRIMARY KEY, 
    region varchar(20) NOT NULL
);

-- Import data
COPY food.state_regions
FROM '/path/to/state_regions.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

-- Add inspection_date column
ALTER TABLE food.meat_poultry_egg_inspect ADD COLUMN inspection_date date;

-- Update inspection dates for New England region
UPDATE food.meat_poultry_egg_inspect inspect
SET inspection_date = '2019-12-01'
WHERE EXISTS (SELECT state_regions.region
              FROM food.state_regions
              WHERE inspect.st = state_regions.st
              AND state_regions.region = 'New England');
```

#### Domain Driven Design Convention:

```sql
-- Create regions table
CREATE TABLE "Food"."StateRegion" (
    "State" varchar(2) CONSTRAINT "PK_Food_StateRegion" PRIMARY KEY, 
    "Region" varchar(20) NOT NULL
);

-- Import data
COPY "Food"."StateRegion"
FROM '/path/to/state_regions.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

-- Add inspection_date column
ALTER TABLE "Food"."MeatPoultryEggInspection" ADD COLUMN "InspectionDate" date;

-- Update inspection dates for New England region
UPDATE "Food"."MeatPoultryEggInspection" "Inspection"
SET "InspectionDate" = '2019-12-01'
WHERE EXISTS (SELECT "StateRegion"."Region"
              FROM "Food"."StateRegion"
              WHERE "Inspection"."State" = "StateRegion"."State"
              AND "StateRegion"."Region" = 'New England');
```

Verify the updates:

```sql
-- PostgreSQL Standard
SELECT st, inspection_date
FROM food.meat_poultry_egg_inspect
GROUP BY st, inspection_date
ORDER BY st;

-- Domain Driven Design
SELECT "State", "InspectionDate"
FROM "Food"."MeatPoultryEggInspection"
GROUP BY "State", "InspectionDate"
ORDER BY "State";
```

Results:

```markdown
State | InspectionDate
------+---------------
...    |
CA    | NULL
CO    | NULL
CT    | 2019-12-01
DC    | NULL
...    |
```

## Deleting Unnecessary Data

### Deleting Rows from a Table

#### PostgreSQL Standard Convention:

```sql
-- Remove all rows from table
DELETE FROM food.meat_poultry_egg_inspect;

-- Remove only specified rows
DELETE FROM food.meat_poultry_egg_inspect
WHERE st IN('PR','VI');
```

#### Domain Driven Design Convention:

```sql
-- Remove all rows from table
DELETE FROM "Food"."MeatPoultryEggInspection";

-- Remove only specified rows
DELETE FROM "Food"."MeatPoultryEggInspection"
WHERE "State" IN('PR','VI');
```

### Deleting a Column from a Table

#### PostgreSQL Standard Convention:

```sql
ALTER TABLE food.meat_poultry_egg_inspect DROP COLUMN zip_copy;
```

#### Domain Driven Design Convention:

```sql
ALTER TABLE "Food"."MeatPoultryEggInspection" DROP COLUMN "ZipCodeCopy";
```

### Deleting a Table from a Database

#### PostgreSQL Standard Convention:

```sql
DROP TABLE food.meat_poultry_egg_inspect_backup;
```

#### Domain Driven Design Convention:

```sql
DROP TABLE "Food"."MeatPoultryEggInspection_Backup";
```

## Using Transaction Blocks to Save or Revert Changes

Transaction blocks allow you to test changes before committing them:

#### PostgreSQL Standard Convention:

```sql
-- Start transaction block
START TRANSACTION;

-- Make changes
UPDATE food.meat_poultry_egg_inspect
SET company = 'AGRO Merchantss Oakland LLC'  -- Note the typo
WHERE company = 'AGRO Merchants Oakland, LLC';

-- Check results
SELECT company
FROM food.meat_poultry_egg_inspect
WHERE company LIKE 'AGRO%'
ORDER BY company;

-- Discard changes due to typo
ROLLBACK;

-- Or save changes if they look good
-- COMMIT;
```

#### Domain Driven Design Convention:

```sql
-- Start transaction block
START TRANSACTION;

-- Make changes
UPDATE "Food"."MeatPoultryEggInspection"
SET "Company" = 'AGRO Merchantss Oakland LLC'  -- Note the typo
WHERE "Company" = 'AGRO Merchants Oakland, LLC';

-- Check results
SELECT "Company"
FROM "Food"."MeatPoultryEggInspection"
WHERE "Company" LIKE 'AGRO%'
ORDER BY "Company";

-- Discard changes due to typo
ROLLBACK;

-- Or save changes if they look good
-- COMMIT;
```

## Improving Performance When Updating Large Tables

For large tables, adding columns and updating values can be expensive. An alternative approach:

#### PostgreSQL Standard Convention:

```sql
-- Create a new table with the added column already populated
CREATE TABLE food.meat_poultry_egg_inspect_backup AS
SELECT *,
       '2018-02-07'::date AS reviewed_date
FROM food.meat_poultry_egg_inspect;

-- Swap table names
ALTER TABLE food.meat_poultry_egg_inspect RENAME TO meat_poultry_egg_inspect_temp;
ALTER TABLE food.meat_poultry_egg_inspect_backup
RENAME TO meat_poultry_egg_inspect;
ALTER TABLE food.meat_poultry_egg_inspect_temp
RENAME TO meat_poultry_egg_inspect_backup;
```

#### Domain Driven Design Convention:

```sql
-- Create a new table with the added column already populated
CREATE TABLE "Food"."MeatPoultryEggInspection_Backup" AS
SELECT *,
       '2018-02-07'::date AS "ReviewDate"
FROM "Food"."MeatPoultryEggInspection";

-- Swap table names
ALTER TABLE "Food"."MeatPoultryEggInspection" RENAME TO "MeatPoultryEggInspection_Temp";
ALTER TABLE "Food"."MeatPoultryEggInspection_Backup"
RENAME TO "MeatPoultryEggInspection";
ALTER TABLE "Food"."MeatPoultryEggInspection_Temp"
RENAME TO "MeatPoultryEggInspection_Backup";
```

This approach avoids updating individual rows and prevents the database from inflating the table size.

## Try It Yourself Exercise

### Challenge: Identifying Meat and Poultry Processing Plants

The goal is to determine how many plants process meat and how many process poultry, using the `activities` column which contains inconsistent text.

#### PostgreSQL Standard Convention:

```sql
-- Create columns for tracking activities
ALTER TABLE food.meat_poultry_egg_inspect 
ADD COLUMN meat_processing boolean,
ADD COLUMN poultry_processing boolean;

-- Set meat_processing flag
UPDATE food.meat_poultry_egg_inspect
SET meat_processing = TRUE
WHERE activities LIKE '%Meat Processing%';

-- Set poultry_processing flag
UPDATE food.meat_poultry_egg_inspect
SET poultry_processing = TRUE
WHERE activities LIKE '%Poultry Processing%';

-- Count meat processing plants
SELECT count(*) AS meat_processing_count
FROM food.meat_poultry_egg_inspect
WHERE meat_processing = TRUE;

-- Count poultry processing plants
SELECT count(*) AS poultry_processing_count
FROM food.meat_poultry_egg_inspect
WHERE poultry_processing = TRUE;

-- Count plants that do both
SELECT count(*) AS both_activities_count
FROM food.meat_poultry_egg_inspect
WHERE meat_processing = TRUE AND poultry_processing = TRUE;
```

#### Domain Driven Design Convention:

```sql
-- Create columns for tracking activities
ALTER TABLE "Food"."MeatPoultryEggInspection" 
ADD COLUMN "ProcessesMeat" boolean,
ADD COLUMN "ProcessesPoultry" boolean;

-- Set meat processing flag
UPDATE "Food"."MeatPoultryEggInspection"
SET "ProcessesMeat" = TRUE
WHERE "Activities" LIKE '%Meat Processing%';

-- Set poultry processing flag
UPDATE "Food"."MeatPoultryEggInspection"
SET "ProcessesPoultry" = TRUE
WHERE "Activities" LIKE '%Poultry Processing%';

```sql
-- Count meat processing plants
SELECT count(*) AS "MeatProcessingCount"
FROM "Food"."MeatPoultryEggInspection"
WHERE "ProcessesMeat" = TRUE;

-- Count poultry processing plants
SELECT count(*) AS "PoultryProcessingCount"
FROM "Food"."MeatPoultryEggInspection"
WHERE "ProcessesPoultry" = TRUE;

-- Count plants that do both
SELECT count(*) AS "DualProcessingCount"
FROM "Food"."MeatPoultryEggInspection"
WHERE "ProcessesMeat" = TRUE AND "ProcessesPoultry" = TRUE;
```

Sample results might look like:

```markdown
MeatProcessingCount
-------------------
4,218

PoultryProcessingCount
----------------------
3,417

DualProcessingCount
------------------
2,284
```

This analysis reveals how many facilities process meat, how many process poultry, and how many do both types of processing. The exercise demonstrates how creating boolean flag columns can transform inconsistent text data into structured information that's easier to query and analyze.

## Summary of Data Cleaning Techniques

Throughout this chapter, we've learned several techniques for identifying and fixing data quality issues:

1. **Data Assessment Techniques**:
   - Using aggregate functions with GROUP BY to identify patterns and anomalies
   - Using LENGTH() to detect malformed or incomplete values
   - Checking for NULL values with IS NULL
   - Finding inconsistent spellings with GROUP BY and COUNT()

2. **Data Modification Techniques**:
   - Creating backup tables before making changes
   - Creating backup columns within tables
   - Using UPDATE to fix missing or incorrect values
   - Using string concatenation (||) to repair values
   - Updating values based on conditions (WHERE)
   - Updating values from other tables (cross-table updates)

3. **Table Structure Modifications**:
   - Adding new columns with ALTER TABLE
   - Removing columns with DROP COLUMN
   - Removing entire tables with DROP TABLE
   - Renaming tables to swap content

4. **Safe Update Practices**:
   - Using transaction blocks (START TRANSACTION, COMMIT, ROLLBACK)
   - Testing updates before committing them
   - Performance optimization for large tables

This chapter emphasizes safety by consistently making backups before changes and using transaction blocks to verify modifications before committing them. These practices are essential when working with production databases.

## Best Practices for Data Cleaning

1. **Always Interview Your Data First**:
   - Examine distributions, counts, and patterns to understand your data
   - Look for anomalies and unexpected values before analysis

2. **Validate Before Trusting**:
   - Investigate suspicious data patterns by comparing with authoritative sources
   - Consider whether inconsistencies are data entry errors or valid variations

3. **Create Backups**:
   - Make table copies before structural changes
   - Create column copies before data modifications
   - Use transaction blocks for test-before-commit operations

4. **Document Changes**:
   - Keep records of all data transformations applied
   - Note the reasons for modifications
   - Maintain data lineage (provenance) information

5. **Standardize Incrementally**:
   - Focus on one issue at a time
   - Test each change before proceeding to the next
   - Verify that each change produces the expected result

6. **Know When to Abandon Data**:
   - Sometimes a dataset is too compromised to be worth cleaning
   - Evaluate the cost/benefit of cleanup vs. seeking alternative data

## When to Toss Your Data

The chapter notes important considerations for determining when data may be too problematic to use:

1. **Too Many Missing Values**: When critical information is absent from a significant portion of the dataset.

2. **Implausible Values**: When values defy common sense (e.g., numbers in billions where thousands are expected).

3. **Import Errors**: When column misalignments or formatting issues corrupt the data during import.

In these cases:

- First, revisit the original data file to ensure correct import
- Contact the agency or company that produced the data for clarification
- Seek advice from others who have used the same data
- Consider alternative data sources if cleanup is impractical

Making a judicious decision to abandon problematic data is sometimes better than basing analysis on unreliable information.

## Conclusion

Dirty data is inevitable in real-world analysis. The ability to identify and rectify data quality issues is an essential skill for data analysts and database administrators. This chapter has provided practical techniques for:

- Assessing data quality
- Creating safety nets before making changes
- Fixing common data problems
- Structuring data for effective analysis

By applying these skills, you can transform static collections of flawed data into living, accurate records that support reliable analysis. The next chapter will build on these foundations to explore advanced statistical functions and analytical techniques.
