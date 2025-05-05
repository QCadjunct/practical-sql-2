# Chapter 6 - JOINING TABLES IN A RELATIONAL DATABASE

## Introduction to Relational Databases

Edgar F. Codd's paper "A Relational Model of Data for Large Shared Data Banks" published in 1970 while working at IBM revolutionized database design and led to SQL development. The relational model allows building tables that:

- Eliminate duplicate data
- Are easier to maintain
- Provide increased flexibility in querying

In a relational database, each table typically holds data on one entity (students, cars, purchases, houses), with each row representing one entity instance. Table joins link rows in one table to rows in other tables.

## Linking Tables Using JOIN

### Basic JOIN Syntax

The fundamental syntax for joining tables connects tables in a query using the `JOIN...ON` statement:

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.table_a JOIN education.table_b
ON education.table_a.key_column = education.table_b.foreign_key_column
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."TableA" JOIN "Education"."TableB"
ON "Education"."TableA"."KeyColumn" = "Education"."TableB"."ForeignKeyColumn"
```

The `ON` clause typically matches based on equality between values, but you can use any expression that evaluates to Boolean results (true/false), such as:

**PostgreSQL Standard (snake_case)**:

```sql
ON education.table_a.key_column >= education.table_b.foreign_key_column
```

**Domain-Driven Design (PascalCase)**:

```sql
ON "Education"."TableA"."KeyColumn" >= "Education"."TableB"."ForeignKeyColumn"
```

## Relating Tables with Key Columns

### Primary and Foreign Keys

The chapter demonstrates a real-world scenario of tracking agency payroll spending by department using two tables: departments and employees.

**PostgreSQL Standard (snake_case)**:

```sql
CREATE TABLE payroll.departments (
    dept_id bigserial,
    dept varchar(100),
    city varchar(100),
    CONSTRAINT payroll_departments_pky PRIMARY KEY (dept_id),
    CONSTRAINT payroll_departments_dept_city_key UNIQUE (dept, city) 
);

CREATE TABLE payroll.employees (
    emp_id bigserial,
    first_name varchar(100),
    last_name varchar(100),
    salary integer,
    dept_id integer REFERENCES payroll.departments (dept_id),
    CONSTRAINT payroll_employees_pky PRIMARY KEY (emp_id),
    CONSTRAINT payroll_employees_emp_dept_key UNIQUE (emp_id, dept_id) 
);

INSERT INTO payroll.departments (dept, city)
VALUES
    ('Tax', 'Atlanta'),
    ('IT', 'Boston');

INSERT INTO payroll.employees (first_name, last_name, salary, dept_id) 
VALUES
    ('Nancy', 'Jones', 62500, 1),
    ('Lee', 'Smith', 59300, 1),
    ('Soo', 'Nguyen', 83000, 2),
    ('Janet', 'King', 95000, 2);
```

**Domain-Driven Design (PascalCase)**:

```sql
CREATE TABLE "Payroll"."Department" (
    "DepartmentId" bigserial,
    "Dept" varchar(100),
    "City" varchar(100),
    CONSTRAINT "PK_Payroll_Department" PRIMARY KEY ("DepartmentId"),
    CONSTRAINT "UQ_Payroll_Department_Dept_City" UNIQUE ("Dept", "City") 
);

CREATE TABLE "Payroll"."Employee" (
    "EmployeeId" bigserial,
    "FirstName" varchar(100),
    "LastName" varchar(100),
    "Salary" integer,
    "DepartmentId" integer REFERENCES "Payroll"."Department" ("DepartmentId"),
    CONSTRAINT "PK_Payroll_Employee" PRIMARY KEY ("EmployeeId"),
    CONSTRAINT "UQ_Payroll_Employee_EmployeeId_DepartmentId" UNIQUE ("EmployeeId", "DepartmentId") 
);

INSERT INTO "Payroll"."Department" ("Dept", "City")
VALUES
    ('Tax', 'Atlanta'),
    ('IT', 'Boston');

