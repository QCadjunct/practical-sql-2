# Chapter 12 - Advanced Query Techniques

## Introduction

This chapter explores advanced SQL techniques that extend beyond basic queries. These techniques help handle complex problems when basic SELECT statements and JOIN operations aren't sufficient. The chapter introduces methods for finding deeper insights in data through sophisticated SQL features including subqueries, Common Table Expressions (CTEs), cross tabulations, and conditional logic with CASE statements.

## Using Subqueries

Subqueries (queries nested inside other queries) serve multiple purposes in SQL. This powerful technique enables more complex data manipulation by using the results of one query as input to another.

### PostgreSQL Standard (snake_case):

```sql
-- Subquery example showing the basic syntax pattern
UPDATE analytics.table_name
SET column = (SELECT column
              FROM analytics.table_b
              WHERE analytics.table_name.column = analytics.table_b.column)
WHERE EXISTS (SELECT column
              FROM analytics.table_b
              WHERE analytics.table_name.column = analytics.table_b.column);
```

### Domain Driven Design (PascalCase):
```sql
-- Subquery example showing the basic syntax pattern
UPDATE "Analytics"."Table"
SET "Column" = (SELECT "Column"
                FROM "Analytics"."ReferenceTable"
                WHERE "Analytics"."Table"."Column" = "Analytics"."ReferenceTable"."Column")
WHERE EXISTS (SELECT "Column"
              FROM "Analytics"."ReferenceTable"
              WHERE "Analytics"."Table"."Column" = "Analytics"."ReferenceTable"."Column");
```

## Filtering with Subqueries in a WHERE Clause

Subqueries can generate values for filtering criteria in WHERE clauses, eliminating the need to write separate queries.

### Generating Values for a Query Expression

#### PostgreSQL Standard (snake_case):
```sql
SELECT geo_name,
       state_us_abbreviation,
       p0010001
FROM census.us_counties_2010
WHERE p0010001 >= (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY p0010001)
    FROM census.us_counties_2010
)
ORDER BY p0010001 DESC;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "CountyName",
       "StateAbbreviation",
       "TotalPopulation"
FROM "Geography"."County"
WHERE "TotalPopulation" >= (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY "TotalPopulation")
    FROM "Geography"."County"
)
ORDER BY "TotalPopulation" DESC;
```

**Output:**
```
CountyName          StateAbbreviation   TotalPopulation
----------------    -----------------   ---------------
Los Angeles County  CA                  9818605
Cook County         IL                  5194675
Harris County       TX                  4092459
Maricopa County     AZ                  3817117
San Diego County    CA                  3095313
...
Elkhart County      IN                  197559
Sangamon County     IL                  197465
```

This query identifies counties with populations at or above the 90th percentile. The subquery calculates the 90th percentile (197444.6), and the main query returns the 315 counties that meet this criterion.

### Using a Subquery to Identify Rows to Delete

Subqueries can also specify what to remove from a table, which is useful when working with large datasets.

#### PostgreSQL Standard (snake_case):
```sql
-- Create a copy of the census table
CREATE TABLE census.us_counties_2010_top10 AS
SELECT * FROM census.us_counties_2010;

-- Delete counties below the 90th percentile
DELETE FROM census.us_counties_2010_top10
WHERE p0010001 < (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY p0010001)
    FROM census.us_counties_2010_top10
);
```

#### Domain Driven Design (PascalCase):
```sql
-- Create a copy of the census table
CREATE TABLE "Geography"."TopPopulationCounty" AS
SELECT * FROM "Geography"."County";

-- Delete counties below the 90th percentile
DELETE FROM "Geography"."TopPopulationCounty"
WHERE "TotalPopulation" < (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY "TotalPopulation")
    FROM "Geography"."TopPopulationCounty"
);
```

After execution, only 315 rows (top 10% by population) remain in the table.

## Creating Derived Tables with Subqueries

A derived table is created when a subquery in a FROM clause returns rows and columns of data. This technique allows for multi-step data processing within a single query.

