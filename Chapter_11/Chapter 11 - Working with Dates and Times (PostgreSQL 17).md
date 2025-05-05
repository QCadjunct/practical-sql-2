# Chapter 11 - Working with Dates and Times (PostgreSQL 17)

## Introduction to Date and Time Types

PostgreSQL provides a rich set of data types for working with dates and times. These types enable efficient storage and manipulation of temporal data with varying levels of precision.

### Date and Time Data Types

PostgreSQL offers several specialized data types for handling temporal information:

- **date**: Stores dates only (year, month, day)
- **time**: Stores time of day without timezone information
- **timetz**: Stores time of day with timezone information
- **timestamp**: Stores both date and time without timezone
- **timestamptz**: Stores both date and time with timezone (recommended for most applications)
- **interval**: Stores time periods/durations

These types provide different levels of precision and timezone awareness for varying application requirements:

```sql
-- Creating a table with various date/time columns
CREATE TABLE time_management.event_calendar (
    event_id SERIAL PRIMARY KEY,
    event_date date,
    event_time time,
    event_time_tz timetz,
    event_timestamp timestamp,
    event_timestamp_tz timestamptz,
    event_duration interval
);

-- Example in Domain Driven Design convention
CREATE TABLE "TimeManagement"."EventCalendar" (
    "EventId" SERIAL PRIMARY KEY,
    "EventDate" date,
    "EventTime" time,
    "EventTimeWithTZ" timetz, 
    "EventTimestamp" timestamp,
    "EventTimestampWithTZ" timestamptz,
    "EventDuration" interval
);
```

The `timestamptz` type is generally recommended for most applications as it stores timestamp values in UTC and converts them to the client's timezone for display.

## Date and Time Input

PostgreSQL accepts various formats for entering date and time values to accommodate different user preferences and regional conventions.

### Date Input

Dates can be entered in several formats, with ISO 8601 (YYYY-MM-DD) being the recommended standard:

```sql
-- Different ways to input dates
INSERT INTO time_management.event_calendar (event_date) VALUES 
('2023-11-15'),        -- ISO 8601 format (recommended)
('15-Nov-2023'),       -- Traditional PostgreSQL format
('11/15/2023'),        -- US format (MM/DD/YYYY)
('15.11.2023'),        -- European format (DD.MM.YYYY)
('November 15, 2023'); -- Month name format

-- DDD convention
INSERT INTO "TimeManagement"."EventCalendar" ("EventDate") VALUES 
('2023-11-15');
```

### Time Input

Time values accept multiple formats as well, with hours, minutes, seconds, and optional fractional seconds:

```sql
-- Different ways to input times
INSERT INTO time_management.event_calendar (event_time) VALUES 
('14:30:00'),           -- 24-hour format with seconds
('14:30'),              -- 24-hour format without seconds
('02:30:00 PM'),        -- 12-hour format with AM/PM
('14:30:00.5');         -- With fractional seconds

-- DDD convention
INSERT INTO "TimeManagement"."EventCalendar" ("EventTime") VALUES 
('14:30:00');
```

### Timestamp Input

Timestamps combine date and time values and can include timezone information:

```sql
-- Different ways to input timestamps
INSERT INTO time_management.event_calendar (event_timestamp) VALUES 
('2023-11-15 14:30:00'),               -- Date and time
('2023-11-15T14:30:00'),               -- ISO 8601 with T separator
('November 15, 2023 14:30:00');        -- Verbose format

-- With timezone (timestamptz)
INSERT INTO time_management.event_calendar (event_timestamp_tz) VALUES
('2023-11-15 14:30:00+02'),           -- With explicit timezone
('2023-11-15 14:30:00 PST'),          -- Named timezone
('2023-11-15 14:30:00 America/Los_Angeles'); -- IANA timezone name

-- DDD convention
INSERT INTO "TimeManagement"."EventCalendar" ("EventTimestampWithTZ") VALUES 
('2023-11-15 14:30:00+02');
```

