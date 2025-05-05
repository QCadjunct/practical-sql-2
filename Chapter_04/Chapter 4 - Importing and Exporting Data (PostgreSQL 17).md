# Chapter 4: Importing and Exporting Data (PostgreSQL 17)

I apologize for the missing header links. Here's the corrected table of contents with functioning links:

# Chapter 4: Importing and Exporting Data (PostgreSQL 17)

## Table of Contents

- [Introduction](#introduction)
- [Working with Delimited Text Files](#working-with-delimited-text-files)
  - [Quoting Columns that Contain Delimiters](#quoting-columns-that-contain-delimiters)
  - [Handling Header Rows](#handling-header-rows)
- [Using COPY to Import Data](#using-copy-to-import-data)
  - [Importing Census Data Describing Counties](#importing-census-data-describing-counties)
  - [Creating the us_counties_2010 Table](#creating-the-us_counties_2010-table)
  - [Census Columns and Data Types](#census-columns-and-data-types)
  - [Performing the Census Import with COPY](#performing-the-census-import-with-copy)
  - [Importing a Subset of Columns with COPY](#importing-a-subset-of-columns-with-copy)
  - [Adding a Default Value to a Column During Import](#adding-a-default-value-to-a-column-during-import)
- [Using COPY to Export Data](#using-copy-to-export-data)
  - [Exporting All Data](#exporting-all-data)
  - [Exporting Particular Columns](#exporting-particular-columns)
  - [Exporting Query Results](#exporting-query-results)
- [Importing and Exporting Through pgAdmin](#importing-and-exporting-through-pgadmin)
- [Wrapping Up](#wrapping-up)
- [Try It Yourself](#try-it-yourself)

The rest of the document remains the same as previously provided. Each of these links should now properly navigate to the corresponding section in the document.

## Introduction

While SQL INSERT statements work well for adding a handful of rows to a table, they become impractical when loading hundreds, thousands, or millions of rows. PostgreSQL's COPY command provides an efficient solution for bulk importing and exporting data between delimited text files and database tables.

This chapter explores how to use COPY for both importing data into PostgreSQL and exporting it to external files. We'll work with real-world census data and learn various techniques for handling different import and export scenarios.

The typical import process follows three key steps:

1. Prepare source data as a delimited text file
2. Create a target table with appropriate structure
3. Write and execute a COPY script to perform the import

After completing an import, we'll verify the data and explore additional options for both importing and exporting.

[Back to Table of Contents](#table-of-contents)

## Working with Delimited Text Files

Delimited text files serve as a universal intermediate format for transferring data between different systems. Unlike proprietary formats, delimited text files can be easily shared across various platforms and applications.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Understanding the predicate logic of delimited text file structure as used in data exchange systems.

```
-- Predicate: Text files contain rows representing database entities
-- ∀r ∈ TextFile: ∃e ∈ Entity where Represents(r, e)

-- A typical row in a delimited text file (Domain-oriented example):
-- John,Doe,123MainSt,HydePark,NY,845-555-1212

-- Each column value separated by a delimiter implies:
-- ∀r ∈ TextFile: HasStructure(r, [attr₁, attr₂, ..., attrₙ])
-- where the delimiter (comma in this case) creates the structure
```

### PostgreSQL Standard Implementation

```
-- A typical row in a delimited text file (PostgreSQL standard example):
-- john,doe,123_main_st,hyde_park,ny,845-555-1212

-- Each value separated by commas represents a column in a database table
```

A delimited text file contains rows of data where:
- Each line represents one row in a database table
- A specific character (the delimiter) separates column values
- Commas are the most common delimiters (CSV - Comma-Separated Values)

### Quoting Columns that Contain Delimiters

When column data contains the delimiter character itself, it creates ambiguity that must be resolved.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Handling text data that contains the delimiter character using text qualifiers.

```
-- Predicate: Text qualifiers establish column boundaries when delimiters appear in data
-- ∀c ∈ Columns: (Contains(c, delimiter) → Enclosed(c, textQualifier))

-- Example with a column containing a delimiter (Domain-oriented example):
-- John,Doe,"123MainSt, Apartment200",HydePark,NY,845-555-1212

-- The predicate logic guarantees:
-- ∀c ∈ Columns: Enclosed(c, textQualifier) → TreatedAsOneValue(c)
```

### PostgreSQL Standard Implementation

```
-- Example with a column containing a delimiter:
-- john,doe,"123_main_st, apartment_200",hyde_park,ny,845-555-1212

-- Double quotes tell PostgreSQL to treat everything inside as one column value,
-- regardless of contained delimiters
```

To handle data values that contain the delimiter character:
- The value is enclosed in text qualifiers (typically double quotes)
- This signals to PostgreSQL to treat everything between the text qualifiers as a single column value
- PostgreSQL's default text qualifier is the double quote, but this can be changed if needed

### Handling Header Rows

Header rows identify the columns in a delimited file, making the data easier to understand.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Using header rows to describe the structure of data in a file.

```
-- Predicate: Header rows identify column attributes in the file
-- ∃!h ∈ TextFile: IsHeader(h) ∧ DefinesStructure(h, TextFile)

-- Example with header row (Domain-oriented example):
-- FirstName,LastName,Street,City,State,Phone
-- John,Doe,"123MainSt, Apartment200",HydePark,NY,845-555-1212

-- The header establishes a schema:
-- Schema(TextFile) = [FirstName, LastName, Street, City, State, Phone]
-- ∀r ∈ TextFile - {header}: ConformsTo(r, Schema(TextFile))
```

### PostgreSQL Standard Implementation

```
-- Example with header row:
-- first_name,last_name,street,city,state,phone
-- john,doe,"123_main_st, apartment_200",hyde_park,ny,845-555-1212
```

Header rows serve several important purposes:
1. They identify the data in each column, making the file contents more understandable
2. Some database systems use them to map columns to the target table (though PostgreSQL does not)
3. They provide documentation of the file structure

Since PostgreSQL doesn't use header rows for mapping, we need to use the HEADER option with COPY to exclude them during import.

## Using COPY to Import Data

The COPY command provides a powerful way to import data from external files into PostgreSQL tables.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Loading data from external files into a structured database system.

```sql
-- Predicate: COPY imports all rows from TextFile into Entity
-- ∀r ∈ TextFile: ∃e ∈ Entity where Imports(COPY, r, e)

COPY "Domain"."Entity"
FROM 'C:\YourDirectory\EntityData.csv'
WITH (FORMAT CSV, HEADER);

-- The COPY operation ensures:
-- 1. Schema(TextFile) ⊆ Schema(Entity)
-- 2. ∀a ∈ Schema(TextFile): ∃a' ∈ Schema(Entity) where Corresponds(a, a')
-- 3. ∀r ∈ TextFile - {header}: ∃e ∈ Entity where Corresponds(r, e)
```

### PostgreSQL Standard Implementation

```sql
COPY table_name
FROM 'C:\YourDirectory\your_file.csv'
WITH (FORMAT CSV, HEADER);
```

The COPY command has three essential components:
1. `COPY table_name` - Specifies which existing table will receive the data
2. `FROM 'path/to/file'` - Provides the complete file path and name
   - Windows paths begin with drive letter: `'C:\Users\Name\Desktop\file.csv'`
   - macOS/Linux paths begin at root: `'/Users/name/Desktop/file.csv'`
3. `WITH` options - Control how the import behaves:
   - `FORMAT` - Specifies file type (CSV, TEXT, or BINARY)
   - `HEADER` - Tells PostgreSQL to skip the header row
   - `DELIMITER` - Sets the character that separates columns
   - `QUOTE` - Specifies the text qualifier character

### Importing Census Data Describing Counties

In this section, we'll import U.S. Census data for counties from a CSV file containing 3,143 rows and 91 columns of demographic information.

### Creating the us_counties_2010 Table

Before importing data, we need to create a table with the appropriate structure.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Creating a structured repository for U.S. census demographic data at the county level, including geographic identifiers, population metrics, and demographic breakdowns.

```sql
-- Predicate: County is an Entity within the Census domain
-- Census(County) → County ∈ CensusDomain

CREATE TABLE "Census"."County2010" (
    "GeoName" VARCHAR(90),
    "StateAbbreviation" VARCHAR(2),
    "SummaryLevel" VARCHAR(3),
    "Region" SMALLINT,
    "Division" SMALLINT,
    "StateFips" VARCHAR(2),
    "CountyFips" VARCHAR(3),
    "AreaLand" BIGINT,
    "AreaWater" BIGINT,
    "Population" INTEGER,
    "HousingUnits" INTEGER,
    "InternalPointLat" NUMERIC(10,7),
    "InternalPointLon" NUMERIC(10,7),
    "P0010001" INTEGER,
    "P0010002" INTEGER,
    "P0010003" INTEGER,
    "P0010004" INTEGER,
    "P0010005" INTEGER,
    
    -- Standard metadata tracking fields
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Default value predicates
    -- ∀x ∈ County: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Census_County2010_IsDataMissing" 
        DEFAULT FALSE FOR "IsDataMissing",
    
    -- ∀x ∈ County: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Census_County2010_UserAuthorizationId" 
        DEFAULT 1 FOR "UserAuthorizationId",
    
    -- ∀x ∈ County: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Census_County2010_DateAdded" 
        DEFAULT NOW() FOR "DateAdded",
    
    -- ∀x ∈ County: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Census_County2010_DateOfLastUpdate" 
        DEFAULT NOW() FOR "DateOfLastUpdate"
);

-- Additional check constraints could be added for data validation
-- For example:
-- ∀x ∈ County: x.Population ≥ 0
-- CONSTRAINT "CHK_Census_County2010_PopulationNonNegative" 
--     CHECK ("Population" >= 0)
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE census.us_counties_2010 (
    geo_name varchar(90),
    state_us_abbreviation varchar(2),
    summary_level varchar(3),
    region smallint,
    division smallint,
    state_fips varchar(2),
    county_fips varchar(3),
    area_land bigint,
    area_water bigint,
    population_count_100_percent integer,
    housing_unit_count_100_percent integer,
    internal_point_lat numeric(10,7),
    internal_point_lon numeric(10,7),
    p0010001 integer,
    p0010002 integer,
    p0010003 integer,
    p0010004 integer,
    p0010005 integer
    -- Additional census columns omitted for brevity
);
```

### Census Columns and Data Types

The table structure uses specific data types chosen to match the characteristics of the census data fields.

#### Key Data Type Selections:

1. `VARCHAR(90)` for `geo_name`/`"GeoName"`: Counties have names up to 90 characters long, and using VARCHAR conserves space for shorter names.

2. `VARCHAR(2)` for `state_us_abbreviation`/`"StateAbbreviation"`: Standard two-letter state codes.

3. `VARCHAR(3)` for `summary_level`/`"SummaryLevel"`: Using VARCHAR preserves leading zeros in codes like "050" that would be lost if stored as numbers.

4. `SMALLINT` for `region`/`"Region"` and `division`/`"Division"`: Values range from 0-9 only, so SMALLINT is sufficient.

5. `VARCHAR(2)` and `VARCHAR(3)` for `state_fips`/`"StateFips"` and `county_fips`/`"CountyFips"`: These are codes, not numbers for calculations, and must preserve leading zeros.

6. `BIGINT` for `area_land`/`"AreaLand"` and `area_water`/`"AreaWater"`: Some Alaska areas exceed the INTEGER limit of 2,147,483,647.

7. `NUMERIC(10,7)` for latitude/longitude: Allows for precision to 7 decimal places with values up to 180 degrees.

### Performing the Census Import with COPY

Once the table is created, we can import the data using COPY.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Populating the county-level census database from standardized census data files.

```sql
-- Predicate: All rows from the CSV file are imported into the County entity
-- ∀r ∈ CensusCSV - {header}: ∃c ∈ County where Imports(COPY, r, c)

COPY "Census"."County2010"
FROM 'C:\YourDirectory\us_counties_2010.csv'
WITH (FORMAT CSV, HEADER);

-- This operation ensures:
-- ∀r ∈ CensusCSV - {header}: TransformsTo(r, County)
-- where the transformation maps CSV columns to database columns
```

### PostgreSQL Standard Implementation

```sql
COPY census.us_counties_2010
FROM 'C:\YourDirectory\us_counties_2010.csv'
WITH (FORMAT CSV, HEADER);
```

After running the COPY command, you should see a message indicating that 3,143 rows were imported.

To verify the import was successful, we can query the data:

```sql
-- Check counties with the largest land areas
SELECT geo_name, state_us_abbreviation, area_land
FROM census.us_counties_2010
ORDER BY area_land DESC
LIMIT 3;
```

Result:
```
geo_name                    state_us_abbreviation  area_land
---------------------------  --------------------  ------------
Yukon-Koyukuk Census Area   AK                    376855656455
North Slope Borough         AK                    229720054439
Bethel Census Area          AK                    105075822708
```

```sql
-- Check the easternmost counties by longitude
SELECT geo_name, state_us_abbreviation, internal_point_lon
FROM census.us_counties_2010
ORDER BY internal_point_lon DESC
LIMIT 5;
```

Result:
```
geo_name                    state_us_abbreviation  internal_point_lon
---------------------------  --------------------  -----------------
Aleutians West Census Area  AK                    178.3388130
Washington County           ME                    -67.6093542
Hancock County              ME                    -68.3707034
Aroostook County            ME                    -68.6494098
Penobscot County            ME                    -68.6574869
```

Interestingly, the Alaskan Aleutian Islands appear at the top of the longitude list because they extend so far west that they cross the antimeridian at 180 degrees, where longitude values become positive as they count back down toward 0.

### Importing a Subset of Columns with COPY

Sometimes your CSV file may not contain data for all columns in your target table. You can still import the available data by specifying which columns to populate.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Creating a repository for local government supervisor salary data with partial information available initially.

```sql
-- Predicate: Government entities have supervisors with compensation data
-- ∀s ∈ Supervisor: HasCompensation(s) ∧ WorksIn(s, Town)

CREATE TABLE "Government"."SupervisorSalary" (
    "Town" VARCHAR(30),
    "County" VARCHAR(30),
    "Supervisor" VARCHAR(30),
    "StartDate" DATE,
    "Salary" MONEY,
    "Benefits" MONEY,
    
    -- Standard metadata tracking fields
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Default constraints
    -- ∀x ∈ SupervisorSalary: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Government_SupervisorSalary_IsDataMissing" 
        DEFAULT FALSE FOR "IsDataMissing",
    
    -- ∀x ∈ SupervisorSalary: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Government_SupervisorSalary_UserAuthorizationId" 
        DEFAULT 1 FOR "UserAuthorizationId",
    
    -- ∀x ∈ SupervisorSalary: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Government_SupervisorSalary_DateAdded" 
        DEFAULT NOW() FOR "DateAdded",
    
    -- ∀x ∈ SupervisorSalary: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Government_SupervisorSalary_DateOfLastUpdate" 
        DEFAULT NOW() FOR "DateOfLastUpdate"
);

-- Import only the columns available in CSV (subset of full table)
-- Predicate: CSV contains only Town, Supervisor, and Salary columns
-- ∀r ∈ CSV: HasAttributes(r, [Town, Supervisor, Salary])
COPY "Government"."SupervisorSalary" ("Town", "Supervisor", "Salary")
FROM 'C:\YourDirectory\supervisor_salaries.csv'
WITH (FORMAT CSV, HEADER);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE government.supervisor_salaries (
    town varchar(30),
    county varchar(30),
    supervisor varchar(30),
    start_date date,
    salary money,
    benefits money
);

-- Import only the columns available in the CSV
COPY government.supervisor_salaries (town, supervisor, salary)
FROM 'C:\YourDirectory\supervisor_salaries.csv'
WITH (FORMAT CSV, HEADER);
```

By specifying which columns to import in parentheses after the table name, we tell PostgreSQL to only expect those columns in the CSV file. After running this command, a query would show data only in the specified columns:

```
town         county       supervisor    start_date    salary       benefits
-----------  -----------  ------------  ------------  -----------  -----------
Anytown                   Jones                       $27,000.00   
Bumblyburg                Baker                       $24,999.00   
```

### Adding a Default Value to a Column During Import

Using temporary tables provides a way to enhance data during the import process, such as adding default values for missing columns.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Enhancing imported supervisor data with standardized county information not present in the source file.

```sql
-- First, clear any existing data
DELETE FROM "Government"."SupervisorSalary";

-- Create temporary table with same structure
-- Predicate: TempEntity has same structure as TargetEntity
-- Structure(TempEntity) = Structure(TargetEntity)
CREATE TEMPORARY TABLE "Tmp_SupervisorSalary" (LIKE "Government"."SupervisorSalary");

-- Import available data to temporary table
COPY "Tmp_SupervisorSalary" ("Town", "Supervisor", "Salary")
FROM 'C:\YourDirectory\supervisor_salaries.csv'
WITH (FORMAT CSV, HEADER);

-- Insert from temp table with default county value
-- Predicate: All records will have a standardized county value
-- ∀s ∈ SupervisorSalary: s.County = 'Some County'
INSERT INTO "Government"."SupervisorSalary" ("Town", "County", "Supervisor", "Salary")
SELECT "Town", 'Some County', "Supervisor", "Salary"
FROM "Tmp_SupervisorSalary";

-- Clean up
DROP TABLE "Tmp_SupervisorSalary";
```

### PostgreSQL Standard Implementation

```sql
-- Clear existing data
DELETE FROM government.supervisor_salaries;

-- Create temporary table with same structure
CREATE TEMPORARY TABLE supervisor_salaries_temp (LIKE government.supervisor_salaries);

-- Import data to temp table
COPY supervisor_salaries_temp (town, supervisor, salary)
FROM 'C:\YourDirectory\supervisor_salaries.csv'
WITH (FORMAT CSV, HEADER);

-- Insert from temp table with added value
INSERT INTO government.supervisor_salaries (town, county, supervisor, salary)
SELECT town, 'Some County', supervisor, salary
FROM supervisor_salaries_temp;

-- Clean up
DROP TABLE supervisor_salaries_temp;
```

This process uses a multi-step approach:
1. Create a temporary table with the same structure as the target table
2. Import CSV data into the temporary table
3. Use INSERT/SELECT to copy data to the final table while adding default values
4. Remove the temporary table

The result shows data with the default county value added:

```
town         county       supervisor    start_date    salary       benefits
-----------  -----------  ------------  ------------  -----------  -----------
Anytown      Some County  Jones                       $27,000.00   
Bumblyburg   Some County  Baker                       $24,999.00   
```

## Using COPY to Export Data

The COPY command can also export data from PostgreSQL to external files.

### Exporting All Data

To export an entire table, use COPY with the TO keyword.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Extracting complete census data for external analysis or data sharing.

```sql
-- Predicate: All County entity records are exported to a text file
-- ∀c ∈ County: ∃r ∈ TextFile where Exports(COPY, c, r)

COPY "Census"."County2010"
TO 'C:\YourDirectory\county_export.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');

-- This ensures:
-- 1. Structure(TextFile) = Structure(County)
-- 2. ∀c ∈ County: ∃r ∈ TextFile where Corresponds(c, r)
-- 3. ∃!h ∈ TextFile: IsHeader(h) ∧ Contains(h, ColumnNames(County))
```

### PostgreSQL Standard Implementation

```sql
COPY census.us_counties_2010
TO 'C:\YourDirectory\us_counties_export.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

This exports all columns and rows from the us_counties_2010 table to a pipe-delimited text file. The WITH options specify:
- FORMAT CSV - Use CSV formatting rules
- HEADER - Include column names as the first row
- DELIMITER '|' - Use pipe characters instead of commas as separators

Using a .txt extension for a pipe-delimited file helps distinguish it from true comma-delimited CSV files.

### Exporting Particular Columns

You can export a subset of columns by listing them in parentheses after the table name.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Extracting only geographic location data from counties for mapping purposes.

```sql
-- Predicate: Only geographic identification and coordinates are exported
-- ∀c ∈ County: ∃r ∈ TextFile where Exports(COPY, [c.GeoName, c.InternalPointLat, c.InternalPointLon], r)

COPY "Census"."County2010" ("GeoName", "InternalPointLat", "InternalPointLon")
TO 'C:\YourDirectory\county_locations.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');

-- This ensures:
-- 1. Structure(TextFile) = [GeoName, InternalPointLat, InternalPointLon]
-- 2. ∀c ∈ County: ∃r ∈ TextFile where [r.col1, r.col2, r.col3] = [c.GeoName, c.InternalPointLat, c.InternalPointLon]
-- 3. TextFile contains only specified columns
```

### PostgreSQL Standard Implementation

```sql
COPY census.us_counties_2010 (geo_name, internal_point_lat, internal_point_lon)
TO 'C:\YourDirectory\us_counties_latlon_export.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

This exports only the specified columns from each row in the table. This approach is useful when you need to:
- Extract only specific data for a particular purpose (like mapping)
- Exclude sensitive information (like personal identifiers)
- Create smaller files with just the essential data

### Exporting Query Results

For even more control, you can export the results of a custom query.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Extracting only county records meeting specific naming criteria for targeted analysis.

```sql
-- Predicate: Only counties with "Mill" in their name are exported
-- ∀c ∈ County: Contains(c.GeoName, "Mill") → ∃r ∈ TextFile where Exports(COPY, [c.GeoName, c.StateAbbreviation], r)

COPY (
    SELECT "GeoName", "StateAbbreviation"
    FROM "Census"."County2010"
    WHERE "GeoName" ILIKE '%Mill%'
)
TO 'C:\YourDirectory\mill_counties.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');

-- This ensures:
-- 1. ∀r ∈ TextFile: ∃c ∈ County where 
--    Contains(c.GeoName, "Mill") ∧ 
--    r.col1 = c.GeoName ∧ 
--    r.col2 = c.StateAbbreviation
-- 2. Only records matching the query predicate are exported
```

### PostgreSQL Standard Implementation

```sql
COPY (
    SELECT geo_name, state_us_abbreviation
    FROM census.us_counties_2010
    WHERE geo_name ILIKE '%mill%'
)
TO 'C:\YourDirectory\us_counties_mill_export.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

By placing a query in parentheses after COPY, you can export just the results of that query. This example finds counties with "mill" in their name (case-insensitive) and exports only their names and state abbreviations. The result file will contain about nine rows with counties like Miller County, Roger Mills County, and Vermillion County.

## Importing and Exporting Through pgAdmin

When SQL COPY commands aren't feasible (such as when connecting to a remote PostgreSQL server), pgAdmin provides a graphical interface for imports and exports.

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Using a graphical interface to import/export data when direct file system access isn't available.

```
-- Predicate: pgAdmin provides an alternative path for import/export operations
-- ∀r ∈ CSV: ImportViaPgAdmin(r, Entity) = Import(r, Entity)
-- ∀e ∈ Entity: ExportViaPgAdmin(e, CSV) = Export(e, CSV)
```

### PostgreSQL Standard Implementation

pgAdmin provides an Import/Export dialog that can be accessed by:
1. Locating your table in the object browser
2. Right-clicking the table and selecting Import/Export
3. Configuring the import/export settings:
   - Set the Import/Export slider to the desired operation
   - Specify the file path using the file browser
   - Select the format (CSV, etc.)
   - Configure header, delimiter, and other options
   - Click OK to proceed

This method is especially useful when:
- You're connected to a remote PostgreSQL server
- You don't have direct access to the server's filesystem
- You prefer a graphical interface over command-line operations

## Wrapping Up

By mastering PostgreSQL's COPY command, you can efficiently move data between external files and database tables. This knowledge enables you to:

1. Import large datasets from various sources
2. Export selected data for sharing or analysis
3. Transform and enhance data during the import process
4. Filter data during export to meet specific needs

These skills are fundamental for working with real-world data, whether publicly available datasets or information specific to your work or research. Always look for data dictionaries to help you understand the structure and choose appropriate data types.

## Try It Yourself

### Exercise 1: Writing a WITH statement for special format

**Business Context:** Importing movie data with specialized formatting requirements.

### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Predicate: Importing movies with actors from a custom-formatted text file
-- ∀r ∈ TextFile: HasFormat(r, [id, movie, actor]) ∧ UsesDelimiter(r, ':') ∧ UsesQuote(r, '#')

COPY "Entertainment"."FilmActor"
FROM 'C:\YourDirectory\movie_data.txt'
WITH (
    FORMAT TEXT,
    DELIMITER ':',
    HEADER,
    QUOTE '#'
);

-- This ensures:
-- 1. DelimiterChar(TextFile) = ':'
-- 2. QuoteChar(TextFile) = '#'
-- 3. ∀v ∈ Values: Contains(v, ':') → Enclosed(v, '#')
```

### PostgreSQL Standard Implementation

```sql
COPY film_actors
FROM 'C:\YourDirectory\movie_data.txt'
WITH (
    FORMAT TEXT,
    DELIMITER ':',
    HEADER,
    QUOTE '#'
);
```

This COPY statement handles a text file with:
- Colon (:) as the delimiter between columns
- Pound/hash sign (#) as the text qualifier
- A header row that should be skipped
- Content like: `50:#Mission: Impossible#:Tom Cruise`

### Exercise 2: Exporting counties with most housing units

**Business Context:** Extracting data about counties with the highest housing density for real estate market analysis.

### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Predicate: Export the 20 counties with the most housing units
-- ∃S ⊂ County: |S| = 20 ∧ ∀c ∈ S, ∀d ∈ (County - S): c.HousingUnits ≥ d.HousingUnits

COPY (
    SELECT "GeoName", "StateAbbreviation", "HousingUnits"
    FROM "Census"."County2010"
    ORDER BY "HousingUnits" DESC
    LIMIT 20
)
TO 'C:\YourDirectory\counties_most_housing.csv'
WITH (FORMAT CSV, HEADER);
```

### PostgreSQL Standard Implementation

```sql
COPY (
    SELECT geo_name, state_us_abbreviation, housing_unit_count_100_percent
    FROM census.us_counties_2010
    ORDER BY housing_unit_count_100_percent DESC
    LIMIT 20
)
TO 'C:\YourDirectory\counties_most_housing.csv'
WITH (FORMAT CSV, HEADER);
```

This query:
1. Selects the county name, state, and housing unit count columns
2. Orders the results by housing unit count in descending order
3. Limits output to just the top 20 counties
4. Exports the results to a CSV file with headers

### Exercise 3: Evaluating numeric data type suitability

**Business Context:** Determining appropriate data type specifications for precise decimal values in financial or scientific applications.

### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Predicate: Numeric values must fit within the precision and scale of their data type
-- ∀v ∈ Values: Fits(v, numeric(p,s)) ↔ 
--   (|Digits(v)| ≤ p) ∧ (|DecimalDigits(v)| ≤ s) ∧ (|WholeDigits(v)| ≤ (p-s))

-- Sample values:
-- 17519.668
-- 20084.461
-- 18976.335

-- Analysis:
-- ∀v ∈ Values: |WholeDigits(v)| = 5 (five digits before decimal point)
-- ∀v ∈ Values: |DecimalDigits(v)| = 3 (three digits after decimal point)
-- ∀v ∈ Values: |Digits(v)| = 8 (total digits)
```

### PostgreSQL Standard Implementation

For the values:
```
17519.668
20084.461
18976.335
```

A column with data type `numeric(3,8)` would **not** work for these values because:

1. `numeric(3,8)` means:
   - 3 total digits allowed
   - 8 decimal places required
   
2. This creates an impossible specification:
   - With 8 decimal places required, you would need at least 8 total digits
   - With only 3 total digits allowed, you can have at most 3 decimal places
   - This effectively means `numeric(3,8)` allows for values with 8 decimal places and -5 digits to the left of the decimal, which is impossible

3. The actual values have:
   - 5 digits to the left of the decimal point (whole number part)
   - 3 digits to the right of the decimal point
   - 8 total digits

4. A suitable data type would be:
   - `numeric(8,3)` - Exactly matches the required digits
   - `numeric(9,3)` or `numeric(10,3)` - Provides room for growth

The fundamental constraint is that the precision (first parameter) must be greater than or equal to the scale (second parameter). Since our values require 5 whole number digits and 3 decimal digits, the minimum valid precision is 8.

## Implementation Analysis

The Domain Driven Database Design approach with predicate logic offers several advantages:

1. **Formalized Business Rules**: By expressing constraints as formal predicates, we can unambiguously specify the rules that govern our data.

2. **Self-Documenting Code**: The DDDD naming conventions and predicate logic expressions make database objects more understandable to both technical and business stakeholders.

3. **Data Quality Guarantees**: The standard metadata fields track data completeness and provenance, ensuring accountability and transparency.

4. **Business-Domain Alignment**: Organizing database objects by business domain improves conceptual clarity and system maintainability.

5. **More Rigorous Validation**: Explicit expression of business rules as constraints enforces data integrity within the database itself.

The PostgreSQL standard approach is more concise but sacrifices some of the explicitness and business domain alignment that makes databases self-documenting and easier to maintain in complex enterprise environments.

[Back to Table of Contents](#table-of-contents)