INSERT INTO "Payroll"."Employee" ("FirstName", "LastName", "Salary", "DepartmentId") 
VALUES
    ('Nancy', 'Jones', 62500, 1),
    ('Lee', 'Smith', 59300, 1),
    ('Soo', 'Nguyen', 83000, 2),
    ('Janet', 'King', 95000, 2);
```

After executing these statements, the database contains:

**departments table contents:**

| dept_id | dept | city    |
|---------|------|---------|
| 1       | Tax  | Atlanta |
| 2       | IT   | Boston  |

**employees table contents:**

| emp_id | first_name | last_name | salary | dept_id |
|--------|------------|-----------|--------|---------|
| 1      | Nancy      | Jones     | 62500  | 1       |
| 2      | Lee        | Smith     | 59300  | 1       |
| 3      | Soo        | Nguyen    | 83000  | 2       |
| 4      | Janet      | King      | 95000  | 2       |

### Key Concepts Explained

- **Primary Key**: A column or collection of columns whose values uniquely identify each row in a table. It must have a unique value for each row and cannot contain NULL values.
  
- **Foreign Key**: A column that refers to a primary key in another table. Foreign keys enforce referential integrity, requiring values to exist in the referenced primary key before being added to the foreign key column.

- **Unique Constraint**: Guarantees that values in a column or combination of columns are unique across all rows. In the examples above, this ensures no duplicate department-city pairs exist.

The relational model offers advantages over a single combined table:

- Avoids data repetition (department information is stored only once)
- Simplifies data management (updating a department name requires changing one row)
- Maintains data integrity through constraints

## Querying Multiple Tables Using JOIN

### Basic JOIN Query

To retrieve data from both tables simultaneously:

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM payroll.employees JOIN payroll.departments
ON payroll.employees.dept_id = payroll.departments.dept_id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Payroll"."Employee" JOIN "Payroll"."Department"
ON "Payroll"."Employee"."DepartmentId" = "Payroll"."Department"."DepartmentId";
```

**Output:**

| emp_id | first_name | last_name | salary | dept_id | dept_id | dept | city    |
|--------|------------|-----------|--------|---------|---------|------|---------|
| 1      | Nancy      | Jones     | 62500  | 1       | 1       | Tax  | Atlanta |
| 2      | Lee        | Smith     | 59300  | 1       | 1       | Tax  | Atlanta |
| 3      | Soo        | Nguyen    | 83000  | 2       | 2       | IT   | Boston  |
| 4      | Janet      | King      | 95000  | 2       | 2       | IT   | Boston  |

The result includes all columns from both tables where the dept_id values match.

## JOIN Types

SQL offers several types of joins to handle different data retrieval needs:

### JOIN (INNER JOIN)

Returns rows from both tables where matching values are found in the joined columns.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left JOIN education.schools_right
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" JOIN "Education"."SchoolRight"
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

To demonstrate JOIN types, the chapter creates two example tables:

**PostgreSQL Standard (snake_case)**:

```sql
CREATE TABLE education.schools_left (
    id integer CONSTRAINT education_schools_left_pky PRIMARY KEY,
    left_school varchar(30)
);

CREATE TABLE education.schools_right (
    id integer CONSTRAINT education_schools_right_pky PRIMARY KEY, 
    right_school varchar(30)
);

INSERT INTO education.schools_left (id, left_school) VALUES 
(1, 'Oak Street School'),
(2, 'Roosevelt High School'),
(5, 'Washington Middle School'),
(6, 'Jefferson High School');

INSERT INTO education.schools_right (id, right_school) VALUES 
(1, 'Oak Street School'),
(2, 'Roosevelt High School'),
(3, 'Morrison Elementary'),
(4, 'Chase Magnet Academy'),
(6, 'Jefferson High School');
```

**Domain-Driven Design (PascalCase)**:

```sql
CREATE TABLE "Education"."SchoolLeft" (
    "SchoolLeftId" integer CONSTRAINT "PK_Education_SchoolLeft" PRIMARY KEY,
    "LeftSchool" varchar(30)
);

CREATE TABLE "Education"."SchoolRight" (
    "SchoolRightId" integer CONSTRAINT "PK_Education_SchoolRight" PRIMARY KEY, 
    "RightSchool" varchar(30)
);

INSERT INTO "Education"."SchoolLeft" ("SchoolLeftId", "LeftSchool") VALUES 
(1, 'Oak Street School'),
(2, 'Roosevelt High School'),
(5, 'Washington Middle School'),
(6, 'Jefferson High School');

INSERT INTO "Education"."SchoolRight" ("SchoolRightId", "RightSchool") VALUES 
(1, 'Oak Street School'),
(2, 'Roosevelt High School'),
(3, 'Morrison Elementary'),
(4, 'Chase Magnet Academy'),
(6, 'Jefferson High School');
```

**Output from INNER JOIN:**

| id | left_school           | id | right_school          |
|----|----------------------|----|----------------------|
| 1  | Oak Street School    | 1  | Oak Street School    |
| 2  | Roosevelt High School| 2  | Roosevelt High School|
| 6  | Jefferson High School| 6  | Jefferson High School|

Only schools with IDs 1, 2, and 6 appear because they exist in both tables.

### LEFT JOIN

Returns every row from the left table plus matching rows from the right table.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left LEFT JOIN education.schools_right
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" LEFT JOIN "Education"."SchoolRight"
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

**Output:**

| id  | left_school             | id   | right_school           |
|-----|-------------------------|------|------------------------|
| 1   | Oak Street School       | 1    | Oak Street School      |
| 2   | Roosevelt High School   | 2    | Roosevelt High School  |
| 5   | Washington Middle School| NULL | NULL                   |
| 6   | Jefferson High School   | 6    | Jefferson High School  |

All rows from schools_left appear. Where there's no match (ID 5), NULL values appear for the right table columns.

### RIGHT JOIN

Returns every row from the right table plus matching rows from the left table.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left RIGHT JOIN education.schools_right
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" RIGHT JOIN "Education"."SchoolRight"
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

**Output:**

| id   | left_school            | id   | right_school          |
|------|------------------------|------|-----------------------|
| 1    | Oak Street School      | 1    | Oak Street School     |
| 2    | Roosevelt High School  | 2    | Roosevelt High School |
| NULL | NULL                   | 3    | Morrison Elementary   |
| NULL | NULL                   | 4    | Chase Magnet Academy  |
| 6    | Jefferson High School  | 6    | Jefferson High School |

All rows from schools_right appear. Where there's no match (IDs 3 and 4), NULL values appear for the left table columns.

### FULL OUTER JOIN

Returns every row from both tables with NULL values where matches don't exist.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left FULL OUTER JOIN education.schools_right
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" FULL OUTER JOIN "Education"."SchoolRight"
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

**Output:**

| id   | left_school             | right_id | right_school          |
|------|-------------------------|----------|-----------------------|
| 1    | Oak Street School       | 1        | Oak Street School     |
| 2    | Roosevelt High School   | 2        | Roosevelt High School |
| 5    | Washington Middle School| NULL     | NULL                  |
| 6    | Jefferson High School   | 6        | Jefferson High School |
| NULL | NULL                    | 3        | Morrison Elementary   |
| NULL | NULL                    | 4        | Chase Magnet Academy  |

All rows from both tables appear, with NULL values where matches don't exist.

### CROSS JOIN

