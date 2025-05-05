# Chapter 8 - Extracting Information by Grouping and Summarizing

## Introduction

Chapter 8 explores how to extract meaningful insights from data through grouping and summarizing techniques in PostgreSQL. The chapter uses a real-world library dataset to demonstrate how to find trends and patterns using SQL aggregate functions and grouping operations. These techniques allow analysts to answer important questions that wouldn't be visible by looking at individual rows of data.

## Creating the Library Survey Tables

### Creating the 2014 Library Data Table

#### PostgreSQL Standard Approach (snake_case) - 2014 Library Data

```sql
CREATE TABLE libraries.pls_fy2014_pupldl4a (
    stabr varchar(2) NOT NULL,
    fscskey varchar(6) CONSTRAINT fscskey2014_key PRIMARY KEY, 
    libid varchar(20) NOT NULL,
    libname varchar(100) NOT NULL,
    obereg varchar(2) NOT NULL,
    rstatus integer NOT NULL,
    statstru varchar(2) NOT NULL,
    statname varchar(2) NOT NULL,
    stataddr varchar(2) NOT NULL,
    -- additional columns omitted for brevity
    wifisess integer NOT NULL,
    yr_sub integer NOT NULL
);

CREATE INDEX libraries_pls_fy2014_pupldl4a_libname_idx ON libraries.pls_fy2014_pupldl4a (libname);
CREATE INDEX libraries_pls_fy2014_pupldl4a_stabr_idx ON libraries.pls_fy2014_pupldl4a (stabr);
CREATE INDEX libraries_pls_fy2014_pupldl4a_city_idx ON libraries.pls_fy2014_pupldl4a (city);
CREATE INDEX libraries_pls_fy2014_pupldl4a_visits_idx ON libraries.pls_fy2014_pupldl4a (visits);

COPY libraries.pls_fy2014_pupldl4a
FROM 'C:\YourDirectory\pls_fy2014_pupldl4a.csv'
WITH (FORMAT CSV, HEADER);
```

#### Domain-Driven Design Approach (PascalCase) - 2014 Library Data

```sql
CREATE TABLE "Libraries"."PublicLibrary2014" (
    "StateAbbr" varchar(2) NOT NULL,
    "PublicLibrary2014Id" varchar(6) CONSTRAINT "PK_Libraries_PublicLibrary2014" PRIMARY KEY, 
    "LibraryId" varchar(20) NOT NULL,
    "LibraryName" varchar(100) NOT NULL,
    "BureauEconRegion" char(2) NOT NULL,
    "ResponseStatus" integer NOT NULL,
    "StateStru" char(2) NOT NULL,  -- What does this mean StateStru
    "StateName" char(2) NOT NULL,
    "StateAddress" char(2) NOT NULL,
    -- additional columns omitted for brevity
    "WifiSessions" integer NOT NULL,
    "YearSubmitted" integer NOT NULL
);

CREATE INDEX "IX_Libraries_PublicLibrary2014_LibraryName" ON "Libraries"."PublicLibrary2014" ("LibraryName");
CREATE INDEX "IX_Libraries_PublicLibrary2014_StateAbbr" ON "Libraries"."PublicLibrary2014" ("StateAbbr");
CREATE INDEX "IX_Libraries_PublicLibrary2014_City" ON "Libraries"."PublicLibrary2014" ("City");
CREATE INDEX "IX_Libraries_PublicLibrary2014_Visits" ON "Libraries"."PublicLibrary2014" ("Visits");

COPY "Libraries"."PublicLibrary2014"
FROM 'C:\YourDirectory\pls_fy2014_pupldl4a.csv'
WITH (FORMAT CSV, HEADER);
```

**Explanation:**

- The code creates a table structure for storing 2014 library survey data from the Public Libraries Survey
- Primary key constraint is defined on the `fscskey` column, representing a unique identifier for each library agency
- Indexes are created on frequently queried columns to improve search performance
- The `COPY` command imports data from a CSV file

### Creating the 2009 Library Data Table

