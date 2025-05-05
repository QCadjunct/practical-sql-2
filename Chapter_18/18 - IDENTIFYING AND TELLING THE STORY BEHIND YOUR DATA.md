# Chapter 18 - IDENTIFYING AND TELLING THE STORY BEHIND YOUR DATA

## Introduction

Chapter 18 explores how to effectively identify trends and create meaningful narratives from data analysis using SQL. The author emphasizes that SQL is not just a technical skill but a tool for uncovering stories within data that can lead to meaningful insights and decisions. This chapter outlines a process used by the author as an investigative journalist to discover stories in data and effectively communicate findings.

## Start with a Question

A good data analysis begins with curiosity and thoughtful questions. The process often starts with:

1. Observing changes or patterns in your surroundings
2. Formulating specific questions about these observations
3. Prioritizing questions based on potential value

**Example - Real Estate Market Analysis:**
```sql
-- PostgreSQL Standard (snake_case)
SELECT 
    neighborhood,
    COUNT(*) AS homes_sold,
    AVG(sale_price) AS average_price,
    EXTRACT(YEAR FROM sale_date) AS sale_year
FROM real_estate.property_sales
WHERE sale_date BETWEEN '2022-01-01' AND '2023-12-31'
GROUP BY neighborhood, EXTRACT(YEAR FROM sale_date)
ORDER BY neighborhood, sale_year;

-- Domain Driven Design (PascalCase)
SELECT 
    "Neighborhood",
    COUNT(*) AS "HomesSold",
    AVG("SalePrice") AS "AveragePrice",
    EXTRACT(YEAR FROM "SaleDate") AS "SaleYear"
FROM "RealEstate"."PropertySale"
WHERE "SaleDate" BETWEEN '2022-01-01' AND '2023-12-31'
GROUP BY "Neighborhood", EXTRACT(YEAR FROM "SaleDate")
ORDER BY "Neighborhood", "SaleYear";
```

This analysis might reveal whether there has been a dramatic increase in home sales in specific neighborhoods compared to previous periods.

## Document Your Process

Transparency and reproducibility are critical for credibility. The author recommends:

- Creating detailed step-by-step documentation
- Storing notes and code in text files or version control systems
- Building a consistent documentation system

**Documentation Example:**
```sql
-- PostgreSQL Standard (snake_case)
-- Purpose: Analyze home sale trends by neighborhood
-- Created: 2023-06-01 by Data Analyst
-- Data Source: real_estate.property_sales (2020-2023)
-- Notes: Excluded foreclosures and non-residential properties

-- Domain Driven Design (PascalCase)
-- Purpose: Analyze home sale trends by neighborhood
-- Created: 2023-06-01 by Data Analyst
-- Data Source: "RealEstate"."PropertySale" (2020-2023)
-- Notes: Excluded foreclosures and non-residential properties
```

## Gather Your Data

Finding relevant data involves:

1. Identifying internal data sources (if available)
2. Consulting experts for data source recommendations
3. Exploring government data portals (e.g., data.gov)
4. Examining local government websites for structured data
5. Analyzing data over extended timeframes when possible (5+ years)

**Data Collection Query Example:**
```sql
-- PostgreSQL Standard (snake_case)
-- Collecting data from multiple government sources
CREATE TABLE analysis.home_sales_combined AS
SELECT * FROM fed_data.property_transactions
UNION ALL
SELECT * FROM state_data.real_estate_transfers
UNION ALL
SELECT * FROM county_data.deed_records
WHERE transaction_date BETWEEN '2018-01-01' AND '2023-12-31';

-- Domain Driven Design (PascalCase)
-- Collecting data from multiple government sources
CREATE TABLE "Analysis"."HomeSalesCombined" AS
SELECT * FROM "FederalData"."PropertyTransaction"
UNION ALL
SELECT * FROM "StateData"."RealEstateTransfer"
UNION ALL
SELECT * FROM "CountyData"."DeedRecord"
WHERE "TransactionDate" BETWEEN '2018-01-01' AND '2023-12-31';
```

### No Data? Build Your Own Database

When data isn't readily available, creating your own structured database is an option:

1. Research relevant sources (news articles, reports, public records)
2. Identify key data points to track
3. Create tables with appropriate fields
4. Systematically collect and enter data

The author provides a real-world example where he built a database tracking student deaths on college campuses, which revealed freshmen were particularly vulnerable.

**Example - Building a Custom Database:**
```sql
-- PostgreSQL Standard (snake_case)
CREATE TABLE education.student_incidents (
    student_id SERIAL PRIMARY KEY,
    age INTEGER NOT NULL,
    school VARCHAR(100) NOT NULL,
    incident_type VARCHAR(50) NOT NULL,
    incident_date DATE NOT NULL,
    year_in_school VARCHAR(20),
    alcohol_involved BOOLEAN,
    drugs_involved BOOLEAN,
    notes TEXT
);

-- Domain Driven Design (PascalCase)
CREATE TABLE "Education"."StudentIncident" (
    "StudentId" SERIAL PRIMARY KEY,
    "Age" INTEGER NOT NULL,
    "School" VARCHAR(100) NOT NULL,
    "IncidentType" VARCHAR(50) NOT NULL,
    "IncidentDate" DATE NOT NULL,
    "YearInSchool" VARCHAR(20),
    "AlcoholInvolved" BOOLEAN,
    "DrugsInvolved" BOOLEAN,
    "Notes" TEXT
);
```

