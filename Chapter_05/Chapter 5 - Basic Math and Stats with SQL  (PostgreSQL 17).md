# Basic Math and Stats with SQL

## Math Operators

PostgreSQL and other database systems support various mathematical operations for data analysis. This section explores key operators available in PostgreSQL.

### Basic Math Operators Table

| Operator | Description |
|----------|-------------|
| + | Addition |
| - | Subtraction |
| * | Multiplication |
| / | Division (returns the quotient only, no remainder) |
| % | Modulo (returns just the remainder) |
| ^ | Exponentiation |
| \ |/ | Square root |
| \|\|/ | Cube root |
| ! | Factorial |

### Math and Data Types

The data type returned by mathematical operations follows these patterns:

- Two integers return an integer
- A numeric on either side of the operator returns a numeric
- Anything with a floating-point number returns a floating-point number of type double precision

Exponentiation, root, and factorial functions return numeric and floating-point types, even when the input is an integer.

### Adding, Subtracting, and Multiplying

#### PostgreSQL Standard (snake_case)

```sql
SELECT 2 + 2;
SELECT 9 - 1;
SELECT 3 * 4;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 2 + 2 AS "Sum";
SELECT 9 - 1 AS "Difference";
SELECT 3 * 4 AS "Product";
```

**Sample output:**
```
?column?
---------
4

?column?
---------
8

?column?
---------
12
```

These examples demonstrate basic arithmetic operations in SQL without needing a table reference, useful for testing or simple calculations.

### Division and Modulo

#### PostgreSQL Standard (snake_case)

```sql
SELECT 11 / 6;          -- Integer division
SELECT 11 % 6;          -- Modulo operation
SELECT 11.0 / 6;        -- Decimal division
SELECT CAST(11 AS numeric(3,1)) / 6;  -- Using cast
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 11 / 6 AS "IntegerQuotient";
SELECT 11 % 6 AS "Remainder";
SELECT 11.0 / 6 AS "DecimalQuotient";
SELECT CAST(11 AS numeric(3,1)) / 6 AS "CastQuotient";
```

**Sample output:**
```
?column?
---------
1        -- Integer division returns only quotient

?column?
---------
5        -- Modulo returns only the remainder

?column?
---------
1.83333  -- Using decimal forces decimal division

?column?
---------
1.83333  -- Cast forces decimal division
```

Division behavior depends on the data types involved. Integer division returns only the quotient, while using a numeric type provides decimal results. The modulo operator (%) is useful for finding remainders and testing conditions like checking if a number is even.

### Exponents, Roots, and Factorials

#### PostgreSQL Standard (snake_case)

```sql
SELECT 3 ^ 4;       -- Exponentiation
SELECT |/ 10;       -- Square root
SELECT sqrt(10);    -- Square root (function form)
SELECT ||/ 10;      -- Cube root
SELECT 4 !;         -- Factorial
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 3 ^ 4 AS "Exponent";
SELECT |/ 10 AS "SquareRoot";
SELECT sqrt(10) AS "SqrtFunction";
SELECT ||/ 10 AS "CubeRoot";
SELECT 4 ! AS "Factorial";
```

**Sample output:**
```
?column?
---------
81       -- 3 to the 4th power

?column?
---------
3.16227  -- Square root of 10

?column?
---------
3.16227  -- Square root using function

?column?
---------
2.15443  -- Cube root of 10

?column?
---------
24       -- 4! = 4×3×2×1
```

These operators are PostgreSQL-specific and provide powerful mathematical functionality. Factorials are useful for calculating permutations (such as determining how many ways items can be ordered).

### Minding the Order of Operations

SQL follows standard mathematical order of operations:
1. Exponents and roots
2. Multiplication, division, modulo
3. Addition and subtraction

#### PostgreSQL Standard (snake_case)