Returns every possible combination of rows from both tables (Cartesian product).

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left CROSS JOIN education.schools_right;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" CROSS JOIN "Education"."SchoolRight";
```

**Output (first few rows):**

| id | left_school            | id | right_school           |
|----|------------------------|----|------------------------|
| 1  | Oak Street School      | 1  | Oak Street School      |
| 1  | Oak Street School      | 2  | Roosevelt High School  |
| 1  | Oak Street School      | 3  | Morrison Elementary    |
| 1  | Oak Street School      | 4  | Chase Magnet Academy   |
| 1  | Oak Street School      | 6  | Jefferson High School  |
| 2  | Roosevelt High School  | 1  | Oak Street School      |
| 2  | Roosevelt High School  | 2  | Roosevelt High School  |
| 2  | Roosevelt High School  | 3  | Morrison Elementary    |
| 2  | Roosevelt High School  | 4  | Chase Magnet Academy   |
| 2  | Roosevelt High School  | 6  | Jefferson High School  |

The result contains 20 rows (4 left table rows × 5 right table rows).

## Using NULL to Find Rows with Missing Values

NULL values in SQL represent missing or unknown data. You can use them to find rows that don't have matches in joined tables.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT *
FROM education.schools_left LEFT JOIN education.schools_right
ON education.schools_left.id = education.schools_right.id
WHERE education.schools_right.id IS NULL;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT *
FROM "Education"."SchoolLeft" LEFT JOIN "Education"."SchoolRight"
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId"
WHERE "Education"."SchoolRight"."SchoolRightId" IS NULL;
```

**Output:**

| id  | left_school             | right_id | right_school |
|-----|-------------------------|----------|--------------|
| 5   | Washington Middle School| NULL     | NULL         |

This query identifies schools that exist only in the left table.

## Three Types of Table Relationships

### One-to-One Relationship

In a one-to-one relationship, each row in the first table corresponds to exactly one row in the second table.

Example: Tables containing state census data where each state has exactly one row in each table.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT c.state_name, c.median_household_income, e.college_degree_percent
FROM census.income c JOIN census.education e
ON c.state_code = e.state_code;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT c."StateName", c."MedianHouseholdIncome", e."CollegeDegreePercent"
FROM "Census"."Income" c JOIN "Census"."Education" e
ON c."StateCode" = e."StateCode";
```

### One-to-Many Relationship

In a one-to-many relationship, one row in the first table corresponds to multiple rows in the second table.

Example: Automobile manufacturers (one) and their models (many).

**PostgreSQL Standard (snake_case)**:

```sql
SELECT m.manufacturer_name, c.model_name
FROM automotive.manufacturers m JOIN automotive.car_models c
ON m.manufacturer_id = c.manufacturer_id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT m."ManufacturerName", c."ModelName"
FROM "Automotive"."Manufacturer" m JOIN "Automotive"."CarModel" c
ON m."ManufacturerId" = c."ManufacturerId";
```

### Many-to-Many Relationship

In a many-to-many relationship, multiple rows in the first table correspond to multiple rows in the second table.

Example: Baseball players and field positions (a player can play multiple positions, and multiple players can play each position).

**PostgreSQL Standard (snake_case)**:

```sql
SELECT p.player_name, pos.position_name
FROM baseball.players p 
JOIN baseball.player_positions pp ON p.player_id = pp.player_id
JOIN baseball.positions pos ON pp.position_id = pos.position_id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT p."PlayerName", pos."PositionName"
FROM "Baseball"."Player" p 
JOIN "Baseball"."PlayerPosition" pp ON p."PlayerId" = pp."PlayerId"
JOIN "Baseball"."Position" pos ON pp."PositionId" = pos."PositionId";
```

## Selecting Specific Columns in a Join

When joining tables, you typically want to select specific columns rather than all columns with the asterisk wildcard.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT education.schools_left.id,
       education.schools_left.left_school,
       education.schools_right.right_school
FROM education.schools_left LEFT JOIN education.schools_right 
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT "Education"."SchoolLeft"."SchoolLeftId",
       "Education"."SchoolLeft"."LeftSchool",
       "Education"."SchoolRight"."RightSchool"
FROM "Education"."SchoolLeft" LEFT JOIN "Education"."SchoolRight" 
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

**Output:**

| id  | left_school             | right_school          |
|-----|-------------------------|-----------------------|
| 1   | Oak Street School       | Oak Street School     |
| 2   | Roosevelt High School   | Roosevelt High School |
| 5   | Washington Middle School| NULL                  |
| 6   | Jefferson High School   | Jefferson High School |

You can also use column aliases to rename columns in the output:

**PostgreSQL Standard (snake_case)**:

```sql
SELECT education.schools_left.id AS left_id,
       education.schools_left.left_school,
       education.schools_right.right_school