#### PostgreSQL Standard Approach (snake_case) - Counting Rows (2014 Data)

```sql
CREATE TABLE libraries.pls_fy2009_pupld09a (
    stabr varchar(2) NOT NULL,
    fscskey varchar(6) CONSTRAINT fscskey2009_key PRIMARY KEY, 
    libid varchar(20) NOT NULL,
    libname varchar(100) NOT NULL,
    address varchar(35) NOT NULL,
    city varchar(20) NOT NULL,
    zip varchar(5) NOT NULL,
    zip4 varchar(4) NOT NULL,
    cnty varchar(20) NOT NULL,
    -- additional columns omitted for brevity
    fipsst varchar(2) NOT NULL,
    fipsco varchar(3) NOT NULL
);

CREATE INDEX libraries_pls_fy2009_pupld09a_libname_idx ON libraries.pls_fy2009_pupld09a (libname);
CREATE INDEX libraries_pls_fy2009_pupld09a_stabr_idx ON libraries.pls_fy2009_pupld09a (stabr);
CREATE INDEX libraries_pls_fy2009_pupld09a_city_idx ON libraries.pls_fy2009_pupld09a (city);
CREATE INDEX libraries_pls_fy2009_pupld09a_visits_idx ON libraries.pls_fy2009_pupld09a (visits);

COPY libraries.pls_fy2009_pupld09a
FROM 'C:\YourDirectory\pls_fy2009_pupld09a.csv'
WITH (FORMAT CSV, HEADER);
```

#### Domain-Driven Design Approach (PascalCase) - 2014 Library Data

```sql
CREATE TABLE "Libraries"."PublicLibrary2009" (
    "StateAbbreviation" char(2) NOT NULL,
    "PublicLibrary2009Id" varchar(6) CONSTRAINT "PK_Libraries_PublicLibrary2009" PRIMARY KEY, 
    "LibraryId" varchar(20) NOT NULL,
    "LibraryName" varchar(100) NOT NULL,
    "Address" varchar(35) NOT NULL,
    "City" varchar(20) NOT NULL,
    "Zip" char(5) NOT NULL,
    "Zip4" char(4) NOT NULL,
    "County" varchar(20) NOT NULL,
    -- additional columns omitted for brevity
    "FipsSt" varchar(2) NOT NULL,
    "FipsCo" varchar(3) NOT NULL
);

CREATE INDEX "IX_Libraries_PublicLibrary2009_LibraryName" ON "Libraries"."PublicLibrary2009" ("LibraryName");
CREATE INDEX "IX_Libraries_PublicLibrary2009_StateAbbr" ON "Libraries"."PublicLibrary2009" ("StateAbbreviation");
CREATE INDEX "IX_Libraries_PublicLibrary2009_City" ON "Libraries"."PublicLibrary2009" ("City");
CREATE INDEX "IX_Libraries_PublicLibrary2009_Visits" ON "Libraries"."PublicLibrary2009" ("Visits");

COPY "Libraries"."PublicLibrary2009"
FROM 'C:\YourDirectory\pls_fy2009_pupld09a.csv'
WITH (FORMAT CSV, HEADER);
```

**Explanation:**

- Similar to the 2014 table, but with slightly different columns reflecting changes in the survey over time
- Both tables share the same primary key column `PublicLibrary2009Id`, which will allow for joining the tables later
- The same indexing strategy is applied to improve query performance

## Exploring the Library Data Using Aggregate Functions

### Counting Rows and Values Using count()

#### PostgreSQL Standard Approach (snake_case) - Counting Rows

```sql
SELECT count(*)
FROM libraries.pls_fy2014_pupldl4a;

SELECT count(*)
FROM libraries.pls_fy2009_pupld09a;
```

**Output:**

```markdown
count
------
9305

count
------
9299
```

#### Domain-Driven Design Approach (PascalCase) - 2014 Library Data

```sql
SELECT count(*)
FROM "Libraries"."PublicLibrary2014";

SELECT count(*)
FROM "Libraries"."PublicLibrary2009";
```