### Interval Input

Intervals represent durations and accept various input formats:

```sql
-- Different ways to input intervals
INSERT INTO time_management.event_calendar (event_duration) VALUES 
('1 day'),                   -- Simple duration
('2 hours 30 minutes'),      -- Multiple units
('1 year 2 months 15 days'), -- Complex duration
('P1Y2M15DT2H30M'),         -- ISO 8601 format
('1-2');                     -- PostgreSQL interval year-month format

-- DDD convention
INSERT INTO "TimeManagement"."EventCalendar" ("EventDuration") VALUES 
('2 hours 30 minutes');
```

## Date and Time Output

PostgreSQL provides various formatting options to display date and time values according to application requirements.

### Output Styles

The output of date and time values can be controlled using session settings:

```sql
-- Setting output styles
SET datestyle = 'ISO';                 -- ISO 8601: 2023-11-15
SET datestyle = 'SQL, MDY';           -- SQL with US ordering: 11/15/2023
SET datestyle = 'Postgres, DMY';      -- Traditional with European ordering: 15/11/2023
SET datestyle = 'German';             -- German format: 15.11.2023
```

### Output Function Formatting

For more precise control, PostgreSQL provides formatting functions:

```sql
-- Format date/time output using TO_CHAR
SELECT TO_CHAR(calendar.event_date, 'DD Month YYYY')
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT TO_CHAR("Calendar"."EventDate", 'DD Month YYYY')
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

Common formatting codes for `TO_CHAR`:

| Format Code | Description | Example |
|-------------|-------------|---------|
| YYYY | 4-digit year | 2023 |
| MM | 2-digit month | 11 |
| DD | 2-digit day | 15 |
| HH24 | Hour (24-hour format) | 14 |
| HH | Hour (12-hour format) | 02 |
| MI | Minutes | 30 |
| SS | Seconds | 45 |
| "Month" | Full month name | November |
| "Day" | Full day of week | Tuesday |
| TZ | Timezone | PST |

## Date and Time Operators and Functions

PostgreSQL provides many functions and operators for manipulating date and time values.

### Basic Arithmetic

You can perform addition and subtraction on date/time values:

```sql
-- Add 3 days to a date
SELECT calendar.event_date + INTERVAL '3 days'
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- Subtract timestamps to get an interval
SELECT calendar.event_timestamp - calendar.event_timestamp_tz AS time_difference
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT "Calendar"."EventDate" + INTERVAL '3 days'
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

### Extracting Date/Time Components

The `EXTRACT` function retrieves specific parts of a date/time value:

```sql
-- Extract year, month, day components
SELECT 
    EXTRACT(YEAR FROM calendar.event_date) AS year,
    EXTRACT(MONTH FROM calendar.event_date) AS month,
    EXTRACT(DAY FROM calendar.event_date) AS day
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- Other useful extractions
SELECT 
    EXTRACT(HOUR FROM calendar.event_timestamp) AS hour,
    EXTRACT(MINUTE FROM calendar.event_timestamp) AS minute,
    EXTRACT(DOW FROM calendar.event_date) AS day_of_week, -- 0 (Sunday) to 6 (Saturday)
    EXTRACT(EPOCH FROM calendar.event_timestamp) AS epoch_seconds
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT 
    EXTRACT(YEAR FROM "Calendar"."EventDate") AS "Year",
    EXTRACT(MONTH FROM "Calendar"."EventDate") AS "Month",
    EXTRACT(DAY FROM "Calendar"."EventDate") AS "Day"
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

### Current Date and Time

PostgreSQL provides functions to retrieve the current date and time:

```sql
SELECT 
    CURRENT_DATE AS today,
    CURRENT_TIME AS now_time,
    CURRENT_TIMESTAMP AS now_timestamp,
    NOW() AS now_full,
    CLOCK_TIMESTAMP() AS exact_current_time;