#### PostgreSQL Standard (snake_case):
```sql
SELECT round(calcs.average, 0) AS average,
       calcs.median,
       round(calcs.average - calcs.median, 0) AS median_average_diff
FROM (
    SELECT avg(p0010001) AS average,
           percentile_cont(.5)
           WITHIN GROUP (ORDER BY p0010001)::numeric(10,1) AS median
    FROM census.us_counties_2010
) AS calcs;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT round("PopulationStats"."AveragePopulation", 0) AS "Average",
       "PopulationStats"."MedianPopulation",
       round("PopulationStats"."AveragePopulation" - "PopulationStats"."MedianPopulation", 0) AS "MedianAverageDifference"
FROM (
    SELECT avg("TotalPopulation") AS "AveragePopulation",
           percentile_cont(.5)
           WITHIN GROUP (ORDER BY "TotalPopulation")::numeric(10,1) AS "MedianPopulation"
    FROM "Geography"."County"
) AS "PopulationStats";
```

**Output:**
```
Average    MedianPopulation    MedianAverageDifference
---------  -----------------   -----------------------
98233      25857               72376
```

This calculation reveals that the average county population (98,233) is significantly higher than the median (25,857), indicating that a few high-population counties skew the average upward.

## Joining Derived Tables

Derived tables can be joined together, enabling complex preprocessing steps before final analysis.

#### PostgreSQL Standard (snake_case):
```sql
SELECT census.state_us_abbreviation AS st,
       census.st_population,
       plants.plant_count,
       round((plants.plant_count/census.st_population::numeric(10,1))*1000000, 1) AS plants_per_million
FROM (
    SELECT st,
           count(*) AS plant_count
    FROM inspection.meat_poultry_egg_inspect
    GROUP BY st
) AS plants
JOIN (
    SELECT state_us_abbreviation,
           sum(p0010001) AS st_population
    FROM census.us_counties_2010
    GROUP BY state_us_abbreviation
) AS census
ON plants.st = census.state_us_abbreviation
ORDER BY plants_per_million DESC;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "StateCensus"."StateAbbreviation" AS "State",
       "StateCensus"."StatePopulation",
       "StateInspection"."FacilityCount",
       round(("StateInspection"."FacilityCount"/"StateCensus"."StatePopulation"::numeric(10,1))*1000000, 1) AS "FacilitiesPerMillion"
FROM (
    SELECT "State",
           count(*) AS "FacilityCount"
    FROM "FoodSafety"."InspectionFacility"
    GROUP BY "State"
) AS "StateInspection"
JOIN (
    SELECT "StateAbbreviation",
           sum("TotalPopulation") AS "StatePopulation"
    FROM "Geography"."County"
    GROUP BY "StateAbbreviation"
) AS "StateCensus"
ON "StateInspection"."State" = "StateCensus"."StateAbbreviation"
ORDER BY "FacilitiesPerMillion" DESC;
```

**Output:**
```
State  StatePopulation  FacilityCount  FacilitiesPerMillion
-----  --------------   ------------   -------------------
NE     1826341          110            60.2
IA     3046355          149            48.9
VT     625741           27             43.1
HI     1360301          47             34.6
ND     672591           22             32.7
...
SC     4625364          55             11.9
LA     4533372          49             10.8
AZ     6392017          37             5.8
DC     601723           2              3.3
WY     563626           1              1.8
```

This query calculates the rate of meat, poultry, and egg processing facilities per million population for each state, showing that Nebraska, Iowa, and Vermont have the highest concentrations.

## Generating Columns with Subqueries

Subqueries can be placed in the column list after SELECT to create new columns with calculated values.

#### PostgreSQL Standard (snake_case):
```sql
SELECT geo_name,
       state_us_abbreviation AS st,
       p0010001 AS total_pop,
       (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001)
        FROM census.us_counties_2010) AS us_median
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "CountyName",
       "StateAbbreviation" AS "State",
       "TotalPopulation",
       (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY "TotalPopulation")
        FROM "Geography"."County") AS "NationalMedianPopulation"
FROM "Geography"."County";
```

**Output:**
```
CountyName        State   TotalPopulation  NationalMedianPopulation
--------------    -----   ---------------  ------------------------
Autauga County    AL      54571            25857
Baldwin County    AL      182265           25857
Barbour County    AL      27457            25857
Bibb County       AL      22915            25857
Blount County     AL      57322            25857
```

This adds a consistent value (the national median population) to each row for reference.

### Finding Counties Close to the National Median

We can extend this technique to find counties with populations close to the national median:

#### PostgreSQL Standard (snake_case):
```sql
SELECT geo_name,
       state_us_abbreviation AS st,
       p0010001 AS total_pop,
       (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001)
        FROM census.us_counties_2010) AS us_median,
       p0010001 - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001)
                  FROM census.us_counties_2010) AS diff_from_median
FROM census.us_counties_2010
WHERE (p0010001 - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001)
                  FROM census.us_counties_2010))
BETWEEN -1000 AND 1000;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "CountyName",
       "StateAbbreviation" AS "State",
       "TotalPopulation",
       (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY "TotalPopulation")
        FROM "Geography"."County") AS "NationalMedianPopulation",
       "TotalPopulation" - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY "TotalPopulation")
                           FROM "Geography"."County") AS "DifferenceFromMedian"
FROM "Geography"."County"
WHERE ("TotalPopulation" - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY "TotalPopulation")
                           FROM "Geography"."County"))
BETWEEN -1000 AND 1000;
```

**Output:**
```
CountyName          State   TotalPopulation  NationalMedianPopulation  DifferenceFromMedian
------------------  -----   ---------------  ------------------------  -------------------
Cherokee County     AL      25989            25857                     132
Clarke County       AL      25833            25857                     -24
Geneva County       AL      26790            25857                     933
Cleburne County     AR      25970            25857                     113
Johnson County      AR      25540            25857                     -317
```

This query finds 71 counties with populations within 1,000 of the national median.

## Subquery Expressions

Subqueries can be used with special expressions to filter rows based on whether conditions evaluate as true or false.

### Generating Values for the IN Operator

The IN operator can use subqueries to generate comparison values:

#### PostgreSQL Standard (snake_case):
```sql
SELECT first_name, last_name
FROM hr.employees
WHERE id IN (
    SELECT id
    FROM hr.retirees
);
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "FirstName", "LastName"
FROM "HumanResources"."Employee"
WHERE "EmployeeId" IN (
    SELECT "EmployeeId"
    FROM "HumanResources"."Retiree"
);
```

This finds employees whose IDs appear in the retirees table.

### Checking Whether Values Exist

The EXISTS expression tests whether a subquery returns at least one row:

#### PostgreSQL Standard (snake_case):
```sql
-- Find employees with matching retiree records
SELECT first_name, last_name
FROM hr.employees
WHERE EXISTS (
    SELECT id
    FROM hr.retirees
    WHERE hr.retirees.id = hr.employees.id
);

-- Find employees without matching retiree records
SELECT first_name, last_name
FROM hr.employees
WHERE NOT EXISTS (
    SELECT id
    FROM hr.retirees
    WHERE hr.retirees.id = hr.employees.id
);
```

#### Domain Driven Design (PascalCase):
```sql
-- Find employees with matching retiree records
SELECT "FirstName", "LastName"
FROM "HumanResources"."Employee"
WHERE EXISTS (
    SELECT "EmployeeId"
    FROM "HumanResources"."Retiree"
    WHERE "HumanResources"."Retiree"."EmployeeId" = "HumanResources"."Employee"."EmployeeId"
);

-- Find employees without matching retiree records
SELECT "FirstName", "LastName"
FROM "HumanResources"."Employee"
WHERE NOT EXISTS (
    SELECT "EmployeeId"
    FROM "HumanResources"."Retiree"
    WHERE "HumanResources"."Retiree"."EmployeeId" = "HumanResources"."Employee"."EmployeeId"
);
```

The EXISTS expression is particularly useful when joining on multiple columns or when checking for the presence of matching records.

## Common Table Expressions

Common Table Expressions (CTEs) provide an alternative to derived tables. They define one or more temporary tables at the beginning of a query, making complex queries more readable and maintainable.

### Simple CTE Example

#### PostgreSQL Standard (snake_case):
```sql
WITH
large_counties (geo_name, st, p0010001)
AS
(
    SELECT geo_name, state_us_abbreviation, p0010001
    FROM census.us_counties_2010
    WHERE p0010001 >= 100000
)
SELECT st, count(*)
FROM large_counties
GROUP BY st
ORDER BY count(*) DESC;
```

#### Domain Driven Design (PascalCase):
```sql
WITH
"LargeCounty" ("CountyName", "State", "Population")
AS
(
    SELECT "CountyName", "StateAbbreviation", "TotalPopulation"
    FROM "Geography"."County"
    WHERE "TotalPopulation" >= 100000
)
SELECT "State", count(*)
FROM "LargeCounty"
GROUP BY "State"
ORDER BY count(*) DESC;
```

**Output:**
```
State  count
-----  -----
TX     39
CA     35
FL     33
PA     31
OH     28
```