FROM education.schools_left LEFT JOIN education.schools_right 
ON education.schools_left.id = education.schools_right.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT "Education"."SchoolLeft"."SchoolLeftId" AS "LeftId",
       "Education"."SchoolLeft"."LeftSchool",
       "Education"."SchoolRight"."RightSchool"
FROM "Education"."SchoolLeft" LEFT JOIN "Education"."SchoolRight" 
ON "Education"."SchoolLeft"."SchoolLeftId" = "Education"."SchoolRight"."SchoolRightId";
```

## Simplifying JOIN Syntax with Table Aliases

Table aliases make JOIN queries more readable by providing shorthand references to table names.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT lt.id,
       lt.left_school,
       rt.right_school
FROM education.schools_left AS lt LEFT JOIN education.schools_right AS rt
ON lt.id = rt.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT lt."SchoolLeftId",
       lt."LeftSchool",
       rt."RightSchool"
FROM "Education"."SchoolLeft" AS lt LEFT JOIN "Education"."SchoolRight" AS rt
ON lt."SchoolLeftId" = rt."SchoolRightId";
```

**Output:**

| id | left_school             | right_school          |
|----|-------------------------|-----------------------|
| 1  | Oak Street School       | Oak Street School     |
| 2  | Roosevelt High School   | Roosevelt High School |
| 5  | Washington Middle School| NULL                  |
| 6  | Jefferson High School   | Jefferson High School |

The code is more concise and readable with table aliases.

## Joining Multiple Tables

You can join more than two tables in a single query by adding additional JOIN clauses.

**PostgreSQL Standard (snake_case)**:

```sql
-- Creating additional tables
CREATE TABLE education.schools_enrollment (
    id integer,
    enrollment integer
);

CREATE TABLE education.schools_grades (
    id integer,
    grades varchar(10)
);

INSERT INTO education.schools_enrollment (id, enrollment) 
VALUES
    (1, 360),
    (2, 1001),
    (5, 450),
    (6, 927);

INSERT INTO education.schools_grades (id, grades) 
VALUES
    (1, 'K-3'),
    (2, '9-12'),
    (5, '6-8'),
    (6, '9-12');

-- Joining three tables
SELECT lt.id, lt.left_school, en.enrollment, gr.grades
FROM education.schools_left AS lt 
LEFT JOIN education.schools_enrollment AS en ON lt.id = en.id
LEFT JOIN education.schools_grades AS gr ON lt.id = gr.id;
```

**Domain-Driven Design (PascalCase)**:

```sql
-- Creating additional tables
CREATE TABLE "Education"."SchoolEnrollment" (
    "SchoolEnrollmentId" integer,
    "Enrollment" integer
);

CREATE TABLE "Education"."SchoolGrade" (
    "SchoolGradeId" integer,
    "Grades" varchar(10)
);

INSERT INTO "Education"."SchoolEnrollment" ("SchoolEnrollmentId", "Enrollment") 
VALUES
    (1, 360),
    (2, 1001),
    (5, 450),
    (6, 927);

INSERT INTO "Education"."SchoolGrade" ("SchoolGradeId", "Grades") 
VALUES
    (1, 'K-3'),
    (2, '9-12'),
    (5, '6-8'),
    (6, '9-12');

-- Joining three tables
SELECT lt."SchoolLeftId", lt."LeftSchool", en."Enrollment", gr."Grades"
FROM "Education"."SchoolLeft" AS lt 
LEFT JOIN "Education"."SchoolEnrollment" AS en ON lt."SchoolLeftId" = en."SchoolEnrollmentId"
LEFT JOIN "Education"."SchoolGrade" AS gr ON lt."SchoolLeftId" = gr."SchoolGradeId";
```