```sql
SELECT 7 + 8 * 9;      -- Multiplication before addition
SELECT (7 + 8) * 9;    -- Parentheses force addition first

SELECT 3 ^ 3 - 1;      -- Exponentiation before subtraction
SELECT 3 ^ (3 - 1);    -- Parentheses force subtraction first
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 7 + 8 * 9 AS "StandardPrecedence";
SELECT (7 + 8) * 9 AS "ParenthesisFirst";

SELECT 3 ^ 3 - 1 AS "ExponentFirst";
SELECT 3 ^ (3 - 1) AS "SubtractionFirst";
```

**Sample output:**
```
?column?
---------
79       -- 8*9=72, then 7+72=79

?column?
---------
135      -- 7+8=15, then 15*9=135

?column?
---------
26       -- 3^3=27, then 27-1=26

?column?
---------
9        -- 3-1=2, then 3^2=9
```

Understanding operator precedence is crucial for writing accurate mathematical expressions in SQL.

## Doing Math Across Census Table Columns

Let's apply mathematical operations to our census data table.

### Adding and Subtracting Columns

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    geo_name,
    state_us_abbreviation AS "st",
    p0010003 AS "White Alone",
    p0010004 AS "Black Alone",
    p0010003 + p0010004 AS "Total White and Black"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    geo_name AS "GeoName",
    state_us_abbreviation AS "State",
    p0010003 AS "WhiteAlone",
    p0010004 AS "BlackAlone",
    p0010003 + p0010004 AS "TotalWhiteAndBlack"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
geo_name         st   White Alone   Black Alone   Total White and Black
----------------  ---  ------------  -----------  ---------------------
Autauga County    AL   42859         9643         52498
Baldwin County    AL   156153        17105        173258
Barbour County    AL   13180         12875        26055
```

We can also check data integrity by testing if all race columns add up to the total population:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    geo_name,
    state_us_abbreviation AS "st",
    p0010001 AS "Total",
    p0010003 + p0010004 + p0010005 + p0010006 + p0010007 + p0010008 + p0010009 AS "All Races",
    (p0010003 + p0010004 + p0010005 + p0010006 + p0010007 + p0010008 + p0010009) - p0010001 AS "Difference"
FROM census.us_counties_2010
ORDER BY "Difference" DESC;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    geo_name AS "GeoName",
    state_us_abbreviation AS "State",
    p0010001 AS "TotalPopulation",
    p0010003 + p0010004 + p0010005 + p0010006 + p0010007 + p0010008 + p0010009 AS "AllRaces",
    (p0010003 + p0010004 + p0010005 + p0010006 + p0010007 + p0010008 + p0010009) - p0010001 AS "Difference"
FROM "Census"."USCounties2010"
ORDER BY "Difference" DESC;
```

**Sample output:**
```
geo_name         st   Total     All Races    Difference
----------------  ---  --------  -----------  ----------
Autauga County    AL   54571    54571        0
Baldwin County    AL   182265   182265       0
Barbour County    AL   27457    27457        0
```

With the Difference column showing zeros, we can confirm our data import was accurate. This validation is an important step when working with new datasets.

### Finding Percentages of the Whole

Calculating percentages helps identify meaningful population demographics differences across counties.

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    geo_name,
    state_us_abbreviation AS "st",
    (CAST(p0010006 AS numeric(8,1)) / p0010001) * 100 AS "pct_asian"
FROM census.us_counties_2010
ORDER BY "pct_asian" DESC;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    geo_name AS "GeoName",
    state_us_abbreviation AS "State",
    (CAST(p0010006 AS numeric(8,1)) / p0010001) * 100 AS "PercentAsian"
FROM "Census"."USCounties2010"
ORDER BY "PercentAsian" DESC;
```

**Sample output:**
```
geo_name             st   pct_asian
--------------------  ---  ---------
Honolulu County       HI   43.89
Aleutians East Borough AK   35.98
San Francisco County  CA   33.27
Santa Clara County    CA   32.02
Kauai County          HI   31.32
Aleutians West Census Area AK 28.88
```

We use CAST to ensure decimal division and multiply by 100 to express the result as a percentage. This technique allows for demographic comparison across different counties.

### Tracking Percent Change

Percent change calculations are valuable for analyzing changes over time.

#### PostgreSQL Standard (snake_case)

```sql
CREATE TABLE percent_change (
    department varchar(20),
    spend_2014 numeric(10,2),
    spend_2017 numeric(10,2)
);