This CTE defines a temporary table of counties with populations of 100,000 or more, then counts them by state.

### Using CTEs for More Complex Analysis

CTEs can simplify complex queries that join derived tables:

#### PostgreSQL Standard (snake_case):
```sql
WITH
counties (st, population) AS
(
    SELECT state_us_abbreviation, sum(p0010001)
    FROM census.us_counties_2010
    GROUP BY state_us_abbreviation
),
plants (st, plants) AS
(
    SELECT st, count(*) AS plants
    FROM inspection.meat_poultry_egg_inspect
    GROUP BY st
)
SELECT counties.st,
       population,
       plants,
       round((plants/population::numeric(10,1)) * 1000000, 1) AS per_million
FROM counties JOIN plants
    ON counties.st = plants.st
ORDER BY per_million DESC;
```

#### Domain Driven Design (PascalCase):
```sql
WITH
"StatePopulation" ("State", "TotalPopulation") AS
(
    SELECT "StateAbbreviation", sum("TotalPopulation")
    FROM "Geography"."County"
    GROUP BY "StateAbbreviation"
),
"StateFacilityCount" ("State", "FacilityCount") AS
(
    SELECT "State", count(*) AS "FacilityCount"
    FROM "FoodSafety"."InspectionFacility"
    GROUP BY "State"
)
SELECT "StatePopulation"."State",
       "TotalPopulation",
       "FacilityCount",
       round(("FacilityCount"/"TotalPopulation"::numeric(10,1)) * 1000000, 1) AS "FacilitiesPerMillion"
FROM "StatePopulation" JOIN "StateFacilityCount"
    ON "StatePopulation"."State" = "StateFacilityCount"."State"
ORDER BY "FacilitiesPerMillion" DESC;
```

This provides the same results as the earlier derived table example but with cleaner code structure.

### Using CTEs to Minimize Redundant Code

CTEs can also reduce code duplication:

#### PostgreSQL Standard (snake_case):
```sql
WITH us_median AS
(
    SELECT percentile_cont(.5)
    WITHIN GROUP (ORDER BY p0010001) AS us_median_pop
    FROM census.us_counties_2010
)
SELECT geo_name,
       state_us_abbreviation AS st,
       p0010001 AS total_pop,
       us_median_pop,
       p0010001 - us_median_pop AS diff_from_median
FROM census.us_counties_2010 CROSS JOIN us_median
WHERE (p0010001 - us_median_pop)
BETWEEN -1000 AND 1000;
```

#### Domain Driven Design (PascalCase):
```sql
WITH "NationalMedian" AS
(
    SELECT percentile_cont(.5)
    WITHIN GROUP (ORDER BY "TotalPopulation") AS "MedianPopulation"
    FROM "Geography"."County"
)
SELECT "CountyName",
       "StateAbbreviation" AS "State",
       "TotalPopulation",
       "MedianPopulation",
       "TotalPopulation" - "MedianPopulation" AS "DifferenceFromMedian"
FROM "Geography"."County" CROSS JOIN "NationalMedian"
WHERE ("TotalPopulation" - "MedianPopulation")
BETWEEN -1000 AND 1000;
```

This query provides the same results as the earlier version that used the same subquery multiple times, but the CTE approach is more efficient and easier to maintain.

## Cross Tabulations

Cross tabulations (crosstabs) present data in a matrix format, with rows representing one variable, columns representing another, and cells containing values where the variables intersect.

### Installing the crosstab() Function

The crosstab() function is part of PostgreSQL's tablefunc module:

```sql
CREATE EXTENSION tablefunc;
```

### Tabulating Survey Results

Using the ice cream survey data:

#### PostgreSQL Standard (snake_case):
```sql
CREATE TABLE survey.ice_cream_survey (
    response_id integer PRIMARY KEY,
    office varchar(20),
    flavor varchar(20)
);

COPY survey.ice_cream_survey
FROM '/path/to/ice_cream_survey.csv'
WITH (FORMAT CSV, HEADER);

SELECT *
FROM crosstab('SELECT office,
                      flavor,
                      count(*)
               FROM survey.ice_cream_survey
               GROUP BY office, flavor
               ORDER BY office',
              'SELECT flavor
               FROM survey.ice_cream_survey
               GROUP BY flavor
               ORDER BY flavor')
AS (office varchar(20),
    chocolate bigint,
    strawberry bigint,
    vanilla bigint);
```