```

The difference between `NOW()` and `CLOCK_TIMESTAMP()` is that `NOW()` is calculated once at the start of the transaction, while `CLOCK_TIMESTAMP()` returns the actual current time for each call.

### Date/Time Construction

Functions to build date and time values from components:

```sql
-- Build a date from year, month, day
SELECT MAKE_DATE(2023, 11, 15) AS constructed_date;

-- Build a time from hour, minute, second
SELECT MAKE_TIME(14, 30, 45) AS constructed_time;

-- Build a timestamp from date and time
SELECT MAKE_TIMESTAMP(2023, 11, 15, 14, 30, 45) AS constructed_timestamp;

-- Build a timestamptz with timezone
SELECT MAKE_TIMESTAMPTZ(2023, 11, 15, 14, 30, 45, 'Europe/Berlin') AS constructed_timestamptz;
```

## Working with Timezones

Understanding and managing timezones is crucial for applications that operate globally.

### Setting the Session Timezone

The session timezone affects how timestamptz values are displayed:

```sql
-- Check current timezone setting
SHOW timezone;

-- Change the session timezone
SET timezone = 'Europe/Berlin';
SET timezone = 'America/New_York';
SET timezone = 'UTC';

-- See how the same timestamptz value appears differently
SELECT calendar.event_timestamp_tz
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT "Calendar"."EventTimestampWithTZ"
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

### Timezone Conversion

Converting between different timezones:

```sql
-- Convert a timestamp to different timezones
SELECT 
    calendar.event_timestamp_tz,
    calendar.event_timestamp_tz AT TIME ZONE 'UTC' AS utc_time,
    calendar.event_timestamp_tz AT TIME ZONE 'America/Los_Angeles' AS la_time,
    calendar.event_timestamp_tz AT TIME ZONE 'Asia/Tokyo' AS tokyo_time
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT 
    "Calendar"."EventTimestampWithTZ",
    "Calendar"."EventTimestampWithTZ" AT TIME ZONE 'UTC' AS "UtcTime",
    "Calendar"."EventTimestampWithTZ" AT TIME ZONE 'America/Los_Angeles' AS "LosAngelesTime"
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

Note: The `AT TIME ZONE` operator has different behavior depending on the input data type:
- With `timestamp with time zone`: Converts to the specified timezone
- With `timestamp without time zone`: Interprets the timestamp as being in the specified timezone

### Available Timezones

PostgreSQL supports both abbreviated timezone names (like 'EST') and full IANA timezone names (like 'America/New_York'):

```sql
-- List all available timezones
SELECT * FROM pg_timezone_names;

-- List supported timezone abbreviations
SELECT * FROM pg_timezone_abbrevs;
```

IANA timezone names are recommended over abbreviations as they handle daylight saving time transitions correctly.

## Date and Time Functions for Reporting

PostgreSQL provides specialized functions that are particularly useful for reporting and data analysis.

### Date/Time Truncation

The `DATE_TRUNC` function rounds a timestamp down to a specified precision:

```sql
-- Truncate timestamps to different levels of precision
SELECT 
    calendar.event_timestamp,
    DATE_TRUNC('year', calendar.event_timestamp) AS year_start,
    DATE_TRUNC('month', calendar.event_timestamp) AS month_start,
    DATE_TRUNC('day', calendar.event_timestamp) AS day_start,
    DATE_TRUNC('hour', calendar.event_timestamp) AS hour_start,
    DATE_TRUNC('week', calendar.event_timestamp) AS week_start
FROM time_management.event_calendar AS calendar
WHERE calendar.event_id = 1;

-- DDD convention
SELECT 
    "Calendar"."EventTimestamp",
    DATE_TRUNC('month', "Calendar"."EventTimestamp") AS "MonthStart"
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventId" = 1;
```

This is particularly useful for grouping data by time periods in reports:

```sql
-- Group sales by month
SELECT 
    DATE_TRUNC('month', sales.sale_date) AS month,
    SUM(sales.amount) AS monthly_total