INSERT INTO percent_change
VALUES
    ('Building', 250000, 289000),
    ('Assessor', 178556, 179500),
    ('Library', 87777, 90001),
    ('Clerk', 451980, 650000),
    ('Police', 250000, 223000),
    ('Recreation', 199000, 195000);

SELECT 
    department,
    spend_2014,
    spend_2017,
    round((spend_2017 - spend_2014) / spend_2014 * 100, 1) AS "pct_change"
FROM finance.percent_change;
```

#### Domain Driven Design (PascalCase)

```sql
CREATE TABLE "Finance"."PercentChange" (
    "Department" varchar(20),
    "Spend2014" numeric(10,2),
    "Spend2017" numeric(10,2)
);

INSERT INTO "Finance"."PercentChange"
VALUES
    ('Building', 250000, 289000),
    ('Assessor', 178556, 179500),
    ('Library', 87777, 90001),
    ('Clerk', 451980, 650000),
    ('Police', 250000, 223000),
    ('Recreation', 199000, 195000);

SELECT 
    "Department",
    "Spend2014",
    "Spend2017",
    round(("Spend2017" - "Spend2014") / "Spend2014" * 100, 1) AS "PercentChange"
FROM "Finance"."PercentChange";
```

**Sample output:**
```
department    spend_2014    spend_2017    pct_change
------------  -----------   -----------   ----------
Building      250000.00     289000.00     15.6
Assessor      178556.00     179500.00     0.5
Library       87777.00      90001.00      2.5
Clerk         451980.00     650000.00     43.8
Police        250000.00     223000.00     -10.8
Recreation    199000.00     195000.00     -2.0
```

This formula ((new - old) / old * 100) calculates percent change between two time periods, showing which departments experienced growth or reduction in spending.

## Aggregate Functions for Averages and Sums

Aggregate functions allow calculations across multiple rows to produce a single result.

### Using sum() and avg()

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    sum(p0010001) AS "County Sum",
    round(avg(p0010001), 0) AS "County Average"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    sum(p0010001) AS "CountySum",
    round(avg(p0010001), 0) AS "CountyAverage"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
County Sum      County Average
--------------  --------------
308745538       98233
```

This shows the total US population across all counties in 2010 was approximately 308.7 million, with an average county population of 98,233.

## Finding the Median

The median is often a better representation of central tendency than the average, especially with skewed data.

### Finding the Median with Percentile Functions

SQL doesn't have a direct median() function, but we can use percentile functions to calculate it.

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "County Median"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "CountyMedian"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
County Median
------------
25857
```

We can also compare with the average to see data distribution characteristics:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    sum(p0010001) AS "County Sum",
    round(avg(p0010001), 0) AS "County Average",
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "County Median"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    sum(p0010001) AS "CountySum",
    round(avg(p0010001), 0) AS "CountyAverage",
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "CountyMedian"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
County Sum      County Average    County Median
--------------  --------------    -------------
308745538       98233             25857
```

The substantial difference between average (98,233) and median (25,857) indicates the data is skewed, with some very populous counties pulling up the average.

### Median and Percentiles with Census Data

Let's demonstrate the difference between percentile_cont() and percentile_disc():

#### PostgreSQL Standard (snake_case)

```sql
CREATE TABLE percentile_test (
    numbers integer
);

INSERT INTO percentile_test (numbers) VALUES
    (1), (2), (3), (4), (5), (6);

SELECT
    percentile_cont(.5) WITHIN GROUP (ORDER BY numbers),
    percentile_disc(.5) WITHIN GROUP (ORDER BY numbers)
FROM analytics.percentile_test;
```

#### Domain Driven Design (PascalCase)