#### Domain Driven Design (PascalCase):
```sql
CREATE TABLE "Survey"."IceCreamPreference" (
    "ResponseId" integer PRIMARY KEY,
    "Office" varchar(20),
    "Flavor" varchar(20)
);

COPY "Survey"."IceCreamPreference"
FROM '/path/to/ice_cream_survey.csv'
WITH (FORMAT CSV, HEADER);

SELECT *
FROM crosstab('SELECT "Office",
                      "Flavor",
                      count(*)
               FROM "Survey"."IceCreamPreference"
               GROUP BY "Office", "Flavor"
               ORDER BY "Office"',
              'SELECT "Flavor"
               FROM "Survey"."IceCreamPreference"
               GROUP BY "Flavor"
               ORDER BY "Flavor"')
AS ("Office" varchar(20),
    "Chocolate" bigint,
    "Strawberry" bigint,
    "Vanilla" bigint);
```

**Output:**
```
Office      Chocolate   Strawberry   Vanilla
----------  ----------  -----------  -------
Downtown    23          32           19
Midtown     41          null         23
Uptown      22          17           23
```

This crosstab makes it easy to see preferences by office location, revealing that Midtown strongly prefers chocolate while Downtown favors strawberry.

### Tabulating City Temperature Readings

Using temperature data from weather stations:

#### PostgreSQL Standard (snake_case):
```sql
CREATE TABLE weather.temperature_readings (
    reading_id bigserial,
    station_name varchar(50),
    observation_date date,
    max_temp integer,
    min_temp integer
);

COPY weather.temperature_readings
(station_name, observation_date, max_temp, min_temp)
FROM '/path/to/temperature_readings.csv'
WITH (FORMAT CSV, HEADER);

SELECT *
FROM crosstab('SELECT
                station_name,
                date_part(''month'', observation_date),
                percentile_cont(.5)
                WITHIN GROUP (ORDER BY max_temp)
              FROM weather.temperature_readings
              GROUP BY station_name, date_part(''month'', observation_date)
              ORDER BY station_name',
             'SELECT month
              FROM generate_series(1,12) month')
AS (station varchar(50),
    jan numeric(3,0),
    feb numeric(3,0),
    mar numeric(3,0),
    apr numeric(3,0),
    may numeric(3,0),
    jun numeric(3,0),
    jul numeric(3,0),
    aug numeric(3,0),
    sep numeric(3,0),
    oct numeric(3,0),
    nov numeric(3,0),
    dec numeric(3,0)
);
```

#### Domain Driven Design (PascalCase):
```sql
CREATE TABLE "Weather"."TemperatureReading" (
    "ReadingId" bigserial,
    "StationName" varchar(50),
    "ObservationDate" date,
    "MaxTemperature" integer,
    "MinTemperature" integer
);

COPY "Weather"."TemperatureReading"
("StationName", "ObservationDate", "MaxTemperature", "MinTemperature")
FROM '/path/to/temperature_readings.csv'
WITH (FORMAT CSV, HEADER);

SELECT *
FROM crosstab('SELECT
                "StationName",
                date_part(''month'', "ObservationDate"),
                percentile_cont(.5)
                WITHIN GROUP (ORDER BY "MaxTemperature")
              FROM "Weather"."TemperatureReading"
              GROUP BY "StationName", date_part(''month'', "ObservationDate")
              ORDER BY "StationName"',
             'SELECT month
              FROM generate_series(1,12) month')
AS ("Station" varchar(50),
    "Jan" numeric(3,0),
    "Feb" numeric(3,0),
    "Mar" numeric(3,0),
    "Apr" numeric(3,0),
    "May" numeric(3,0),
    "Jun" numeric(3,0),
    "Jul" numeric(3,0),
    "Aug" numeric(3,0),
    "Sep" numeric(3,0),
    "Oct" numeric(3,0),
    "Nov" numeric(3,0),
    "Dec" numeric(3,0)
);
```

**Output:**
```
Station                           Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec
--------------------------------- --- --- --- --- --- --- --- --- --- --- --- ---
CHICAGO NORTHERLY ISLAND IL US     34  36  46  50  66  77  81  80  77  65  57  35
SEATTLE BOEING FIELD WA US         50  54  56  64  66  71  76  77  69  62  55  42
WAIKIKI 717.2 HI US               83  84  84  86  87  87  88  87  87  86  84  82
```

This crosstab presents median high temperatures by month for each station, making it easy to compare seasonal patterns between locations.

## Reclassifying Values with CASE