**Output:**

```markdown
count
------
9305

count
------
9299
```

**Explanation:**

- These queries count all rows in each table regardless of whether columns contain NULL values
- The result shows 9,305 library agencies in the 2014 data and 9,299 in the 2009 data
- This verifies that the data was imported correctly according to the documentation

### Counting Values Present in a Column

#### PostgreSQL Standard Approach (snake_case) - Counting Rows

```sql
SELECT count("Salary") as "TotalSalaryCount"
FROM libraries.pls_fy2014_pupldl4a;
```

**Output:**

```markdown
count
------
5983
```

#### Domain-Driven Design Approach (PascalCase) - 2014 Library Data

```sql
SELECT count("Salary") as "TotalSalaryCount"
FROM "Libraries"."PublicLibrary2014";
```

**Output:**

```markdown
count
------
5983
```

**Explanation:**

- When specifying a column name in `count()`, only rows with non-NULL values in that column are counted
- The result shows only 5,983 library agencies reported salary data in 2014
- This is significantly less than the total number of rows (9,305), indicating many libraries did not report salary information
- This finding is important to consider when analyzing salary data, as it represents only about 64% of the total sample

### Counting Distinct Values in a Column

#### PostgreSQL Standard Approach (snake_case) - Counting Rows and Values

```sql
SELECT count(libname) as TotalNamedLibraryCount
FROM libraries.pls_fy2014_pupldl4a;

SELECT count(DISTINCT libname) as UniqueNamedLibraryCount
FROM libraries.pls_fy2014_pupldl4a;
```

**Output:**

```markdown
TotalNamedLibraryCount
------
9305

UniqueNamedLibraryCount
------
8515
```

#### Domain-Driven Design Approach (PascalCase) - Counting Rows

```sql
SELECT count("LibraryName")
FROM "Libraries"."PublicLibrary2014";

SELECT count(DISTINCT "LibraryName")
FROM "Libraries"."PublicLibrary2014";
```

**Output:**

```markdown
count
------
9305

count
------
8515
```

**Explanation:**

- The first query counts all non-NULL library names (all 9,305 rows have library names)
- The second query counts only unique library names, showing there are 8,515 distinct library names
- The difference indicates that 790 library names are duplicated across different agencies
- This makes sense because many towns with the same name exist in different states (like Oxford Public Library)

### Finding Maximum and Minimum Values Using max() and min()

#### PostgreSQL Standard Approach (snake_case) - Counting Rows and Values

```sql
SELECT max(visits), min(visits)
FROM libraries.pls_fy2014_pupldl4a
WHERE visits >= 0;
```

**Output:**

```markdown
    max     | min
-----------+-----
  17729020  | -3
```

#### Domain-Driven Design Approach (PascalCase) - Unique Heading

```sql
SELECT max("Visits") as "LibraryMaximumVisits", min("Visits") as "LibraryMinimumVisits"
FROM "Libraries"."PublicLibrary2014";
```

**Output:**

```markdown
    LibraryMaximumVisits     | LibraryMinimumVisits 
    ----------------------+---------------------
    17729020                 | -3
```

**Explanation:**

- The maximum value of over 17.7 million visits (`"LibraryMaximumVisits"`) is reasonable for a large library system
- The minimum value of -3 reveals that negative values (`"LibraryMinimumVisits"`) are used as codes rather than actual counts:
  - -1 indicates a "nonresponse" to that question
  - -3 indicates "not applicable" when a library has closed
- This finding is crucial as these negative values must be filtered out in subsequent analysis to avoid skewing results

## Aggregating Data Using GROUP BY

### Basic GROUP BY Usage

#### PostgreSQL Standard Approach (snake_case) - Counting Rows

```sql
SELECT stabr
FROM libraries.pls_fy2014_pupldl4a
GROUP BY stabr
ORDER BY stabr;
```

**Output:**

```markdown
stabr
------
AK
AL
AR
AS
AZ
CA
...
WV
WY
```