```sql
CREATE TABLE "Analytics"."PercentileTest" (
    "Number" integer
);

INSERT INTO "Analytics"."PercentileTest" ("Number") VALUES
    (1), (2), (3), (4), (5), (6);

SELECT
    percentile_cont(.5) WITHIN GROUP (ORDER BY "Number") AS "ContinuousPercentile",
    percentile_disc(.5) WITHIN GROUP (ORDER BY "Number") AS "DiscretePercentile"
FROM "Analytics"."PercentileTest";
```

**Sample output:**
```
percentile_cont    percentile_disc
---------------    ---------------
3.5                3
```

- percentile_cont() returns continuous values, averaging the middle values when needed (traditional median calculation)
- percentile_disc() returns discrete values, taking the first value that falls within the percentile

### Finding Other Quantiles with Percentile Functions

We can find quartiles, quintiles, or deciles using arrays with percentile functions:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    percentile_cont(array[.25,.5,.75]) WITHIN GROUP (ORDER BY p0010001) AS "quartiles"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    percentile_cont(array[.25,.5,.75]) WITHIN GROUP (ORDER BY p0010001) AS "Quartiles"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
quartiles
--------------------------------
{11104.5,25857,66699}
```

To make this output more readable, we can use the unnest() function:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    unnest(percentile_cont(array[.25,.5,.75]) WITHIN GROUP (ORDER BY p0010001)) AS "quartiles"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    unnest(percentile_cont(array[.25,.5,.75]) WITHIN GROUP (ORDER BY p0010001)) AS "Quartiles"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
quartiles
---------
11104.5
25857
66699
```

### Creating a median() Function

While PostgreSQL doesn't have a built-in median() function, we can create one:

#### PostgreSQL Standard (snake_case)

```sql
CREATE OR REPLACE FUNCTION _final_median(anyarray)
RETURNS float8 AS
$$
WITH q AS
(
    SELECT val
    FROM unnest($1) val
    WHERE val IS NOT NULL
    ORDER BY 1
),
cnt AS
(
    SELECT COUNT(*) AS c FROM q
)
SELECT AVG(val)::float8
FROM
(
    SELECT val FROM q
    LIMIT 2 - MOD((SELECT c FROM cnt), 2)
    OFFSET GREATEST(CEIL((SELECT c FROM cnt) / 2.0) - 1,0) 
) q2;
$$
LANGUAGE sql IMMUTABLE;

CREATE AGGREGATE median(anyelement) (
    SFUNC=array_append,
    STYPE=anyarray,
    FINALFUNC=_final_median,
    INITCOND='{}'
);
```

#### Domain Driven Design (PascalCase)

```sql
CREATE OR REPLACE FUNCTION "Statistics"."_FinalMedian"(anyarray)
RETURNS float8 AS
$$
WITH q AS
(
    SELECT val
    FROM unnest($1) val
    WHERE val IS NOT NULL
    ORDER BY 1
),
cnt AS
(
    SELECT COUNT(*) AS c FROM q
)
SELECT AVG(val)::float8
FROM
(
    SELECT val FROM q
    LIMIT 2 - MOD((SELECT c FROM cnt), 2)
    OFFSET GREATEST(CEIL((SELECT c FROM cnt) / 2.0) - 1,0) 
) q2;
$$
LANGUAGE sql IMMUTABLE;

CREATE AGGREGATE "Statistics"."Median"(anyelement) (
    SFUNC=array_append,
    STYPE=anyarray,
    FINALFUNC="Statistics"."_FinalMedian",
    INITCOND='{}'
);
```

Using this custom function:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    sum(p0010001) AS "County Sum",
    round(avg(p0010001), 0) AS "County Average",
    median(p0010001) AS "County Median",
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "50th Percentile"
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    sum(p0010001) AS "CountySum",
    round(avg(p0010001), 0) AS "CountyAverage",
    "Statistics"."Median"(p0010001) AS "CountyMedian",
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "FiftiethPercentile"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
County Sum      County Average    County Median    50th Percentile
--------------  --------------    -------------    ---------------
308745538       98233             25857            25857
```

This confirms that our median() function and percentile_cont(.5) return the same value.

## Finding the Mode

The mode is the value that appears most frequently in a data set. PostgreSQL provides a mode() function:

#### PostgreSQL Standard (snake_case)

```sql
SELECT 
    mode() WITHIN GROUP (ORDER BY p0010001)