FROM retail.sales AS sales
GROUP BY DATE_TRUNC('month', sales.sale_date)
ORDER BY month;

-- DDD convention
SELECT 
    DATE_TRUNC('month', "Sales"."SaleDate") AS "Month",
    SUM("Sales"."Amount") AS "MonthlyTotal"
FROM "Retail"."Sales" AS "Sales"
GROUP BY DATE_TRUNC('month', "Sales"."SaleDate")
ORDER BY "Month";
```

### Date/Time Intervals and Ranges

Working with date ranges and generating series of dates:

```sql
-- Generate a series of dates
SELECT generate_series(
    '2023-01-01'::date,
    '2023-01-31'::date,
    '1 day'::interval
) AS calendar_day;

-- Generate a series of timestamps
SELECT generate_series(
    '2023-01-01 00:00:00'::timestamp,
    '2023-01-01 23:59:59'::timestamp,
    '1 hour'::interval
) AS hourly_timestamp;
```

This is useful for creating calendar tables or ensuring all periods are represented in reports even when data is missing:

```sql
-- Create a complete sales report with all days, including days with no sales
WITH date_range AS (
    SELECT generate_series(
        '2023-01-01'::date,
        '2023-01-31'::date,
        '1 day'::interval
    )::date AS day
)
SELECT 
    date_range.day,
    COALESCE(SUM(sales.amount), 0) AS daily_total
FROM date_range
LEFT JOIN retail.sales AS sales ON date_range.day = sales.sale_date::date
GROUP BY date_range.day
ORDER BY date_range.day;

-- DDD convention
WITH "DateRange" AS (
    SELECT generate_series(
        '2023-01-01'::date,
        '2023-01-31'::date,
        '1 day'::interval
    )::date AS "Day"
)
SELECT 
    "DateRange"."Day",
    COALESCE(SUM("Sales"."Amount"), 0) AS "DailyTotal"
FROM "DateRange"
LEFT JOIN "Retail"."Sales" AS "Sales" ON "DateRange"."Day" = "Sales"."SaleDate"::date
GROUP BY "DateRange"."Day"
ORDER BY "DateRange"."Day";
```

### Age and Difference Calculations

Calculate ages and differences between dates:

```sql
-- Calculate age between two dates
SELECT AGE('2023-11-15', '1990-05-10') AS age_interval;
-- Result: "33 years 6 months 5 days"

-- Calculate age in years
SELECT EXTRACT(YEAR FROM AGE('2023-11-15', '1990-05-10')) AS age_years;
-- Result: 33

-- Calculate difference in days
SELECT ('2023-11-15'::date - '2023-10-01'::date) AS days_difference;
-- Result: 45

-- Calculate difference in business days (excluding weekends)
SELECT business_days.calculate_business_days('2023-10-01', '2023-11-15') AS business_days_count;
-- Would require a custom function 'calculate_business_days'

-- DDD convention
SELECT AGE('2023-11-15', "Employee"."BirthDate") AS "AgeInterval"
FROM "HR"."Employee" AS "Employee"
WHERE "Employee"."EmployeeId" = 1;
```

## Working with Intervals

Intervals represent durations or periods of time and can be manipulated flexibly.

### Creating and Manipulating Intervals

```sql
-- Create intervals using various formats
SELECT 
    INTERVAL '1 year 2 months 15 days 12 hours 30 minutes' AS complex_interval,
    INTERVAL '15 days' AS days_interval,
    INTERVAL '4 hours' AS hours_interval;

-- Arithmetic with intervals
SELECT 
    INTERVAL '1 day' + INTERVAL '3 hours' AS combined_interval,
    INTERVAL '2 days' * 3 AS multiplied_interval,
    INTERVAL '12 hours' / 3 AS divided_interval;

-- Interval with timestamp
SELECT 
    NOW(),
    NOW() + INTERVAL '3 days' AS three_days_later,
    NOW() - INTERVAL '1 month' AS one_month_ago;