The CASE statement allows for conditional logic in SQL queries, which is particularly useful for reclassifying values into categories.

### Basic CASE Example

#### PostgreSQL Standard (snake_case):
```sql
SELECT max_temp,
       CASE WHEN max_temp >= 90 THEN 'Hot'
            WHEN max_temp BETWEEN 70 AND 89 THEN 'Warm'
            WHEN max_temp BETWEEN 50 AND 69 THEN 'Pleasant'
            WHEN max_temp BETWEEN 33 AND 49 THEN 'Cold'
            WHEN max_temp BETWEEN 20 AND 32 THEN 'Freezing'
            ELSE 'Inhumane'
       END AS temperature_group
FROM weather.temperature_readings;
```

#### Domain Driven Design (PascalCase):
```sql
SELECT "MaxTemperature",
       CASE WHEN "MaxTemperature" >= 90 THEN 'Hot'
            WHEN "MaxTemperature" BETWEEN 70 AND 89 THEN 'Warm'
            WHEN "MaxTemperature" BETWEEN 50 AND 69 THEN 'Pleasant'
            WHEN "MaxTemperature" BETWEEN 33 AND 49 THEN 'Cold'
            WHEN "MaxTemperature" BETWEEN 20 AND 32 THEN 'Freezing'
            ELSE 'Inhumane'
       END AS "TemperatureCategory"
FROM "Weather"."TemperatureReading";
```

**Output:**
```
MaxTemperature  TemperatureCategory
--------------  ------------------
31              Freezing
34              Cold
32              Freezing
32              Freezing
34              Cold
```

### Using CASE in a Common Table Expression

We can combine the CASE expression with a CTE to analyze temperature patterns:

#### PostgreSQL Standard (snake_case):
```sql
WITH temps_collapsed (station_name, max_temperature_group) AS
(
    SELECT station_name,
           CASE WHEN max_temp >= 90 THEN 'Hot'
                WHEN max_temp BETWEEN 70 AND 89 THEN 'Warm'
                WHEN max_temp BETWEEN 50 AND 69 THEN 'Pleasant'
                WHEN max_temp BETWEEN 33 AND 49 THEN 'Cold'
                WHEN max_temp BETWEEN 20 AND 32 THEN 'Freezing'
                ELSE 'Inhumane'
           END
    FROM weather.temperature_readings
)
SELECT station_name, max_temperature_group, count(*)
FROM temps_collapsed
GROUP BY station_name, max_temperature_group
ORDER BY station_name, count(*) DESC;
```

#### Domain Driven Design (PascalCase):
```sql
WITH "TemperatureCategory" ("StationName", "Category") AS
(
    SELECT "StationName",
           CASE WHEN "MaxTemperature" >= 90 THEN 'Hot'
                WHEN "MaxTemperature" BETWEEN 70 AND 89 THEN 'Warm'
                WHEN "MaxTemperature" BETWEEN 50 AND 69 THEN 'Pleasant'
                WHEN "MaxTemperature" BETWEEN 33 AND 49 THEN 'Cold'
                WHEN "MaxTemperature" BETWEEN 20 AND 32 THEN 'Freezing'
                ELSE 'Inhumane'
           END
    FROM "Weather"."TemperatureReading"
)
SELECT "StationName", "Category", count(*)
FROM "TemperatureCategory"
GROUP BY "StationName", "Category"
ORDER BY "StationName", count(*) DESC;
```

**Output:**
```
StationName                      Category   count
-------------------------------- ---------- -----
CHICAGO NORTHERLY ISLAND IL US   Pleasant   198
CHICAGO NORTHERLY ISLAND IL US   Warm       98
CHICAGO NORTHERLY ISLAND IL US   Cold       50
CHICAGO NORTHERLY ISLAND IL US   Freezing   30
CHICAGO NORTHERLY ISLAND IL US   Inhumane   8
CHICAGO NORTHERLY ISLAND IL US   Hot        5
SEATTLE BOEING FIELD WA US       Warm       133
SEATTLE BOEING FIELD WA US       Cold       92
SEATTLE BOEING FIELD WA US       Pleasant   91
SEATTLE BOEING FIELD WA US       Freezing   30
SEATTLE BOEING FIELD WA US       Inhumane   8
SEATTLE BOEING FIELD WA US       Hot        3
WAIKIKI 717.2 HI US             Warm       361
WAIKIKI 717.2 HI US             Hot        5
```