FROM census.us_counties_2010;
```

#### Domain Driven Design (PascalCase)

```sql
SELECT 
    mode() WITHIN GROUP (ORDER BY p0010001) AS "MostFrequentValue"
FROM "Census"."USCounties2010";
```

**Sample output:**
```
?column?
--------
21720
```

This tells us that 21,720 is the most frequent population value among counties, appearing in counties across Mississippi, Oregon, and West Virginia.

## Wrapping Up

Mathematical operations and statistical functions in SQL allow for powerful data analysis directly within the database. From basic arithmetic to complex statistical calculations, these tools enable analysts to derive meaningful insights from their data. Understanding the differences between average, median, and mode helps ensure accurate interpretation of data distributions, especially when working with skewed datasets like population figures.

## Try It Yourself

1. Write a SQL statement for calculating the area of a circle whose radius is 5 inches.

   #### PostgreSQL Standard (snake_case)
   ```sql
   SELECT 3.14159 * 5 * 5 AS area_of_circle;
   -- or using pi() function if available
   SELECT pi() * power(5, 2) AS area_of_circle;
   ```

   #### Domain Driven Design (PascalCase)
   ```sql
   SELECT 3.14159 * 5 * 5 AS "AreaOfCircle";
   -- or using pi() function if available
   SELECT pi() * power(5, 2) AS "AreaOfCircle";
   ```

2. Find the New York county with the highest percentage of American Indian/Alaska Native Alone population.

   #### PostgreSQL Standard (snake_case)
   ```sql
   SELECT 
       geo_name,
       (CAST(p0010005 AS numeric(8,1)) / p0010001) * 100 AS pct_native
   FROM census.us_counties_2010
   WHERE state_us_abbreviation = 'NY'
   ORDER BY pct_native DESC
   LIMIT 1;
   ```

   #### Domain Driven Design (PascalCase)
   ```sql
   SELECT 
       geo_name AS "GeoName",
       (CAST(p0010005 AS numeric(8,1)) / p0010001) * 100 AS "PercentNative"
   FROM "Census"."USCounties2010"
   WHERE state_us_abbreviation = 'NY'
   ORDER BY "PercentNative" DESC
   LIMIT 1;
   ```

3. Determine which state had the higher 2010 median county population: California or New York.

   #### PostgreSQL Standard (snake_case)
   ```sql
   SELECT 
       state_us_abbreviation,
       percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS median_pop
   FROM census.us_counties_2010
   WHERE state_us_abbreviation IN ('CA', 'NY')
   GROUP BY state_us_abbreviation
   ORDER BY median_pop DESC;
   ```

   #### Domain Driven Design (PascalCase)

   ```sql
   SELECT 
       state_us_abbreviation AS "State",
       percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "MedianPopulation"
   FROM "Census"."USCounties2010"
   WHERE state_us_abbreviation IN ('CA', 'NY')
   GROUP BY "State"
   ORDER BY "MedianPopulation" DESC;
   ```

   #### Domain Driven Design (PascalCase)
```sql
SELECT 
    state_us_abbreviation AS "State",
    percentile_cont(.5) WITHIN GROUP (ORDER BY p0010001) AS "MedianPopulation"
FROM "Census"."USCounties2010"
WHERE state_us_abbreviation IN ('CA', 'NY')
GROUP BY "State"
ORDER BY "MedianPopulation" DESC;
```

**Sample output:**
```
State    MedianPopulation
-------  ----------------
CA       179140.5
NY       117747.0
```

This analysis shows that California had a higher median county population than New York in 2010. This means that a typical California county had more residents than a typical New York county, providing insight into the population distribution patterns within these states.

The result reflects the states' different geographic and demographic structures - California's counties tend to be more populous on average, while New York has a mix of highly populated urban counties and more sparsely populated rural ones.

This demonstrates how SQL's statistical functions can provide meaningful demographic comparisons between different regions, enabling data-driven policy and planning decisions.