## Assess the Data's Origins

Understanding how data is collected and maintained is crucial for:

1. Evaluating credibility and standardization
2. Understanding inconsistencies (like multiple spellings of company names)
3. Interpreting time-based data correctly
4. Accounting for collection methods in your analysis

**Example - Data Origin Check Query:**
```sql
-- PostgreSQL Standard (snake_case)
-- Identify potential data collection issues
SELECT 
    company_name,
    COUNT(*) AS frequency
FROM agriculture.food_producers
GROUP BY company_name
HAVING COUNT(*) > 1
ORDER BY frequency DESC, company_name;

-- Domain Driven Design (PascalCase)
-- Identify potential data collection issues
SELECT 
    "CompanyName",
    COUNT(*) AS "Frequency"
FROM "Agriculture"."FoodProducer"
GROUP BY "CompanyName"
HAVING COUNT(*) > 1
ORDER BY "Frequency" DESC, "CompanyName";
```

## Interview the Data with Queries

The author recommends "interviewing" your data by exploring it with queries that:

1. Calculate aggregates (counts, sums, averages)
2. Identify minimum and maximum values
3. Check for duplicate entries
4. Test joins between related tables
5. Document questions and concerns as they arise

**Example - Data Interview Queries:**
```sql
-- PostgreSQL Standard (snake_case)
-- Basic data interview
SELECT 
    COUNT(*) AS total_records,
    COUNT(DISTINCT tripid) AS unique_trips,
    MIN(trip_start_time) AS earliest_trip,
    MAX(trip_start_time) AS latest_trip,
    AVG(trip_duration) AS avg_duration_seconds
FROM transportation.nyc_taxi_trips;

-- Check for orphaned records
SELECT COUNT(*) 
FROM transportation.taxi_payments p
LEFT JOIN transportation.nyc_taxi_trips t ON p.tripid = t.tripid
WHERE t.tripid IS NULL;

-- Domain Driven Design (PascalCase)
-- Basic data interview
SELECT 
    COUNT(*) AS "TotalRecords",
    COUNT(DISTINCT "TripId") AS "UniqueTrips",
    MIN("TripStartTime") AS "EarliestTrip",
    MAX("TripStartTime") AS "LatestTrip",
    AVG("TripDuration") AS "AvgDurationSeconds"
FROM "Transportation"."NycTaxiTrip";

-- Check for orphaned records
SELECT COUNT(*) 
FROM "Transportation"."TaxiPayment" p
LEFT JOIN "Transportation"."NycTaxiTrip" t ON p."TripId" = t."TripId"
WHERE t."TripId" IS NULL;
```

## Consult the Data's Owner

After initial exploration, consulting with someone who knows the data well helps to:

1. Clarify understanding of the data
2. Verify initial findings
3. Discover potential issues with the data
4. Understand data limitations and completeness
5. Confirm suitability for your analysis needs

## Identify Key Indicators and Trends over Time

With a clear understanding of the data, the next step is to:

1. Choose specific indicators to track
2. Analyze how these indicators change over multiple time periods

**Example - Trend Analysis Query:**
```sql
-- PostgreSQL Standard (snake_case)
-- Analyze enrollment trends over time
SELECT
    academic_year,
    COUNT(*) AS total_enrollments,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY academic_year) AS yearly_change,
    ROUND(
        (COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY academic_year))::numeric / 
        NULLIF(LAG(COUNT(*)) OVER (ORDER BY academic_year), 0) * 100, 
    2) AS percent_change
FROM education.student_enrollments
WHERE academic_year BETWEEN '2015-2016' AND '2022-2023'
GROUP BY academic_year
ORDER BY academic_year;

-- Domain Driven Design (PascalCase)
-- Analyze enrollment trends over time
SELECT
    "AcademicYear",
    COUNT(*) AS "TotalEnrollments",
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY "AcademicYear") AS "YearlyChange",
    ROUND(
        (COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY "AcademicYear"))::numeric / 
        NULLIF(LAG(COUNT(*)) OVER (ORDER BY "AcademicYear"), 0) * 100, 
    2) AS "PercentChange"
FROM "Education"."StudentEnrollment"
WHERE "AcademicYear" BETWEEN '2015-2016' AND '2022-2023'
GROUP BY "AcademicYear"
ORDER BY "AcademicYear";
```

The author emphasizes the importance of:
- Looking at both short-term changes and long-term trends
- Contextualizing shorter-term changes within longer histories
- Testing for statistical significance when working with sample data