-- Justification of intervals (normalizing units)
SELECT JUSTIFY_HOURS(INTERVAL '36 hours') AS justified_hours; -- Converts to 1 day 12 hours
SELECT JUSTIFY_DAYS(INTERVAL '40 days') AS justified_days;    -- Converts to 1 month 10 days
```

### Using Intervals in Practical Applications

```sql
-- Find records from the last 30 days
SELECT calendar.event_id, calendar.event_date
FROM time_management.event_calendar AS calendar
WHERE calendar.event_date >= CURRENT_DATE - INTERVAL '30 days';

-- Schedule future events
INSERT INTO time_management.event_calendar (event_date, event_time, event_duration)
VALUES (
    CURRENT_DATE + INTERVAL '2 weeks',
    '14:00:00',
    INTERVAL '2 hours'
);

-- Calculate subscription durations
SELECT 
    subscription.user_id,
    subscription.start_date,
    subscription.end_date,
    AGE(subscription.end_date, subscription.start_date) AS subscription_duration
FROM subscription_management.subscriptions AS subscription
WHERE subscription.end_date IS NOT NULL;

-- DDD convention
SELECT "Calendar"."EventId", "Calendar"."EventDate"
FROM "TimeManagement"."EventCalendar" AS "Calendar"
WHERE "Calendar"."EventDate" >= CURRENT_DATE - INTERVAL '30 days';
```

## Date/Time Indexing and Optimization

Properly indexing date and time columns is essential for query performance.

### Indexing Strategies

```sql
-- Standard B-tree index for exact matches and ranges
CREATE INDEX idx_time_management_event_calendar_event_date 
ON time_management.event_calendar (event_date);

-- Functional index for date truncation queries
CREATE INDEX idx_time_management_event_calendar_month 
ON time_management.event_calendar (DATE_TRUNC('month', event_timestamp));

-- Composite index for time-based filtering with additional conditions
CREATE INDEX idx_time_management_event_calendar_date_type 
ON time_management.event_calendar (event_date, event_type);

-- DDD convention
CREATE INDEX "IX_TimeManagement_EventCalendar_EventDate" 
ON "TimeManagement"."EventCalendar" ("EventDate");
```

### Query Optimization

Tips for optimizing date/time-based queries:

```sql
-- Use sargable conditions to allow index usage
-- Good (index can be used)
SELECT * FROM time_management.event_calendar
WHERE event_date >= '2023-01-01' AND event_date < '2023-02-01';

-- Bad (prevents index usage)
SELECT * FROM time_management.event_calendar
WHERE EXTRACT(MONTH FROM event_date) = 1 AND EXTRACT(YEAR FROM event_date) = 2023;
-- Better alternative
SELECT * FROM time_management.event_calendar
WHERE event_date >= '2023-01-01' AND event_date < '2023-02-01';

-- DDD convention
-- Good (index can be used)
SELECT * FROM "TimeManagement"."EventCalendar"
WHERE "EventDate" >= '2023-01-01' AND "EventDate" < '2023-02-01';
```

### Partitioning by Date

For large time-series tables, consider using PostgreSQL's partitioning feature:

```sql
-- Create a partitioned table by month
CREATE TABLE time_management.event_log (
    log_id SERIAL,
    event_time timestamptz NOT NULL,
    event_type VARCHAR(50),
    event_data JSONB
) PARTITION BY RANGE (event_time);

-- Create monthly partitions
CREATE TABLE time_management.event_log_2023_01 PARTITION OF time_management.event_log
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
    
CREATE TABLE time_management.event_log_2023_02 PARTITION OF time_management.event_log
    FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');
    
-- And so on for other months

-- DDD convention
CREATE TABLE "TimeManagement"."EventLog" (
    "LogId" SERIAL,
    "EventTime" timestamptz NOT NULL,
    "EventType" VARCHAR(50),
    "EventData" JSONB
) PARTITION BY RANGE ("EventTime");