**Output:**

| id  | left_school              | enrollment | grades |
|-----|--------------------------|------------|--------|
| 1   | Oak Street School        | 360        | K-3    |
| 2   | Roosevelt High School    | 1001       | 9-12   |
| 5   | Washington Middle School | 450        | 6-8    |
| 6   | Jefferson High School    | 927        | 9-12   |

The query joins schools_left with schools_enrollment and schools_grades using the id column, producing a comprehensive view of each school.

## Performing Math on Joined Table Columns

You can perform calculations using columns from joined tables. This is useful for analyzing trends over time.

**PostgreSQL Standard (snake_case)**:

```sql
SELECT c2010.geo_name,
       c2010.state_us_abbreviation AS state,
       c2010.p0010001 AS pop_2010,
       c2000.p0010001 AS pop_2000,
       c2010.p0010001 - c2000.p0010001 AS raw_change,
       round((CAST(c2010.p0010001 AS numeric(8,1)) - c2000.p0010001)
           / c2000.p0010001 * 100, 1) AS pct_change
FROM census.us_counties_2010 c2010 INNER JOIN census.us_counties_2000 c2000
ON c2010.state_fips = c2000.state_fips
AND c2010.county_fips = c2000.county_fips
AND c2010.p0010001 <> c2000.p0010001
ORDER BY pct_change DESC;
```

**Domain-Driven Design (PascalCase)**:

```sql
SELECT c2010."GeoName",
       c2010."StateUsAbbreviation" AS "State",
       c2010."P0010001" AS "Pop2010",
       c2000."P0010001" AS "Pop2000",
       c2010."P0010001" - c2000."P0010001" AS "RawChange",
       round((CAST(c2010."P0010001" AS numeric(8,1)) - c2000."P0010001")
           / c2000."P0010001" * 100, 1) AS "PctChange"
FROM "Census"."UsCounty2010" c2010 INNER JOIN "Census"."UsCounty2000" c2000
ON c2010."StateFips" = c2000."StateFips"
AND c2010."CountyFips" = c2000."CountyFips"
AND c2010."P0010001" <> c2000."P0010001"
ORDER BY "PctChange" DESC;
```

**Output (first few rows):**

| geo_name       | state | pop_2010 | pop_2000 | raw_change | pct_change |
|----------------|-------|----------|----------|------------|------------|
| Kendall County | IL    | 114736   | 54544    | 60192      | 110.4      |
| Pinal County   | AZ    | 375770   | 179727   | 196043     | 109.1      |
| Flagler County | FL    | 95696    | 49832    | 45864      | 92.0       |
| Lincoln County | SD    | 44828    | 24131    | 20697      | 85.8       |
| Loudoun County | VA    | 312311   | 169599   | 142712     | 84.1       |

This query:

1. Joins county census data from 2000 and 2010
2. Calculates the raw population change (2010 minus 2000)
3. Calculates the percentage change
4. Filters to include only counties where population changed
5. Orders results by percentage change in descending order

The results reveal that Kendall County, Illinois and Pinal County, Arizona more than doubled their populations in the decade.

## Try It Yourself Exercises

The chapter concludes with exercises to practice join techniques:

1. Identify counties that exist in only one of the us_counties_2010 or us_counties_2000 tables using appropriate joins and NULL values.

2. Calculate the median percent change in county population using the median() or percentile_cont() functions.

3. Identify which county had the greatest percentage population loss between 2000 and 2010; the hint suggests it may be related to a major weather event in 2005 (likely referring to Hurricane Katrina).

---

These exercises encourage practical application of the join concepts covered in the chapter, reinforcing skills in finding missing data, performing calculations across joined tables, and drawing insights from the results.