## Ask Why

Data analysis reveals what happened but not why it happened. To understand the driving forces:

1. Revisit the data with subject matter experts
2. Ask them to verify or question your findings
3. Request their perspectives on causation
4. Incorporate their insights into your final presentation

## Communicate Your Findings

Effective communication of findings depends on the audience and context. The author recommends:

1. Identify an overarching theme for your presentation
2. Present overall numbers to show general trends
3. Highlight specific examples supporting the trend
4. Acknowledge counter-examples that don't fit the pattern
5. Stick to factual statements without distortion
6. Include expert opinions for context
7. Use visual elements (charts, graphs) to illustrate findings
8. Cite all data sources with appropriate qualifications
9. Share your data and methodology for transparency

**Example - Final Presentation Query:**
```sql
-- PostgreSQL Standard (snake_case)
-- Generate data for final presentation on neighborhood home sales
WITH yearly_stats AS (
    SELECT
        neighborhood,
        EXTRACT(YEAR FROM sale_date) AS year,
        COUNT(*) AS sales_count,
        ROUND(AVG(sale_price)::numeric, 2) AS avg_price,
        ROUND(
            (COUNT(*) - LAG(COUNT(*)) OVER (PARTITION BY neighborhood ORDER BY EXTRACT(YEAR FROM sale_date)))::numeric /
            NULLIF(LAG(COUNT(*)) OVER (PARTITION BY neighborhood ORDER BY EXTRACT(YEAR FROM sale_date)), 0) * 100,
        2) AS sales_growth
    FROM real_estate.property_sales
    WHERE sale_date BETWEEN '2018-01-01' AND '2023-12-31'
    GROUP BY neighborhood, EXTRACT(YEAR FROM sale_date)
)
SELECT
    neighborhood,
    year,
    sales_count,
    avg_price,
    sales_growth,
    CASE
        WHEN neighborhood_type = 'suburban' AND sales_growth > 0 THEN 'Supports trend'
        WHEN neighborhood_type = 'urban' AND sales_growth < 0 THEN 'Supports trend'
        ELSE 'Counter to trend'
    END AS trend_alignment
FROM yearly_stats ys
JOIN real_estate.neighborhood_types nt USING (neighborhood)
ORDER BY trend_alignment, ABS(sales_growth) DESC;

-- Domain Driven Design (PascalCase)
-- Generate data for final presentation on neighborhood home sales
WITH "YearlyStats" AS (
    SELECT
        "Neighborhood",
        EXTRACT(YEAR FROM "SaleDate") AS "Year",
        COUNT(*) AS "SalesCount",
        ROUND(AVG("SalePrice")::numeric, 2) AS "AvgPrice",
        ROUND(
            (COUNT(*) - LAG(COUNT(*)) OVER (PARTITION BY "Neighborhood" ORDER BY EXTRACT(YEAR FROM "SaleDate")))::numeric /
            NULLIF(LAG(COUNT(*)) OVER (PARTITION BY "Neighborhood" ORDER BY EXTRACT(YEAR FROM "SaleDate")), 0) * 100,
        2) AS "SalesGrowth"
    FROM "RealEstate"."PropertySale"
    WHERE "SaleDate" BETWEEN '2018-01-01' AND '2023-12-31'
    GROUP BY "Neighborhood", EXTRACT(YEAR FROM "SaleDate")
)
SELECT
    "Neighborhood",
    "Year",
    "SalesCount",
    "AvgPrice",
    "SalesGrowth",
    CASE
        WHEN "NeighborhoodType" = 'suburban' AND "SalesGrowth" > 0 THEN 'Supports trend'
        WHEN "NeighborhoodType" = 'urban' AND "SalesGrowth" < 0 THEN 'Supports trend'
        ELSE 'Counter to trend'
    END AS "TrendAlignment"
FROM "YearlyStats" ys
JOIN "RealEstate"."NeighborhoodType" nt USING ("Neighborhood")
ORDER BY "TrendAlignment", ABS("SalesGrowth") DESC;
```

## Try It Yourself

The chapter concludes with a hands-on exercise encouraging readers to:
1. Choose a local or national topic of interest
2. Search for available data on the topic
3. Assess the data's quality and timeliness
4. Consult with a subject matter expert
5. Load the data into PostgreSQL
6. Interview the data using aggregate queries
7. Identify meaningful trends
8. Present findings in a concise, clear format

## Key Takeaways

The process of data analysis is iterative and requires:
1. Beginning with thoughtful questions
2. Gathering high-quality data from reliable sources
3. Understanding data origins and limitations
4. Exploring data systematically with SQL queries
5. Verifying findings with subject matter experts
6. Identifying key trends over time
7. Communicating findings effectively with a clear narrative
8. Practicing transparency by sharing data and methodology

This structured approach to data analysis helps transform raw data into meaningful insights that can inform decisions or tell important stories.