CREATE TABLE "TimeManagement"."EventLog_2023_01" PARTITION OF "TimeManagement"."EventLog"
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
```

## Special Date/Time Functions

PostgreSQL provides additional specialized functions for date/time manipulation.

### Holiday Calculations

Creating custom functions to determine holidays:

```sql
-- Create a function to check if a date is a US federal holiday
CREATE OR REPLACE FUNCTION time_management.is_us_federal_holiday(check_date date) 
RETURNS boolean AS $$
DECLARE
    year_val integer := EXTRACT(YEAR FROM check_date);
    month_val integer := EXTRACT(MONTH FROM check_date);
    day_val integer := EXTRACT(DAY FROM check_date);
BEGIN
    -- New Year's Day
    IF month_val = 1 AND day_val = 1 THEN
        RETURN true;
    END IF;
    
    -- Memorial Day (last Monday in May)
    IF month_val = 5 AND
       day_val BETWEEN 25 AND 31 AND
       EXTRACT(DOW FROM check_date) = 1 THEN
        RETURN true;
    END IF;
    
    -- Independence Day
    IF month_val = 7 AND day_val = 4 THEN
        RETURN true;
    END IF;
    
    -- Labor Day (first Monday in September)
    IF month_val = 9 AND
       day_val BETWEEN 1 AND 7 AND
       EXTRACT(DOW FROM check_date) = 1 THEN
        RETURN true;
    END IF;
    
    -- Thanksgiving (fourth Thursday in November)
    IF month_val = 11 AND
       EXTRACT(DOW FROM check_date) = 4 AND
       day_val BETWEEN 22 AND 28 THEN
        RETURN true;
    END IF;
    
    -- Christmas
    IF month_val = 12 AND day_val = 25 THEN
        RETURN true;
    END IF;
    
    RETURN false;
END;
$$ LANGUAGE plpgsql;

-- DDD convention
CREATE OR REPLACE FUNCTION "TimeManagement"."IsUsFederalHoliday"(check_date date) 
RETURNS boolean AS $$
-- Same function body as above
$$ LANGUAGE plpgsql;
```

### Date/Time Validation

Functions to validate date/time input:

```sql
-- Create a function to validate date format
CREATE OR REPLACE FUNCTION time_management.is_valid_date(text_date text)
RETURNS boolean AS $$
BEGIN
    PERFORM text_date::date;
    RETURN true;
EXCEPTION
    WHEN OTHERS THEN
        RETURN false;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT time_management.is_valid_date('2023-11-15');  -- Returns true
SELECT time_management.is_valid_date('2023-13-15');  -- Returns false (invalid month)

-- DDD convention
CREATE OR REPLACE FUNCTION "TimeManagement"."IsValidDate"(text_date text)
RETURNS boolean AS $$
-- Same function body as above
$$ LANGUAGE plpgsql;
```

### Business Days Calculation

Function to calculate business days between dates:

```sql
-- Create a function to calculate business days between two dates
CREATE OR REPLACE FUNCTION time_management.calculate_business_days(
    start_date date,
    end_date date,
    exclude_holidays boolean DEFAULT true
) 
RETURNS integer AS $$
DECLARE
    count_days integer := 0;
    current_date date := start_date;
BEGIN
    WHILE current_date <= end_date LOOP
        -- Check if day is weekday (not Saturday or Sunday)
        IF EXTRACT(DOW FROM current_date) NOT IN (0, 6) THEN
            -- Check if we should exclude holidays
            IF NOT exclude_holidays OR 
               NOT time_management.is_us_federal_holiday(current_date) THEN
                count_days := count_days + 1;
            END IF;
        END IF;
        current_date := current_date + 1;
    END LOOP;
    
    RETURN count_days;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT time_management.calculate_business_days('2023-01-01', '2023-01-31');

