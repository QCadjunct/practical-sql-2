# Chapter 15 - Saving Time with Views, Functions, and Triggers

## Introduction

This chapter explores techniques to encapsulate queries and logic into reusable PostgreSQL database objects that will speed up your workflow. By following the DRY (Don't Repeat Yourself) programming principle, we'll learn how to avoid repetition, save time, and prevent unnecessary mistakes. The chapter covers:

- Creating and using database views
- Building custom functions for data operations
- Setting up triggers for automating database actions

Each of these features helps reduce repetitive work and maintain data integrity. We'll use both PostgreSQL Standard (snake_case) and Domain Driven Design (PascalCase) conventions throughout our examples.

## Using Views to Simplify Queries

A view is a virtual table created dynamically using a saved query. Views execute their underlying query each time you access them, allowing you to:

- Write a query once and access the results when needed
- Reduce complexity by showing only relevant columns
- Provide security by limiting access to certain data

### Creating and Querying Views

#### PostgreSQL Standard (snake_case)

```sql
-- Create a view showing Nevada counties' population data
CREATE OR REPLACE VIEW nevada_counties_pop_2010 AS
SELECT geo_name,
       state_fips,
       county_fips,
       p0010001 AS pop_2010
FROM us_counties_2010
WHERE state_us_abbreviation = 'NV'
ORDER BY county_fips;

-- Query the view
SELECT *
FROM nevada_counties_pop_2010
LIMIT 5;
```

**Sample Output:**
```
geo_name          state_fips county_fips pop_2010
Churchill County  32         001         24877
Clark County      32         003         1951269
Douglas County    32         005         46997
Elko County       32         007         48818
Esmeralda County  32         009         783
```

#### Domain Driven Design (PascalCase)

```sql
-- Create a view showing Nevada counties' population data
CREATE OR REPLACE VIEW "Census"."NevadaCountiesPopulation2010" AS
SELECT "GeoName",
       "StateFips",
       "CountyFips",
       "P0010001" AS "Population2010"
FROM "Census"."USCounty2010"
WHERE "StateAbbreviation" = 'NV'
ORDER BY "CountyFips";

-- Query the view
SELECT *
FROM "Census"."NevadaCountiesPopulation2010"
LIMIT 5;
```

**Sample Output:**
```
GeoName           StateFips CountyFips Population2010
Churchill County  32        001        24877
Clark County      32        003        1951269
Douglas County    32        005        46997
Elko County       32        007        48818
Esmeralda County  32        009        783
```

### Creating a View for Population Change Analysis

#### PostgreSQL Standard (snake_case)

```sql
CREATE OR REPLACE VIEW county_pop_change_2010_2000 AS
SELECT c2010.geo_name,
       c2010.state_us_abbreviation AS st,
       c2010.state_fips,
       c2010.county_fips,
       c2010.p0010001 AS pop_2010,
       c2000.p0010001 AS pop_2000,
       round((CAST(c2010.p0010001 AS numeric(8,1)) - c2000.p0010001)
             / c2000.p0010001 * 100, 1) AS pct_change_2010_2000
FROM us_counties_2010 c2010 INNER JOIN us_counties_2000 c2000
ON c2010.state_fips = c2000.state_fips
AND c2010.county_fips = c2000.county_fips
ORDER BY c2010.state_fips, c2010.county_fips;

-- Querying specific columns from the view for Nevada counties
SELECT geo_name,
       st,
       pop_2010,
       pct_change_2010_2000
FROM county_pop_change_2010_2000
WHERE st = 'NV'
LIMIT 5;
```

**Sample Output:**
```
geo_name          st  pop_2010  pct_change_2010_2000
Churchill County  NV  24877     3.7
Clark County      NV  1951269   41.8
Douglas County    NV  46997     13.9
Elko County       NV  48818     7.8
Esmeralda County  NV  783       -19.4
```

#### Domain Driven Design (PascalCase)

```sql
CREATE OR REPLACE VIEW "Census"."VW_CountyPopulationChange20102000" AS
SELECT c2010."GeoName",
       c2010."StateAbbreviation" AS "State",
       c2010."StateFips",
       c2010."CountyFips",
       c2010."P0010001" AS "Population2010",
       c2000."P0010001" AS "Population2000",
       round((CAST(c2010."P0010001" AS numeric(8,1)) - c2000."P0010001")
             / c2000."P0010001" * 100, 1) AS "PercentChange20102000"
FROM "Census"."USCounty2010" c2010 INNER JOIN "Census"."USCounty2000" c2000
ON c2010."StateFips" = c2000."StateFips"
AND c2010."CountyFips" = c2000."CountyFips"
ORDER BY c2010."StateFips", c2010."CountyFips";

-- Querying specific columns from the view for Nevada counties
SELECT "GeoName",
       "State",
       "Population2010",
       "PercentChange20102000"
FROM "Census"."VW_CountyPopulationChange20102000"
WHERE "State" = 'NV'
LIMIT 5;
```

**Sample Output:**
```
GeoName           State Population2010  PercentChange20102000
Churchill County  NV    24877           3.7
Clark County      NV    1951269         41.8
Douglas County    NV    46997           13.9
Elko County       NV    48818           7.8
Esmeralda County  NV    783             -19.4
```

### Inserting, Updating, and Deleting Data Using a View

Views can be used to insert, update, or delete data in underlying tables if they meet certain conditions:
- The view must reference a single table
- The view's query can't contain DISTINCT, GROUP BY, or similar clauses

#### Creating a View for Tax Department Employees

#### PostgreSQL Standard (snake_case)

```sql
CREATE OR REPLACE VIEW employees_tax_dept AS
SELECT emp_id,
       first_name,
       last_name,
       dept_id
FROM employees
WHERE dept_id = 1
ORDER BY emp_id
WITH LOCAL CHECK OPTION;

-- View the Tax Department employees
SELECT * FROM employees_tax_dept;
```

**Sample Output:**
```
emp_id  first_name  last_name  dept_id
1       Nancy       Jones      1
2       Lee         Smith      1
```

#### Domain Driven Design (PascalCase)

```sql
CREATE OR REPLACE VIEW "HR"."VW_TaxDepartmentEmployee" AS
SELECT "EmployeeId",
       "FirstName",
       "LastName",
       "DepartmentId"
FROM "HR"."Employee"
WHERE "DepartmentId" = 1
ORDER BY "EmployeeId"
WITH LOCAL CHECK OPTION;

-- View the Tax Department employees
SELECT * FROM "HR"."VW_TaxDepartmentEmployee";
```

**Sample Output:**
```
EmployeeId  FirstName  LastName  DepartmentId
1           Nancy      Jones     1
2           Lee        Smith     1
```

#### Inserting Rows Using the View

#### PostgreSQL Standard (snake_case)

```sql
-- Successful insert (Tax Department employee)
INSERT INTO employees_tax_dept (first_name, last_name, dept_id)
VALUES ('Suzanne', 'Legere', 1);

-- Failed insert (not Tax Department)
INSERT INTO employees_tax_dept (first_name, last_name, dept_id)
VALUES ('Jamil', 'White', 2);
-- Error: new row violates check option for view "employees_tax_dept"

-- View the updated Tax Department employees
SELECT * FROM employees_tax_dept;
```

**Sample Output:**
```
emp_id  first_name  last_name  dept_id
1       Nancy       Jones      1
2       Lee         Smith      1
5       Suzanne     Legere     1
```

#### Domain Driven Design (PascalCase)

```sql
-- Successful insert (Tax Department employee)
INSERT INTO "HR"."VW_TaxDepartmentEmployee" ("FirstName", "LastName", "DepartmentId")
VALUES ('Suzanne', 'Legere', 1);

-- Failed insert (not Tax Department)
INSERT INTO "HR"."VW_TaxDepartmentEmployee" ("FirstName", "LastName", "DepartmentId")
VALUES ('Jamil', 'White', 2);
-- Error: new row violates check option for view "HR"."VW_TaxDepartmentEmployee"

-- View the updated Tax Department employees
SELECT * FROM "HR"."VW_TaxDepartmentEmployee";
```

**Sample Output:**
```
EmployeeId  FirstName  LastName  DepartmentId
1           Nancy      Jones     1
2           Lee        Smith     1
5           Suzanne    Legere    1
```

#### Updating and Deleting Rows Using a View

#### PostgreSQL Standard (snake_case)

```sql
-- Update an employee's name
UPDATE employees_tax_dept
SET last_name = 'Le Gere'
WHERE emp_id = 5;

-- Delete an employee
DELETE FROM employees_tax_dept
WHERE emp_id = 5;
```

#### Domain Driven Design (PascalCase)

```sql
-- Update an employee's name
UPDATE "HR"."VW_TaxDepartmentEmployee"
SET "LastName" = 'Le Gere'
WHERE "EmployeeId" = 5;

-- Delete an employee
DELETE FROM "HR"."VW_TaxDepartmentEmployee"
WHERE "EmployeeId" = 5;
```

## Programming Your Own Functions

Functions allow you to encapsulate code for reuse. PostgreSQL supports several programming languages for functions, including SQL, PL/pgSQL, PL/Python, and others.

### Creating the percent_change() Function

#### PostgreSQL Standard (snake_case)

```sql
CREATE OR REPLACE FUNCTION percent_change(
    new_value numeric,
    old_value numeric,
    decimal_places integer DEFAULT 1)
RETURNS numeric AS
'SELECT round(
    ((new_value - old_value) / old_value) * 100, 
    decimal_places
);'
LANGUAGE SQL
IMMUTABLE
RETURNS NULL ON NULL INPUT;

-- Test the function
SELECT percent_change(110, 108, 2);
```

**Sample Output:**
```
percent_change
1.85
```

#### Domain Driven Design (PascalCase)

```sql
CREATE OR REPLACE FUNCTION "Math"."PercentChange"(
    "NewValue" numeric,
    "OldValue" numeric,
    "DecimalPlaces" integer DEFAULT 1)
RETURNS numeric AS
'SELECT round(
    (("NewValue" - "OldValue") / "OldValue") * 100, 
    "DecimalPlaces"
);'
LANGUAGE SQL
IMMUTABLE
RETURNS NULL ON NULL INPUT;

-- Test the function
SELECT "Math"."PercentChange"(110, 108, 2);
```

**Sample Output:**
```
PercentChange
1.85
```

### Using the Function with Census Data

#### PostgreSQL Standard (snake_case)

```sql
SELECT c2010.geo_name,
       c2010.state_us_abbreviation AS st,
       c2010.p0010001 AS pop_2010,
       percent_change(c2010.p0010001, c2000.p0010001) AS pct_chg_func,
       round((CAST(c2010.p0010001 AS numeric(8,1)) - c2000.p0010001)
             / c2000.p0010001 * 100, 1) AS pct_chg_formula
FROM us_counties_2010 c2010 INNER JOIN us_counties_2000 c2000
ON c2010.state_fips = c2000.state_fips
AND c2010.county_fips = c2000.county_fips
ORDER BY pct_chg_func DESC
LIMIT 5;
```

**Sample Output:**
```
geo_name         st  pop_2010  pct_chg_func  pct_chg_formula
Kendall County   IL  114736    110.4         110.4
Pinal County     AZ  375770    109.1         109.1
Flagler County   FL  95696     92.0          92.0
Lincoln County   SD  44882     85.8          85.8
Loudoun County   VA  312311    84.1          84.1
```

#### Domain Driven Design (PascalCase)

```sql
SELECT c2010."GeoName",
       c2010."StateAbbreviation" AS "State",
       c2010."P0010001" AS "Population2010",
       "Math"."PercentChange"(c2010."P0010001", c2000."P0010001") AS "PercentChangeFunc",
       round((CAST(c2010."P0010001" AS numeric(8,1)) - c2000."P0010001")
             / c2000."P0010001" * 100, 1) AS "PercentChangeFormula"
FROM "Census"."USCounty2010" c2010 INNER JOIN "Census"."USCounty2000" c2000
ON c2010."StateFips" = c2000."StateFips"
AND c2010."CountyFips" = c2000."CountyFips"
ORDER BY "PercentChangeFunc" DESC
LIMIT 5;
```

**Sample Output:**
```
GeoName          State Population2010  PercentChangeFunc  PercentChangeFormula
Kendall County   IL    114736          110.4              110.4
Pinal County     AZ    375770          109.1              109.1
Flagler County   FL    95696           92.0               92.0
Lincoln County   SD    44882           85.8               85.8
Loudoun County   VA    312311          84.1               84.1
```

### Updating Data with a Function

#### PostgreSQL Standard (snake_case)

```sql
-- Add a personal_days column to teachers table
ALTER TABLE teachers ADD COLUMN personal_days integer;

-- Create a function to update personal days based on hire date
CREATE OR REPLACE FUNCTION update_personal_days()
RETURNS void AS $$
BEGIN
    UPDATE teachers
    SET personal_days =
        CASE WHEN (now() - hire_date) BETWEEN '5 years'::interval 
                                     AND '10 years'::interval THEN 4
             WHEN (now() - hire_date) > '10 years'::interval THEN 5
             ELSE 3
        END;
    RAISE NOTICE 'personal_days updated!';
END;
$$ LANGUAGE plpgsql;

-- Execute the function
SELECT update_personal_days();

-- View the results
SELECT first_name, last_name, hire_date, personal_days
FROM teachers;
```

**Sample Output:**
```
first_name  last_name  hire_date    personal_days
Janet       Smith      2011-10-30   4
Lee         Reynolds   1993-05-22   5
Samuel      Cole       2005-08-01   5
Samantha    Bush       2011-10-30   4
Betty       Diaz       2005-08-30   5
Kathleen    Roush      2010-10-22   4
```

#### Domain Driven Design (PascalCase)

```sql
-- Add a personal_days column to teachers table
ALTER TABLE "School"."Teacher" ADD COLUMN "PersonalDaysAllowed" integer;

-- Create a function to update personal days based on hire date
CREATE OR REPLACE FUNCTION "School"."UpdatePersonalDays"()
RETURNS void AS $$
BEGIN
    UPDATE "School"."Teacher"
    SET "PersonalDaysAllowed" =
        CASE WHEN (now() - "HireDate") BETWEEN '5 years'::interval 
                                      AND '10 years'::interval THEN 4
             WHEN (now() - "HireDate") > '10 years'::interval THEN 5
             ELSE 3
        END;
    RAISE NOTICE 'Personal days allowance updated!';
END;
$$ LANGUAGE plpgsql;

-- Execute the function
SELECT "School"."UpdatePersonalDays"();

-- View the results
SELECT "FirstName", "LastName", "HireDate", "PersonalDaysAllowed"
FROM "School"."Teacher";
```

**Sample Output:**
```
FirstName  LastName  HireDate     PersonalDaysAllowed
Janet      Smith     2011-10-30   4
Lee        Reynolds  1993-05-22   5
Samuel     Cole      2005-08-01   5
Samantha   Bush      2011-10-30   4
Betty      Diaz      2005-08-30   5
Kathleen   Roush     2010-10-22   4
```

### Using the Python Language in a Function

PostgreSQL supports creating functions using languages like Python. First, enable the PL/Python extension:

```sql
CREATE EXTENSION plpythonu;
```

#### PostgreSQL Standard (snake_case)

```sql
CREATE OR REPLACE FUNCTION trim_county(input_string text)
RETURNS text AS $$
import re
cleaned = re.sub(r' County', '', input_string)
return cleaned
$$ LANGUAGE plpythonu;

-- Test the function
SELECT geo_name,
       trim_county(geo_name)
FROM us_counties_2010
ORDER BY state_fips, county_fips
LIMIT 5;
```

**Sample Output:**
```
geo_name          trim_county
Autauga County    Autauga
Baldwin County    Baldwin
Barbour County    Barbour
Bibb County       Bibb
Blount County     Blount
```

#### Domain Driven Design (PascalCase)

```sql
CREATE OR REPLACE FUNCTION "Census"."TrimCounty"("InputString" text)
RETURNS text AS $$
import re
cleaned = re.sub(r' County', '', input_string)
return cleaned
$$ LANGUAGE plpythonu;

-- Test the function
SELECT "GeoName",
       "Census"."TrimCounty"("GeoName")
FROM "Census"."USCounty2010"
ORDER BY "StateFips", "CountyFips"
LIMIT 5;
```

**Sample Output:**
```
GeoName           TrimCounty
Autauga County    Autauga
Baldwin County    Baldwin
Barbour County    Barbour
Bibb County       Bibb
Blount County     Blount
```

## Automating Database Actions with Triggers

A trigger executes a function when a specified event (INSERT, UPDATE, DELETE) occurs on a table or view. You can set a trigger to fire before, after, or instead of the event, and it can execute once per row or once per statement.

### Logging Grade Updates to a Table

#### PostgreSQL Standard (snake_case)

```sql
-- Create tables for grades and history
CREATE TABLE grades (
    student_id bigint, 
    course_id bigint, 
    course varchar(30) NOT NULL, 
    grade varchar(5) NOT NULL,
    PRIMARY KEY (student_id, course_id) 
);

INSERT INTO grades
VALUES
    (1, 1, 'Biology 2', 'F'),
    (1, 2, 'English 11B', 'D'),
    (1, 3, 'World History 11B', 'C'),
    (1, 4, 'Trig 2', 'B');

CREATE TABLE grades_history (
    student_id bigint NOT NULL, 
    course_id bigint NOT NULL, 
    change_time timestamp with time zone NOT NULL, 
    course varchar(30) NOT NULL,
    old_grade varchar(5) NOT NULL, 
    new_grade varchar(5) NOT NULL,
    PRIMARY KEY (student_id, course_id, change_time) 
);

-- Create function to record grade changes
CREATE OR REPLACE FUNCTION record_if_grade_changed()
RETURNS trigger AS
$$
BEGIN
    IF NEW.grade <> OLD.grade THEN
        INSERT INTO grades_history (
            student_id,
            course_id,
            change_time,
            course,
            old_grade,
            new_grade)
        VALUES
            (OLD.student_id,
             OLD.course_id,
             now(),
             OLD.course,
             OLD.grade,
             NEW.grade);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER grades_update
AFTER UPDATE
ON grades
FOR EACH ROW
EXECUTE PROCEDURE record_if_grade_changed();

-- Test with an update
UPDATE grades
SET grade = 'C'
WHERE student_id = 1 AND course_id = 1;

-- Check history
SELECT student_id,
       change_time,
       course,
       old_grade,
       new_grade
FROM grades_history;
```

**Sample Output:**
```
student_id  change_time                      course      old_grade  new_grade
1           2023-11-15 15:42:37.268-05:00    Biology 2   F          C
```

#### Domain Driven Design (PascalCase)

```sql
-- Create tables for grades and history
CREATE TABLE "School"."StudentGrade" (
    "StudentId" bigint, 
    "CourseId" bigint, 
    "CourseName" varchar(30) NOT NULL, 
    "Grade" varchar(5) NOT NULL,
    PRIMARY KEY ("StudentId", "CourseId") 
);

INSERT INTO "School"."StudentGrade"
VALUES
    (1, 1, 'Biology 2', 'F'),
    (1, 2, 'English 11B', 'D'),
    (1, 3, 'World History 11B', 'C'),
    (1, 4, 'Trig 2', 'B');

CREATE TABLE "School"."StudentGradeHistory" (
    "StudentId" bigint NOT NULL, 
    "CourseId" bigint NOT NULL, 
    "ChangeTime" timestamp with time zone NOT NULL, 
    "CourseName" varchar(30) NOT NULL,
    "OldGrade" varchar(5) NOT NULL, 
    "NewGrade" varchar(5) NOT NULL,
    PRIMARY KEY ("StudentId", "CourseId", "ChangeTime") 
);

-- Create function to record grade changes
CREATE OR REPLACE FUNCTION "School"."RecordIfGradeChanged"()
RETURNS trigger AS
$$
BEGIN
    IF NEW."Grade" <> OLD."Grade" THEN
        INSERT INTO "School"."StudentGradeHistory" (
            "StudentId",
            "CourseId",
            "ChangeTime",
            "CourseName",
            "OldGrade",
            "NewGrade")
        VALUES
            (OLD."StudentId",
             OLD."CourseId",
             now(),
             OLD."CourseName",
             OLD."Grade",
             NEW."Grade");
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER "TR_School_StudentGrade_AfterUpdate"
AFTER UPDATE
ON "School"."StudentGrade"
FOR EACH ROW
EXECUTE PROCEDURE "School"."RecordIfGradeChanged"();

-- Test with an update
UPDATE "School"."StudentGrade"
SET "Grade" = 'C'
WHERE "StudentId" = 1 AND "CourseId" = 1;

-- Check history
SELECT "StudentId",
       "ChangeTime",
       "CourseName",
       "OldGrade",
       "NewGrade"
FROM "School"."StudentGradeHistory";
```

**Sample Output:**
```
StudentId  ChangeTime                       CourseName  OldGrade  NewGrade
1          2023-11-15 15:42:37.268-05:00    Biology 2   F         C
```

### Automatically Classifying Temperatures

#### PostgreSQL Standard (snake_case)

```sql
-- Create temperature test table
CREATE TABLE temperature_test (
    station_name varchar(50),
    observation_date date,
    max_temp integer,
    min_temp integer,
    max_temp_group varchar(40),
    PRIMARY KEY (station_name, observation_date)
);

-- Create function to classify temperatures
CREATE OR REPLACE FUNCTION classify_max_temp()
RETURNS trigger AS
$$
BEGIN
    CASE
        WHEN NEW.max_temp >= 90 THEN
            NEW.max_temp_group := 'Hot';
        WHEN NEW.max_temp BETWEEN 70 AND 89 THEN
            NEW.max_temp_group := 'Warm';
        WHEN NEW.max_temp BETWEEN 50 AND 69 THEN
            NEW.max_temp_group := 'Pleasant';
        WHEN NEW.max_temp BETWEEN 33 AND 49 THEN
            NEW.max_temp_group := 'Cold';
        WHEN NEW.max_temp BETWEEN 20 AND 32 THEN
            NEW.max_temp_group := 'Freezing';
        ELSE NEW.max_temp_group := 'Inhumane';
    END CASE;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER temperature_insert
BEFORE INSERT
ON temperature_test
FOR EACH ROW
EXECUTE PROCEDURE classify_max_temp();

-- Test with sample data
INSERT INTO temperature_test (station_name, observation_date, max_temp, min_temp) 
VALUES
    ('North Station', '1/19/2019', 10, -3),
    ('North Station', '3/20/2019', 28, 19),
    ('North Station', '5/2/2019', 65, 42),
    ('North Station', '8/9/2019', 93, 74);

-- View results
SELECT * FROM temperature_test;
```

**Sample Output:**
```
station_name   observation_date  max_temp  min_temp  max_temp_group
North Station  2019-01-19        10        -3        Inhumane
North Station  2019-03-20        28        19        Freezing
North Station  2019-05-02        65        42        Pleasant
North Station  2019-08-09        93        74        Hot
```

#### Domain Driven Design (PascalCase)

```sql
-- Create temperature test table
CREATE TABLE "Weather"."TemperatureReading" (
    "StationName" varchar(50),
    "ObservationDate" date,
    "MaxTemperature" integer,
    "MinTemperature" integer,
    "TemperatureCategory" varchar(40),
    PRIMARY KEY ("StationName", "ObservationDate")
);

-- Create function to classify temperatures
CREATE OR REPLACE FUNCTION "Weather"."ClassifyMaxTemperature"()
RETURNS trigger AS
$$
BEGIN
    CASE
        WHEN NEW."MaxTemperature" >= 90 THEN
            NEW."TemperatureCategory" := 'Hot';
        WHEN NEW."MaxTemperature" BETWEEN 70 AND 89 THEN
            NEW."TemperatureCategory" := 'Warm';
        WHEN NEW."MaxTemperature" BETWEEN 50 AND 69 THEN
            NEW."TemperatureCategory" := 'Pleasant';
        WHEN NEW."MaxTemperature" BETWEEN 33 AND 49 THEN
            NEW."TemperatureCategory" := 'Cold';
        WHEN NEW."MaxTemperature" BETWEEN 20 AND 32 THEN
            NEW."TemperatureCategory" := 'Freezing';
        ELSE NEW."TemperatureCategory" := 'Inhumane';
    END CASE;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER "TR_Weather_TemperatureReading_BeforeInsert"
BEFORE INSERT
ON "Weather"."TemperatureReading"
FOR EACH ROW
EXECUTE PROCEDURE "Weather"."ClassifyMaxTemperature"();

-- Test with sample data
INSERT INTO "Weather"."TemperatureReading" ("StationName", "ObservationDate", "MaxTemperature", "MinTemperature") 
VALUES
    ('North Station', '1/19/2019', 10, -3),
    ('North Station', '3/20/2019', 28, 19),
    ('North Station', '5/2/2019', 65, 42),
    ('North Station', '8/9/2019', 93, 74);

-- View results
SELECT * FROM "Weather"."TemperatureReading";
```

**Sample Output:**
```
StationName    ObservationDate  MaxTemperature  MinTemperature  TemperatureCategory
North Station  2019-01-19       10              -3              Inhumane
North Station  2019-03-20       28              19              Freezing
North Station  2019-05-02       65              42              Pleasant
North Station  2019-08-09       93              74              Hot
```

## Wrapping Up

This chapter explored techniques for encapsulating SQL logic into reusable database objects:

1. **Views** provide an easy way to save and reuse queries, simplify access to data, and secure sensitive information.

2. **Functions** allow you to:
   - Create reusable code for common calculations (like percent_change)
   - Define procedural logic for data updates (like updating personal days)
   - Leverage multiple programming languages including SQL, PL/pgSQL, and Python

3. **Triggers** offer automatic execution of functions in response to database events, enabling:
   - Audit logging (like tracking grade changes)
   - Automatic data classification (like categorizing temperatures)
   - Data validation and maintenance

These techniques help follow the DRY programming principle and will increase your productivity by reducing repetitive tasks, preventing errors, and maintaining consistent data processing logic.

## Try It Yourself Exercises

1. Create a view that displays the number of New York City taxi trips per hour using the taxi data from Chapter 11.

2. Create a `rates_per_thousand()` function that takes three arguments to calculate the result: observed_number, base_number, and decimal_places.

3. Write a trigger that automatically adds an inspection date each time you insert a new facility into the meat_poultry_egg_inspect table, setting the date to six months from the current date.