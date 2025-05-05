# Chapter 10 - Statistical Functions in SQL

## Table of Contents

- [Introduction](#introduction)
- [Creating a Census Stats Table](#creating-a-census-stats-table)
- [Measuring Correlation with corr(Y, X)](#measuring-correlation-with-corry-x)
- [Predicting Values with Regression Analysis](#predicting-values-with-regression-analysis)
- [Finding the Effect of an Independent Variable with r-squared](#finding-the-effect-of-an-independent-variable-with-r-squared)
- [Creating Rankings with SQL](#creating-rankings-with-sql)
- [Calculating Rates for Meaningful Comparisons](#calculating-rates-for-meaningful-comparisons)
- [Try It Yourself](#try-it-yourself)

## Introduction

[⬆️ Back to TOC](#table-of-contents)

This chapter explores statistical functions in SQL databases. While specialized statistical software like SPSS, SAS, R, Python, or Excel are often preferred for comprehensive statistical analysis, PostgreSQL offers powerful built-in statistical functions that can provide valuable insights without exporting your data. We'll explore these functions using data from the U.S. Census Bureau and the FBI.

## Creating a Census Stats Table

[⬆️ Back to TOC](#table-of-contents)

We'll begin by creating a table to store county data from the 2011-2015 American Community Survey (ACS) 5-Year Estimates compiled by the U.S. Census Bureau.

### PostgreSQL Standard Convention (snake_case)

```sql
CREATE TABLE census.acs_2011_2015_stats (
    geoid varchar(14) CONSTRAINT census_acs_2011_2015_stats_pky PRIMARY KEY,
    county varchar(50) NOT NULL,
    st varchar(20) NOT NULL,
    pct_travel_60_min numeric(5,3) NOT NULL,
    pct_bachelors_higher numeric(5,3) NOT NULL,
    pct_masters_higher numeric(5,3) NOT NULL,
    median_hh_income integer,
    CHECK (pct_masters_higher <= pct_bachelors_higher)
);

COPY census.acs_2011_2015_stats
FROM 'C:\YourDirectory\acs_2011_2015_stats.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

SELECT * FROM census.acs_2011_2015_stats;
```

### Domain Driven Design Convention for Census Stats Table (PascalCase)

```sql
CREATE TABLE "Census"."AmericanCommunitySurvey" (
    "AmericanCommunitySurveyId" varchar(14) CONSTRAINT "PK_Census_AmericanCommunitySurvey" PRIMARY KEY,
    "GeoId" varchar(14) NOT NULL,
    "County" varchar(50) NOT NULL,
    "State" varchar(20) NOT NULL,
    "PctCommute60Min" numeric(5,3) NOT NULL,
    "PctBachelorsOrHigher" numeric(5,3) NOT NULL,
    "PctMastersOrHigher" numeric(5,3) NOT NULL,
    "MedianHouseholdIncome" integer,
    CHECK ("PctMastersOrHigher" <= "PctBachelorsOrHigher")
);

COPY "Census"."AmericanCommunitySurvey" 
FROM 'C:\YourDirectory\acs_2011_2015_stats.csv'
WITH (FORMAT CSV, HEADER, DELIMITER ',');

SELECT * FROM "Census"."AmericanCommunitySurvey" ;
```

The table contains data for 3,142 counties with information about education, income, and commuting patterns.

## Measuring Correlation with corr(Y, X)

[⬆️ Back to TOC](#table-of-contents)

The `corr(Y, X)` function calculates the Pearson correlation coefficient, which measures the strength of the linear relationship between two variables.

### Interpreting Correlation Coefficients

| Correlation coefficient (+/-) | Relationship strength                 |
|-------------------------------|---------------------------------------|
| 0                             | No relationship                       |
| .01 to .29                    | Weak relationship                     |
| .3 to .59                     | Moderate relationship                 |
| .6 to .99                     | Strong to nearly perfect relationship |
| 1                             | Perfect relationship                  |

Let's analyze the relationship between education level and income:

### PostgreSQL Standard Convention (snake_case)

```sql
SELECT corr(median_hh_income, pct_bachelors_higher)
    AS bachelors_income_r
FROM census.acs_2011_2015_stats;
```

### Domain Driven Design Convention (PascalCase)

```sql
SELECT corr("MedianHouseholdIncome", "PctBachelorsOrHigher")
    AS "EducationIncomeCorrelation"
FROM "Census"."AmericanCommunitySurvey";
```

**Output:**

```markdown
EducationIncomeCorrelation
--------------------------
0.682185675451399
```

The correlation coefficient of approximately 0.68 indicates a strong positive relationship between education and income.

### Checking Additional Correlations

Let's examine other variable relationships:

### PostgreSQL Standard Convention (snake_case)

```sql
SELECT
    round(
        corr(median_hh_income, pct_bachelors_higher)::numeric, 2
    ) AS bachelors_income_r,
    round(
        corr(pct_travel_60_min, median_hh_income)::numeric, 2
    ) AS income_travel_r,
    round(
        corr(pct_travel_60_min, pct_bachelors_higher)::numeric, 2
    ) AS bachelors_travel_r
FROM census.acs_2011_2015_stats;
```

**Output:**

```markdown
bachelors_income_r | income_travel_r | bachelors_travel_r
-------------------|-----------------|-------------------
0.68               | 0.05            | -0.14
```

These results show:

- bachelors_income_r Correlation (0.68): Strong positive relationship
- income_travel_r Correlation (0.05): Practically zero correlation - a county's median income bears little connection to commute time
- bachelors_travel_r Correlation (-0.14): Weak negative relationship - as education increases, the percentage of long commutes slightly decreases

Important caveats:

- Correlation doesn't imply causality
- Statistical significance testing should be performed on correlations (beyond the scope of this chapter)

### Domain Driven Design Convention (PascalCase)

```sql
SELECT
    round(
        corr("MedianHouseholdIncome", "PctBachelorsOrHigher")::numeric, 2
    ) AS "EducationIncomeCorrelation",
    round(
        corr("PctCommute60Min", "MedianHouseholdIncome")::numeric, 2
    ) AS "CommuteIncomeCorrelation",
    round(
        corr("PctCommute60Min", "PctBachelorsOrHigher")::numeric, 2
    ) AS "EducationCommuteCorrelation"
FROM "Census"."AmericanCommunitySurvey";
```

**Output:**

```markdown

EducationIncomeCorrelation | CommuteIncomeCorrelation | EducationCommuteCorrelation
---------------------------|--------------------------|----------------------------
0.68                       | 0.05                     | -0.14

```

These results show:

- Education-Income Correlation (0.68): Strong positive relationship
- Income-Commute Correlation (0.05): Practically zero correlation - a county's median income bears little connection to commute time
- Education-Commute Correlation (-0.14): Weak negative relationship - as education increases, the percentage of long commutes slightly decreases

Important caveats:

- Correlation doesn't imply causality
- Statistical significance testing should be performed on correlations (beyond the scope of this chapter)

## Predicting Values with Regression Analysis

[⬆️ Back to TOC](#table-of-contents)

Linear regression finds the best straight line describing the relationship between variables. The formula is Y = bX + a, where:

- Y is the predicted value (dependent variable)
- b is the slope (how much Y changes per unit of X)
- X is the independent variable
- a is the y-intercept (value of Y when X is zero)

Let's calculate the regression line for education and income:

### PostgreSQL Standard Convention (snake_case)

```sql
SELECT
    round(
        regr_slope(median_hh_income, pct_bachelors_higher)::numeric, 2
    ) AS slope,
    round(
        regr_intercept(median_hh_income, pct_bachelors_higher)::numeric, 2
    ) AS y_intercept
FROM census.acs_2011_2015_stats;
```

**Output:**

```markdown
slope    | y_intercept
---------|------------
926.95   | 27901.15
```

This means:

- For every one percent increase in population with bachelor's degrees, we expect median income to increase by $926.95
- When the percentage with bachelor's degrees is 0, the expected median income is $27,901.15

Using this formula to predict median income for a county where 30% have bachelor's degrees:
Y = 926.95(30) + 27901.15 = $55,709.65

### Domain Driven Design Convention (PascalCase)

```sql
SELECT
    round(
        regr_slope("MedianHouseholdIncome", "PctBachelorsOrHigher")::numeric, 2
    ) AS "RegressionSlope",
    round(
        regr_intercept("MedianHouseholdIncome", "PctBachelorsOrHigher")::numeric, 2
    ) AS "RegressionIntercept"
FROM "Census"."AmericanCommunitySurvey";
```

**Output:**

```markdown

"RegressionSlope" | "RegressionIntercept"
------------------|------------
926.95            | 27901.15
```

This means:

- For every one percent increase in population with bachelor's degrees, we expect median income to increase by $926.95
- When the percentage with bachelor's degrees is 0, the expected median income is $27,901.15

Using this formula to predict median income for a county where 30% have bachelor's degrees:
Y = 926.95(30) + 27901.15 = $55,709.65

## Finding the Effect of an Independent Variable with r-squared

[⬆️ Back to TOC](#table-of-contents)

The coefficient of determination (r-squared) indicates what percentage of variation in the dependent variable is explained by the independent variable.

### PostgreSQL Standard Convention (snake_case)

```sql
SELECT round(
    regr_r2(median_hh_income, pct_bachelors_higher)::numeric, 3
) AS r_squared
FROM census.acs_2011_2015_stats;
```

**Output:**

```markdown
r_squared
---------
0.465
```

The r-squared value of 0.465 indicates that about 47% of the variation in median household income can be explained by the percentage of people with a bachelor's degree or higher. The remaining 53% is influenced by other factors.


### Domain Driven Design Convention (PascalCase)

```sql
SELECT round(
    regr_r2("MedianHouseholdIncome", "PctBachelorsOrHigher")::numeric, 3
) AS "CoefficientOfDetermination"
FROM "Census"."AmericanCommunitySurvey";
```

**Output:**

```markdown
CoefficientOfDetermination
--------------------------
0.465
```

The r-squared value of 0.465 indicates that about 47% of the variation in median household income can be explained by the percentage of people with a bachelor's degree or higher. The remaining 53% is influenced by other factors.

## Creating Rankings with SQL

[⬆️ Back to TOC](#table-of-contents)

SQL provides window functions like `rank()` and `dense_rank()` for creating rankings.

### Ranking with rank() and dense_rank()

Let's examine widget manufacturing companies ranked by output:

### PostgreSQL Standard Convention (snake_case)

```sql
CREATE TABLE manufacturing.widget_companies (
    id bigserial,
    company varchar(30) NOT NULL, 
    widget_output integer NOT NULL
);

INSERT INTO manufacturing.widget_companies (company, widget_output) 
VALUES
    ('Morse Widgets', 125000),
    ('Springfield Widget Masters', 143000), 
    ('Best Widgets', 196000),
    ('Acme Inc.', 133000),
    ('District Widget Inc.', 201000),
    ('Clarke Amalgamated', 620000),
    ('Stavesacre Industries', 244000),
    ('Bowers Widget Emporium', 201000);

SELECT
    company,
    widget_output,
    rank() OVER (ORDER BY widget_output DESC),
    dense_rank() OVER (ORDER BY widget_output DESC) 
FROM manufacturing.widget_companies;
```

**Output:**

```markdown
company                    | widget_output | rank | dense_rank
---------------------------|--------------|------|------------
Clarke Amalgamated         | 620000       | 1    | 1
Stavesacre Industries      | 244000       | 2    | 2
Bowers Widget Emporium     | 201000       | 3    | 3
District Widget Inc.       | 201000       | 3    | 3
Best Widgets               | 196000       | 5    | 4
Springfield Widget Masters | 143000       | 6    | 5
Acme Inc.                  | 133000       | 7    | 6
Morse Widgets              | 125000       | 8    | 7
```

The key difference between `rank()` and `dense_rank()`:

- `rank()` leaves gaps after ties (Best Widgets is ranked 5)
- `dense_rank()` doesn't skip values (Best Widgets is ranked 4)


### Domain Driven Design Convention (PascalCase)

```sql
CREATE TABLE "Manufacturing"."WidgetCompany" (
    "WidgetCompanyId" bigserial,
    "CompanyName" varchar(30) NOT NULL, 
    "WidgetOutput" integer NOT NULL
);

INSERT INTO "Manufacturing"."WidgetCompany" ("CompanyName", "WidgetOutput") 
VALUES
    ('Morse Widgets', 125000),
    ('Springfield Widget Masters', 143000), 
    ('Best Widgets', 196000),
    ('Acme Inc.', 133000),
    ('District Widget Inc.', 201000),
    ('Clarke Amalgamated', 620000),
    ('Stavesacre Industries', 244000),
    ('Bowers Widget Emporium', 201000);

SELECT
    "CompanyName",
    "WidgetOutput",
    rank() OVER (ORDER BY "WidgetOutput" DESC) as Rank,
    dense_rank() OVER (ORDER BY "WidgetOutput" DESC) as DenseRank
FROM "Manufacturing"."WidgetCompany";
```

**Output:**

```markdown
CompanyName                | WidgetOutput | Rank | DenseRank
---------------------------|--------------|------|------------
Clarke Amalgamated         | 620000       | 1    | 1
Stavesacre Industries      | 244000       | 2    | 2
Bowers Widget Emporium     | 201000       | 3    | 3
District Widget Inc.       | 201000       | 3    | 3
Best Widgets               | 196000       | 5    | 4
Springfield Widget Masters | 143000       | 6    | 5
Acme Inc.                  | 133000       | 7    | 6
Morse Widgets              | 125000       | 8    | 7
```

The key difference between `rank()` and `dense_rank()`:

- `rank()` leaves gaps after ties (Best Widgets is ranked 5)
- `dense_rank()` doesn't skip values (Best Widgets is ranked 4)

### Ranking Within Subgroups with PARTITION BY

We can rank within groups using the `PARTITION BY` clause:

### PostgreSQL Standard Convention (snake_case)

```sql
CREATE TABLE retail.store_sales (
    store varchar(30),
    category varchar(30) NOT NULL,
    unit_sales bigint NOT NULL,
    CONSTRAINT retail_store_sales_pky PRIMARY KEY (store, category)
);

INSERT INTO retail.store_sales (store, category, unit_sales) 
VALUES
    ('Broders', 'Cereal', 1104),
    ('Wallace', 'Ice Cream', 1863),
    ('Broders', 'Ice Cream', 2517),
    ('Cramers', 'Ice Cream', 2112),
    ('Broders', 'Beer', 641),
    ('Cramers', 'Cereal', 1003),
    ('Cramers', 'Beer', 640),
    ('Wallace', 'Cereal', 980),
    ('Wallace', 'Beer', 988);

SELECT
    category,
    store,
    unit_sales,
    rank() OVER (PARTITION BY category ORDER BY unit_sales DESC)
FROM retail.store_sales;
```

**Output:**

```markdown
category   | store   | unit_sales | rank
-----------|---------|------------|-----
Beer       | Wallace | 988        | 1
Beer       | Broders | 641        | 2
Beer       | Cramers | 640        | 3
Cereal     | Broders | 1104       | 1
Cereal     | Cramers | 1003       | 2
Cereal     | Wallace | 980        | 3
Ice Cream  | Broders | 2517       | 1
Ice Cream  | Cramers | 2112       | 2
Ice Cream  | Wallace | 1863       | 3
```

This partitioned ranking shows each store's performance within each product category, making it easy to identify top performers by category.

### Domain Driven Design Convention (PascalCase)

```sql
CREATE TABLE "Retail"."StoreSale" (
    "StoreName" varchar(30),
    "Category" varchar(30) NOT NULL,
    "UnitSales" bigint NOT NULL,
    CONSTRAINT "PK_Retail_StoreSale" PRIMARY KEY ("StoreName", "Category")
);

INSERT INTO "Retail"."StoreSale" ("StoreName", "Category", "UnitSales") 
VALUES
    ('Broders', 'Cereal', 1104),
    ('Wallace', 'Ice Cream', 1863),
    ('Broders', 'Ice Cream', 2517),
    ('Cramers', 'Ice Cream', 2112),
    ('Broders', 'Beer', 641),
    ('Cramers', 'Cereal', 1003),
    ('Cramers', 'Beer', 640),
    ('Wallace', 'Cereal', 980),
    ('Wallace', 'Beer', 988);

SELECT
    "Category",
    "StoreName",
    "UnitSales",
    rank() OVER (PARTITION BY "Category" ORDER BY "UnitSales" DESC) as "Rank"
FROM "Retail"."StoreSale";
```

**Output:**

```markdown
Category    | StoreName   | UnitSales | Rank
-----------|---------|------------|-----
Beer       | Wallace | 988        | 1
Beer       | Broders | 641        | 2
Beer       | Cramers | 640        | 3
Cereal     | Broders | 1104       | 1
Cereal     | Cramers | 1003       | 2
Cereal     | Wallace | 980        | 3
Ice Cream  | Broders | 2517       | 1
Ice Cream  | Cramers | 2112       | 2
Ice Cream  | Wallace | 1863       | 3
```

This partitioned ranking shows each store's performance within each product category, making it easy to identify top performers by category.

## Calculating Rates for Meaningful Comparisons

[⬆️ Back to TOC](#table-of-contents)

Raw counts can be misleading when comparing entities of different sizes. Calculating rates per 1,000 population provides more meaningful comparisons.

Let's analyze FBI crime data for cities:

### PostgreSQL Standard Convention (snake_case)

```sql
CREATE TABLE criminal_justice.fbi_crime_data_2015 (
    st varchar(20),
    city varchar(50),
    population integer,
    violent_crime integer,
    property_crime integer,
    burglary integer,
    larceny_theft integer,
    motor_vehicle_theft integer,
    CONSTRAINT criminal_justice_fbi_crime_data_2015_pky PRIMARY KEY (st, city) 
);

COPY criminal_justice.fbi_crime_data_2015
FROM 'C:\YourDirectory\fbi_crime_data_2015.csv' 
WITH (FORMAT CSV, HEADER, DELIMITER ',');

SELECT
    city,
    st,
    population,
    property_crime,
    round(
        (property_crime::numeric / population) * 1000, 1
    ) AS pc_per_1000
FROM criminal_justice.fbi_crime_data_2015
WHERE population >= 500000
ORDER BY (property_crime::numeric / population) DESC;
```

**Output:**

```markdown
city          | st        | population | property_crime | pc_per_1000
--------------|-----------|------------|----------------|------------
Tucson        | Arizona   | 529675     | 35185          | 66.4
San Francisco | California| 863782     | 53019          | 61.4
Albuquerque   | New Mexico| 559721     | 33993          | 60.7
Memphis       | Tennessee | 657936     | 37047          | 56.3
Seattle       | Washington| 683700     | 37754          | 55.2
...
El Paso       | Texas     | 686077     | 13133          | 19.1
New York      | New York  | 8550860    | 129860         | 15.2
```

The results show that although New York City has the highest raw number of property crimes, its rate per 1,000 residents (15.2) is the lowest among large cities. Tucson has the highest rate (66.4) despite having far fewer total crimes.

Note: The FBI discourages using its data to rank localities, as crime rates are influenced by many factors including population density, economic conditions, and climate.

### Domain Driven Design Convention (PascalCase)

```sql
CREATE TABLE "CriminalJustice"."FbiCrimeData" (
    "FbiCrimeDataId" bigserial not null,
    "State" varchar(20),
    "City" varchar(50),
    "Population" integer,
    "ViolentCrime" integer,
    "PropertyCrime" integer,
    "Burglary" integer,
    "LarcenyTheft" integer,
    "MotorVehicleTheft" integer,
    CONSTRAINT "PK_CriminalJustice_FbiCrimeData" PRIMARY KEY ("FbiCrimeDataId"),  
    CONSTRAINT "UQ_CriminalJustice_FbiCrimeData_State_City" UNIQUE ("State", "City")
    );

COPY "CriminalJustice"."FbiCrime-- 
);

COPY "CriminalJustice"."FbiCrimeData"
FROM 'C:\YourDirectory\fbi_crime_data_2015.csv' 
WITH (FORMAT CSV, HEADER, DELIMITER ',');

SELECT
    "City",
    "State",
    "Population",
    "PropertyCrime",
    round(
        ("PropertyCrime"::numeric / "Population") * 1000, 1
    ) AS "PropertyCrimesPer1000"
FROM "CriminalJustice"."FbiCrimeData"
WHERE "Population" >= 500000
ORDER BY ("PropertyCrime"::numeric / "Population") DESC;
```

**Output:**

```markdown

City          | State        | Population | PropertyCrime | PropertyCrimesPer1000
--------------|--------------|------------|---------------|----------------------
Tucson        | Arizona      | 529675     | 35185         | 66.4
San Francisco | California   | 863782     | 53019         | 61.4
Albuquerque   | New Mexico   | 559721     | 33993         | 60.7
Memphis       | Tennessee    | 657936     | 37047         | 56.3
Seattle       | Washington   | 683700     | 37754         | 55.2
...
El Paso       | Texas        | 686077     | 13133         | 19.1
New York      | New York     | 8550860    | 129860        | 15.2
```

The results show that although New York City has the highest raw number of property crimes, its rate per 1,000 residents (15.2) is the lowest among large cities. Tucson has the highest rate (66.4) despite having far fewer total crimes.

Note: The FBI discourages using its data to rank localities, as crime rates are influenced by many factors including population density, economic conditions, and climate.

## Try It Yourself

[⬆️ Back to TOC](#table-of-contents)

Test your new skills with these questions:

1. Write a query to show the correlation between `"PctBachelorsOrHigher"` and `"MedianHouseholdIncome"`. Compare the r value to the correlation between `"PctBachelorsOrHigher"` and `"MedianHouseholdIncome"`. What might explain any difference?

2. Using the FBI crime data, identify which cities with 500,000+ population have the highest rates of motor vehicle theft and violent crime.

3. Bonus challenge: Revisit the libraries data table `"PublicLibrary2014"` from Chapter 8. Rank library agencies based on visit rates per 1,000 population, limiting your analysis to agencies serving 250,000+ people.