-- DDD convention
CREATE OR REPLACE FUNCTION "TimeManagement"."CalculateBusinessDays"(
    start_date date,
    end_date date,
    exclude_holidays boolean DEFAULT true
) 
RETURNS integer AS $$
-- Same function body as above but using "TimeManagement"."IsUsFederalHoliday"
$$ LANGUAGE plpgsql;
```

## Best Practices and Common Pitfalls

### Best Practices

1. **Use `timestamptz` for most applications**: Store timestamps with timezone information to avoid ambiguity.

2. **Apply consistent timezone handling**:
   ```sql
   SET timezone = 'UTC';
   -- Store all times in the database as UTC
   ```

3. **Use ISO 8601 format for date/time input**:
   ```sql
   -- Prefer this format
   INSERT INTO time_management.event_calendar (event_date) VALUES ('2023-11-15');
   ```

4. **Use named parameters for clarity in interval specifications**:
   ```sql
   -- More readable
   SELECT NOW() + INTERVAL '3 months 2 days';
   ```

5. **Index date/time columns appropriately**:
   ```sql
   -- For range queries
   CREATE INDEX idx_time_management_event_calendar_date
   ON time_management.event_calendar (event_date);
   
   -- DDD convention
   CREATE INDEX "IX_TimeManagement_EventCalendar_EventDate"
   ON "TimeManagement"."EventCalendar" ("EventDate");
   ```

### Common Pitfalls

1. **Timezone confusion**:
   ```sql
   -- Beware of timezone differences
   SELECT '2023-11-15 12:00:00'::timestamp;      -- Assumes local timezone
   SELECT '2023-11-15 12:00:00'::timestamptz;    -- Converts to UTC based on session timezone
   ```

2. **Date arithmetic errors**:
   ```sql
   -- Incorrect: Does not account for varying month lengths
   SELECT '2023-01-31'::date + INTERVAL '1 month';  -- Results in 2023-02-28
   
   -- Better method for adding months
   SELECT date_trunc('month', '2023-01-31'::date + INTERVAL '1 month') + 
          INTERVAL '1 month - 1 day';  -- Gets last day of next month
   ```

3. **Performance issues with non-sargable conditions**:
   ```sql
   -- Poor performance (can't use index)
   SELECT * FROM time_management.event_calendar 
   WHERE EXTRACT(YEAR FROM event_date) = 2023;
   
   -- Better performance (can use index)
   SELECT * FROM time_management.event_calendar 
   WHERE event_date >= '2023-01-01' AND event_date < '2024-01-01';
   ```

4. **Ignoring fractional seconds**:
   ```sql
   -- May lose precision
   SELECT '2023-11-15 12:30:45.123456'::timestamp::time(0);  -- Truncates milliseconds
   ```

5. **Misunderstanding the `AT TIME ZONE` operator**:
   ```sql
   -- Different behavior based on input type
   SELECT '2023-11-15 12:00:00'::timestamptz AT TIME ZONE 'UTC';  -- Converts to UTC
   SELECT '2023-11-15 12:00:00'::timestamp AT TIME ZONE 'UTC';    -- Interprets as UTC
   ```

## Summary

Working with dates and times in PostgreSQL offers powerful capabilities for temporal data management. The key points from this chapter include:

1. PostgreSQL provides specialized data types for dates, times, timestamps, and intervals with varying precision and timezone awareness.

2. Timezone handling is robust with support for IANA timezone names, offering global time tracking capabilities.

3. Input and output formats are flexible, with ISO 8601 being the recommended standard for clarity and consistency.

4. A rich set of functions enables extraction, manipulation, and calculation with date and time values.

5. Intervals allow representation of time durations with arithmetic operations.

6. Performance optimization through proper indexing and query structure is essential for large temporal datasets.

7. Custom functions can extend PostgreSQL's capabilities for specialized time-related calculations.

By leveraging these features, applications can efficiently manage schedules, track events, analyze time-series data, and handle global time differences with confidence.