This analysis shows that Waikiki has 361 warm days per year, making it an appealing vacation destination. Seattle offers nearly 300 days of pleasant or warm temperatures, though its rainfall isn't considered in this analysis. Chicago has 38 days of freezing or "inhumane" maximum temperatures.

## Try It Yourself Exercises

1. Revise the code to analyze Waikiki's high temperatures in more specific categories:
   
   #### PostgreSQL Standard (snake_case):
   ```sql
   WITH waikiki_temps (temperature_group) AS
   (
       SELECT CASE WHEN max_temp >= 90 THEN '90 or more'
                   WHEN max_temp BETWEEN 88 AND 89 THEN '88-89'
                   WHEN max_temp BETWEEN 86 AND 87 THEN '86-87'
                   WHEN max_temp BETWEEN 84 AND 85 THEN '84-85'
                   WHEN max_temp BETWEEN 82 AND 83 THEN '82-83'
                   WHEN max_temp BETWEEN 80 AND 81 THEN '80-81'
                   ELSE '79 or less'
              END
       FROM weather.temperature_readings
       WHERE station_name = 'WAIKIKI 717.2 HI US'
   )
   SELECT temperature_group, count(*)
   FROM waikiki_temps
   GROUP BY temperature_group
   ORDER BY count(*) DESC;
   ```

   #### Domain Driven Design (PascalCase):

  ```sql
WITH "WaikikiTemperature" ("Category") AS
(
    SELECT CASE WHEN "MaxTemperature" >= 90 THEN '90 or more'
                WHEN "MaxTemperature" BETWEEN 88 AND 89 THEN '88-89'
                WHEN "MaxTemperature" BETWEEN 86 AND 87 THEN '86-87'
                WHEN "MaxTemperature" BETWEEN 84 AND 85 THEN '84-85'
                WHEN "MaxTemperature" BETWEEN 82 AND 83 THEN '82-83'
                WHEN "MaxTemperature" BETWEEN 80 AND 81 THEN '80-81'
                ELSE '79 or less'
           END
    FROM "Weather"."TemperatureReading"
    WHERE "StationName" = 'WAIKIKI 717.2 HI US'
)
SELECT "Category", count(*)
FROM "WaikikiTemperature"
GROUP BY "Category"
ORDER BY count(*) DESC;
```

The result would show which temperature range is most common in Waikiki's daily maximum temperatures, allowing for a more detailed understanding of its climate patterns.

2. Flip the ice cream survey crosstab to show flavors as rows and offices as columns:

#### PostgreSQL Standard (snake_case):
```sql
SELECT *
FROM crosstab('SELECT flavor,
                      office,
                      count(*)
               FROM survey.ice_cream_survey
               GROUP BY flavor, office
               ORDER BY flavor',
              'SELECT office
               FROM survey.ice_cream_survey
               GROUP BY office
               ORDER BY office')
AS (flavor varchar(20),
    downtown bigint,
    midtown bigint,
    uptown bigint);
```

#### Domain Driven Design (PascalCase):
```sql
SELECT *
FROM crosstab('SELECT "Flavor",
                      "Office",
                      count(*)
               FROM "Survey"."IceCreamPreference"
               GROUP BY "Flavor", "Office"
               ORDER BY "Flavor"',
              'SELECT "Office"
               FROM "Survey"."IceCreamPreference"
               GROUP BY "Office"
               ORDER BY "Office"')
AS ("Flavor" varchar(20),
    "Downtown" bigint,
    "Midtown" bigint,
    "Uptown" bigint);
```

**Output:**
```
Flavor      Downtown  Midtown  Uptown
---------   --------  -------  ------
Chocolate   23        41       22
Strawberry  32        null     17
Vanilla     19        23       23
```

This flipped version of the crosstab places flavors on rows and offices in columns, providing an alternative view that might better suit certain analysis needs. The key changes include:

1. Swapping the positions of "Flavor" and "Office" in the first subquery
2. Using "Office" values in the second subquery to define column names
3. Changing the output structure definition to match this new organization

## Summary

This chapter explored advanced SQL techniques that enable more sophisticated data analysis:

1. **Subqueries** allow you to nest queries inside other queries in various contexts:
   - In WHERE clauses to filter based on calculated values
   - In FROM clauses to create derived tables
   - In column lists to add computed values
   - With expressions like IN and EXISTS to test for presence of values