#### Domain-Driven Design Approach (PascalCase) - Counting Rows

```sql
SELECT "StateAbbreviation"
FROM "Libraries"."PublicLibrary2014"
GROUP BY "StateAbbreviation"
ORDER BY "StateAbbreviation";
```

**Output:**

```markdown
StateAbbreviation
---------
AK
AL
AR
AS
AZ
CA
...
WV
WY
```

**Explanation:**

- The `GROUP BY` clause groups rows with the same state abbreviation
- It returns unique values similar to using `DISTINCT`
- The result shows all 56 state/territory abbreviations in alphabetical order

### GROUP BY on Multiple Columns

#### PostgreSQL Standard Approach (snake_case) - Counting Rows and Values

```sql
SELECT city, stabr
FROM libraries.pls_fy2014_pupldl4a
GROUP BY city, stabr
ORDER BY city, stabr;
```

**Output:**

```markdown
city      | stabr
----------+-------
ABBEVILLE | AL
ABBEVILLE | LA
ABBEVILLE | SC
ABBOTSFORD| WI
ABERDEEN  | ID
ABERDEEN  | SD
ABERNATHY | TX
...
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT "City", "StateAbbreviation"
FROM "Libraries"."PublicLibrary2014"
GROUP BY "City", "StateAbbreviation"
ORDER BY "City", "StateAbbreviation";
```

**Output:**

```markdown
| City       | StateAbbreviation |
|------------|-------------------|
| ABBEVILLE  | AL                |
| ABBEVILLE  | LA                |
| ABBEVILLE  | SC                |
| ABBOTSFORD | WI                |
| ABERDEEN   | ID                |
| ABERDEEN   | SD                |
| ABERNATHY  | TX                |
| ...        | ...               |

```

**Explanation:**

- This query groups by unique combinations of city and state
- The result shows 9,088 rows, which is 217 fewer than the total table rows (9,305)
- This indicates that some cities have multiple library agencies within the same city and state

### Combining GROUP BY with count()

#### PostgreSQL Standard Approach (snake_case) - Example 1

```sql
SELECT stabr, count(*)
FROM libraries.pls_fy2014_pupldl4a
GROUP BY stabr
ORDER BY count(*) DESC;
```

**Output:**
```
stabr | count
------+------
NY    | 756
IL    | 625
TX    | 556
IA    | 543
PA    | 455
MI    | 389
WI    | 381
MA    | 370
...
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT "StateAbbreviation", count("StateAbbreviation") as "StateAbbreviationLibraryCount"
FROM "Libraries"."PublicLibrary2014"
GROUP BY "StateAbbreviation"
ORDER BY count("StateAbbreviation") DESC;
```

**Output:**

```markdown
| StateAbbreviation | StateAbbreviationLibraryCount |
|--------------------|-------------------------------|
| NY                | 756                           |
| IL                | 625                           |
| TX                | 556                           |
| IA                | 543                           |
| PA                | 455                           |
| MI                | 389                           |
| WI                | 381                           |
| MA                | 370                           |
| ...               | ...                           |

```

**Explanation:**

- This query counts the number of library agencies in each state/territory
- New York has the most library agencies (756), followed by Illinois (625) and Texas (556)
- The results show the distribution of library agencies across states but don't indicate the number of physical library branches

### Using GROUP BY on Multiple Columns with count()

#### PostgreSQL Standard Approach (snake_case)

```sql
SELECT stabr, stataddr, count(*)
FROM libraries.pls_fy2014_pupldl4a
GROUP BY stabr, stataddr
ORDER BY stabr ASC, count(*) DESC;
```

**Output:**

```markdown
stabr | stataddr | count
------+----------+------
AK    | 00       | 70
AK    | 15       | 10
AK    | 07       | 5
AL    | 00       | 221
AL    | 07       | 3
AR    | 00       | 58
AS    | 00       | 1
AZ    | 00       | 91
...
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT "StateAbbreviation", count("StateAbbreviation") as "StateAbbreviationLibraryCount"
FROM "Libraries"."PublicLibrary2014"
GROUP BY "StateAbbreviation","StateAddress"
ORDER BY "StateAbbreviation" ASC, count("StateAbbreviation") DESC;

```

**Output:**

| StateAbbreviation | StateAddress | StateAbbreviationLibraryCount |
|-------------------|--------------|-------------------------------|
| AK                | 00           | 70                            |
| AK                | 15           | 10                            |
| AK                | 07           | 5                             |
| AL                | 00           | 221                           |
| AL                | 07           | 3                             |
| AR                | 00           | 58                            |
| AS                | 00           | 1                             |
| AZ                | 00           | 91                            |
| ...               | ...          | ...                           |

```

**Explanation:**
- This query groups by state and address status code to count how many libraries in each state had address changes
- The `stataddr` codes represent:
  - 00: No change from last year
  - 07: Moved to a new location
  - 15: Minor address change
- For each state, code 00 (no change) is the most common, which is expected

## Revisiting sum() to Examine Library Visits

### Calculating Total Visits for Each Year

#### PostgreSQL Standard Approach (snake_case)

```sql
SELECT sum(visits) AS visits_2014
FROM libraries.pls_fy2014_pupldl4a
WHERE visits >= 0;

SELECT sum(visits) AS visits_2009
FROM libraries.pls_fy2009_pupld09a
WHERE visits >= 0;
```

**Output:**

```markdown
visits_2014
-----------
1425930900

visits_2009
-----------
1591799201
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT sum("Visits") AS "TotalVisitsIn2014"
FROM "Libraries"."PublicLibrary2014"
WHERE "Visits" >= 0;

SELECT sum("Visits") AS "TotalVisitsIn2009"
FROM "Libraries"."PublicLibrary2009"
WHERE "Visits" >= 0;
```

**Output:**

```markdown
TotalVisitsIn2014
-----------------
1425930900

TotalVisitsIn2009
-----------------
1591799201
```

**Explanation:**

- These queries sum all library visits for each year, filtering out negative values
- The WHERE clause excludes the negative codes (-1 and -3) that represent missing data
- The results show about 1.4 billion visits in 2014 and 1.6 billion in 2009
- This represents approximately a 10% decline in library visits over 5 years

### Comparing Visits Between Years Using Joins

#### PostgreSQL Standard Approach (snake_case)

```sql
SELECT sum(pls14.visits) AS visits_2014,
       sum(pls09.visits) AS visits_2009
FROM libraries.pls_fy2014_pupldl4a pls14 
JOIN libraries.pls_fy2009_pupld09a pls09
ON pls14.fscskey = pls09.fscskey
WHERE pls14.visits >= 0 AND pls09.visits >= 0;
```

**Output:**
```
visits_2014  | visits_2009
-------------+------------
1417299241   | 1585455205
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT sum(pl14."Visits") AS "Visits2014",
       sum(pl09."Visits") AS "Visits2009"
FROM "Libraries"."PublicLibrary2014" pl14 
JOIN "Libraries"."PublicLibrary2009" pl09
ON pl14."PublicLibrary2014Id" = pl09."PublicLibrary2009Id"
WHERE pl14."Visits" >= 0 AND pl09."Visits" >= 0;
```

**Output:**

```markdown

Visits2014   | Visits2009
-------------+------------
1417299241   | 1585455205
```

**Explanation:**

- This query joins the 2014 and 2009 tables on the `PublicLibrary2009Id` and `PublicLibrary2009Id` primary key
- By joining the tables, we only count visits for library agencies that exist in both years
- The totals are slightly lower (by 6-8 million) than when counting the tables separately
- The downward trend remains consistent, showing a decline in library visits

### Grouping Visit Sums by State

#### PostgreSQL Standard Approach (snake_case)

```sql
SELECT pls14.stabr,
       sum(pls14.visits) AS visits_2014,
       sum(pls09.visits) AS visits_2009,
       round( (CAST(sum(pls14.visits) AS decimal(10,1)) - sum(pls09.visits)) / 
              sum(pls09.visits) * 100, 2 ) AS pct_change