2. **Common Table Expressions (CTEs)** provide a more readable alternative to subqueries:
   - Define temporary tables at the beginning of a query
   - Can be referenced multiple times in the main query
   - Improve code organization and readability

3. **Cross Tabulations** transform data into matrix format:
   - Present relationships between two variables clearly
   - Require PostgreSQL's tablefunc module with the crosstab() function
   - Facilitate visual comparison of data

4. **CASE Expressions** add conditional logic to queries:
   - Allow reclassification of values into categories
   - Can be used in SELECTs, WHERE clauses, and ORDER BY clauses
   - Often combined with aggregation functions for data analysis

These advanced SQL techniques allow for more powerful and flexible data manipulation than basic single-table queries, enabling analysts to derive deeper insights from their data while maintaining query efficiency and readability.

# Try It Yourself

Here are two tasks to help you become more familiar with the concepts introduced in the chapter:

1. Revise the code in Listing 12-15 to dig deeper into the nuances of Waikiki's high temperatures. Limit the temps_collapsed table to the Waikiki maximum daily temperature observations, then use the WHEN clauses in the CASE statement to reclassify the temperatures into seven groups.

   #### PostgreSQL Standard (snake_case):
   ```sql
   WITH waikiki_temps (temperature_group) AS
   (
       SELECT CASE WHEN max_temp >= 90 THEN '90 or more'
                   WHEN max_temp BETWEEN 88 AND 89 THEN '88-89'
                   WHEN max_temp BETWEEN 86 AND 87 THEN '86-87'
                   WHEN max_temp BETWEEN 84 AND 85 THEN '84-85'
                   WHEN max_temp BETWEEN 82 AND 83 THEN '82-83'
                   WHEN max_temp BETWEEN 80 AND 81 THEN '80-81'
                   ELSE '79 or less'
              END
       FROM weather.temperature_readings
       WHERE station_name = 'WAIKIKI 717.2 HI US'
   )
   SELECT temperature_group, count(*)
   FROM waikiki_temps
   GROUP BY temperature_group
   ORDER BY count(*) DESC;
   ```

   #### Domain Driven Design (PascalCase):
   ```sql
   WITH "WaikikiTemperature" ("Category") AS
   (
       SELECT CASE WHEN "MaxTemperature" >= 90 THEN '90 or more'
                   WHEN "MaxTemperature" BETWEEN 88 AND 89 THEN '88-89'
                   WHEN "MaxTemperature" BETWEEN 86 AND 87 THEN '86-87'
                   WHEN "MaxTemperature" BETWEEN 84 AND 85 THEN '84-85'
                   WHEN "MaxTemperature" BETWEEN 82 AND 83 THEN '82-83'
                   WHEN "MaxTemperature" BETWEEN 80 AND 81 THEN '80-81'
                   ELSE '79 or less'
              END
       FROM "Weather"."TemperatureReading"
       WHERE "StationName" = 'WAIKIKI 717.2 HI US'
   )
   SELECT "Category", count(*)
   FROM "WaikikiTemperature"
   GROUP BY "Category"
   ORDER BY count(*) DESC;
   ```

2. Revise the ice cream survey crosstab in Listing 12-11 to flip the table, making flavor the rows and office the columns.

   #### PostgreSQL Standard (snake_case):
   ```sql
   SELECT *
   FROM crosstab('SELECT flavor,
                         office,
                         count(*)
                  FROM survey.ice_cream_survey
                  GROUP BY flavor, office
                  ORDER BY flavor',
                 'SELECT office
                  FROM survey.ice_cream_survey
                  GROUP BY office
                  ORDER BY office')
   AS (flavor varchar(20),
       downtown bigint,
       midtown bigint,
       uptown bigint);
   ```

   #### Domain Driven Design (PascalCase):
   ```sql
   SELECT *
   FROM crosstab('SELECT "Flavor",
                         "Office",
                         count(*)
                  FROM "Survey"."IceCreamPreference"
                  GROUP BY "Flavor", "Office"
                  ORDER BY "Flavor"',
                 'SELECT "Office"
                  FROM "Survey"."IceCreamPreference"
                  GROUP BY "Office"
                  ORDER BY "Office"')
   AS ("Flavor" varchar(20),
       "Downtown" bigint,
       "Midtown" bigint,
       "Uptown" bigint);
   ```

   When flipping a crosstab, you need to change:
   1. The order of columns in the first SELECT statement
   2. The column used in the second subquery to generate column names
   3. The output structure definition (column names and types)