FROM libraries.pls_fy2014_pupldl4a pls14 
JOIN libraries.pls_fy2009_pupld09a pls09
ON pls14.fscskey = pls09.fscskey
WHERE pls14.visits >= 0 AND pls09.visits >= 0
GROUP BY pls14.stabr
ORDER BY pct_change DESC;
```

**Output Excerpt:**

```markdown
stabr | visits_2014 | visits_2009 | pct_change
------+-------------+-------------+-----------
GU    | 103593      | 60763       | 70.49
DC    | 4230790     | 2944774     | 43.67
LA    | 17242110    | 15591805    | 10.58
MT    | 4582604     | 4386504     | 4.47
AL    | 17113602    | 16933967    | 1.06
...
RI    | 5259143     | 6612167     | -20.46
NC    | 33952977    | 43111094    | -21.24
PR    | 193279      | 257032      | -24.80
GA    | 28891017    | 40922598    | -29.40
OK    | 13678542    | 21171452    | -35.39
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT pl14."StateAbbreviation",
       sum(pl14."Visits") AS "Visits2014",
       sum(pl09."Visits") AS "Visits2009",
       round( (CAST(sum(pl14."Visits") AS decimal(10,1)) - sum(pl09."Visits")) / 
              sum(pl09."Visits") * 100, 2 ) AS "PercentChange"
FROM "Libraries"."PublicLibrary2014" pl14 
JOIN "Libraries"."PublicLibrary2009" pl09
ON pl14."PublicLibrary2014Id" = pl09."PublicLibrary2009Id"
WHERE pl14."Visits" >= 0 AND pl09."Visits" >= 0
GROUP BY pl14."StateAbbreviation"
ORDER BY "PercentChange" DESC;
```

**Output Excerpt:**

| StateAbbreviation | Visits2014 | Visits2009 | PercentChange |
|--------------------|------------|------------|---------------|
| GU                | 103593     | 60763      | 70.49         |
| DC                | 4230790    | 2944774    | 43.67         |
| LA                | 17242110   | 15591805   | 10.58         |
| MT                | 4582604    | 4386504    | 4.47          |
| AL                | 17113602   | 16933967   | 1.06          |
| ...               | ...        | ...        | ...           |
| RI                | 5259143    | 6612167    | -20.46        |
| NC                | 33952977   | 43111094   | -21.24        |
| PR                | 193279     | 257032     | -24.80        |
| GA                | 28891017   | 40922598   | -29.40        |
| OK                | 13678542   | 21171452   | -35.39        |
--------------------+------------+------------+----------------

```

**Explanation:**
- This query calculates the percentage change in library visits by state between 2009 and 2014
- Results are grouped by state abbreviation and ordered by percentage change (largest increase to largest decrease)
- Only 10 states/territories showed an increase in visits, with Guam having the largest increase (70.49%)
- Oklahoma had the largest decrease at -35.39%
- This detailed breakdown shows significant regional variation in the trend of library visits

### Filtering an Aggregate Query Using HAVING

#### PostgreSQL Standard Approach (snake_case)

```sql
SELECT pls14.stabr,
       sum(pls14.visits) AS visits_2014,
       sum(pls09.visits) AS visits_2009,
       round( (CAST(sum(pls14.visits) AS decimal(10,1)) - sum(pls09.visits)) / 
              sum(pls09.visits) * 100, 2 ) AS pct_change
FROM libraries.pls_fy2014_pupldl4a pls14 
JOIN libraries.pls_fy2009_pupld09a pls09
ON pls14.fscskey = pls09.fscskey
WHERE pls14.visits >= 0 AND pls09.visits >= 0
GROUP BY pls14.stabr
HAVING sum(pls14.visits) > 50000000
ORDER BY pct_change DESC;
```

**Output:**
```
stabr | visits_2014 | visits_2009 | pct_change
------+-------------+-------------+-----------
TX    | 72876601    | 78838400    | -7.56
CA    | 162787836   | 182181408   | -10.65
OH    | 82495138    | 92402369    | -10.72
NY    | 106453546   | 119810969   | -11.15
IL    | 72598213    | 82438755    | -11.94
FL    | 73165352    | 87730886    | -16.60
```

#### Domain-Driven Design Approach (PascalCase)

```sql
SELECT pl14."StateAbbreviation",
       sum(pl14."Visits") AS "Visits2014",
       sum(pl09."Visits") AS "Visits2009",
       round( (CAST(sum(pl14."Visits") AS decimal(10,1)) - sum(pl09."Visits")) / 
              sum(pl09."Visits") * 100, 2 ) AS "PercentChange"
FROM "Libraries"."PublicLibrary2014" pl14 
JOIN "Libraries"."PublicLibrary2009" pl09
ON pl14."PublicLibrary2014Id" = pl09."PublicLibrary2009Id"
WHERE pl14."Visits" >= 0 AND pl09."Visits" >= 0
GROUP BY pl14."StateAbbreviation"
HAVING sum(pl14."Visits") > 50000000
ORDER BY "PercentChange" DESC;

```

**Output:**

```markdown
| StateAbbreviation | Visits2014  | Visits2009  | PercentChange |
|--------------------|-------------|-------------|---------------|
| TX                | 72,876,601  | 78,838,400  | -7.56         |
| CA                | 162,787,836 | 182,181,408 | -10.65        |
| OH                | 82,495,138  | 92,402,369  | -10.72        |
| NY                | 106,453,546 | 119,810,969 | -11.15        |
| IL                | 72,598,213  | 82,438,755  | -11.94        |
| FL                | 73,165,352  | 87,730,886  | -16.60        |
| ...               | ...         | ...         | ...           |
--------------------+-------------+-------------+----------------

```

**Explanation:**

- This query adds a `HAVING` clause to filter the grouped results
- Only states with more than 50 million library visits in 2014 are included in the results
- All six states with the highest library visit counts showed decreases ranging from 7.56% to 16.60%
- This analysis focuses only on the largest states by library usage, showing a more consistent pattern of decline
- The `HAVING` clause operates on aggregated data (after the `GROUP BY`), unlike the `WHERE` clause which filters rows before aggregation

## Summary

This chapter demonstrates how to extract meaningful information from data through aggregation and grouping techniques in PostgreSQL. Key concepts covered include:

1. **Using aggregate functions** like `count()`, `max()`, `min()`, and `sum()` to summarize data
2. **Grouping data** with the `GROUP BY` clause to organize results by categories
3. **Filtering aggregated data** with the `HAVING` clause
4. **Joining tables** to compare data across different time periods
5. **Calculating percentage changes** to identify trends

These techniques enabled us to discover several important insights from the library dataset:

- Overall library visits declined by approximately 10% from 2009 to 2014
- Regional differences showed significant variation, with some areas increasing visits while others declined
- The largest states by library usage all experienced decreasing visit counts
- Data coding practices (using negative values as indicators) highlighted the importance of data cleaning

These SQL aggregation and grouping techniques are essential tools for data analysis, allowing analysts to transform raw data into actionable insights.

## Try It Yourself Exercises

1. Analyze technology usage trends in libraries:
   - Compare the `gpterms` (internet-connected computers) and `pitusr` (public internet computer uses) columns between 2009 and 2014
   - Calculate the percent change for each metric
   - Be sure to filter out negative values

2. Analyze library visit changes by U.S. region:
   - Use the `obereg` column (Bureau of Economic Analysis Code) to group libraries by region
   - Calculate the percent change in visits by region
   - Create a reference table to map region codes to names for more readable output

3. Find libraries that appear in only one of the two tables:
   - Use an outer join to show all rows from both tables
   - Add an `IS NULL` filter to identify libraries that exist in one year but not the other
   - Determine potential reasons for libraries appearing or disappearing between surveys