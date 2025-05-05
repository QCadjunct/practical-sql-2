# Chapter 2 - BEGINNING DATA EXPLORATION WITH SELECT

## Table of Contents

- [Introduction](#introduction)
- [Basic SELECT Syntax](#basic-select-syntax)
- [Querying a Subset of Columns](#querying-a-subset-of-columns)
- [Using DISTINCT to Find Unique Values](#using-distinct-to-find-unique-values)
- [Sorting Data with ORDER BY](#sorting-data-with-order-by)
- [Filtering Rows with WHERE](#filtering-rows-with-where)
- [Using LIKE and ILIKE with WHERE](#using-like-and-ilike-with-where)
- [Combining Operators with AND and OR](#combining-operators-with-and-and-or)
- [Putting It All Together](#putting-it-all-together)
- [Try It Yourself Exercises](#try-it-yourself-exercises)

## Introduction

This chapter introduces the concept of "interviewing" your data through SQL queries as a means of discovering data quality issues and uncovering meaningful patterns. Using the SELECT statement, you can retrieve and analyze information from database tables to reveal insights into your data. The author metaphorically compares this process to interviewing a job candidate - you ask strategic questions to determine whether the reality matches what's presented on the surface.

Data exploration begins with understanding what information is available, identifying quality issues such as inconsistencies or missing values, and incrementally building more complex queries to discover relationships and patterns. The SELECT statement is the foundation of this exploration process, with capabilities ranging from simple retrievals to complex filtering and sorting.

[Back to Table of Contents](#table-of-contents)

## Basic SELECT Syntax

The most fundamental SQL query uses the SELECT statement to retrieve rows and columns from a table:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Retrieving all teacher information from the education system database.

```sql
-- Business context: Education department employee records
-- Predicate: Domain(Teacher) → Teacher ∈ Education
SELECT * FROM "Education"."Teacher";
```

When executed on the "Education"."Teacher" table, this returns all rows and columns:

| Id | FirstName | LastName | School             | HireDate   | Salary |
|----|-----------|----------|-------------------|------------|--------|
| 1  | Janet     | Smith    | F.D. Roosevelt HS | 2011-10-30 | 36200  |
| 2  | Lee       | Reynolds | F.D. Roosevelt HS | 1993-05-22 | 65000  |
| 3  | Samuel    | Cole     | Myers Middle School| 2005-08-01 | 43500  |
| 4  | Samantha  | Bush     | Myers Middle School| 2011-10-30 | 36200  |
| 5  | Betty     | Diaz     | Myers Middle School| 2005-08-30 | 43500  |
| 6  | Kathleen  | Roush    | F.D. Roosevelt HS | 2010-10-22 | 38500  |

### PostgreSQL Standard Implementation

```sql
SELECT * FROM teachers;
```

### Implementation Analysis

The SELECT statement consists of:
- The SELECT keyword that initiates the query
- An asterisk (*) wildcard that represents "select all columns"
- The FROM keyword indicating which table to retrieve data from
- A semicolon (;) that terminates the statement

The result set contains all rows and columns from the table. The "Id" column (of type bigserial) automatically increments with each new row, acting as a unique identifier for each record without requiring explicit insertion.

[Back to Table of Contents](#table-of-contents)

## Querying a Subset of Columns

Instead of retrieving all columns with the asterisk wildcard, you can specify exactly which columns you want:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Analyzing teacher compensation by focusing only on names and salary information.

```sql
-- Business context: Teacher salary review process
-- Predicate: ∀t ∈ Teacher: HasProperties(t, {LastName, FirstName, Salary})
SELECT "LastName", "FirstName", "Salary" 
FROM "Education"."Teacher";
```

This returns only the specified columns:

| LastName | FirstName | Salary |
|----------|-----------|--------|
| Smith    | Janet     | 36200  |
| Reynolds | Lee       | 65000  |
| Cole     | Samuel    | 43500  |
| Bush     | Samantha  | 36200  |
| Diaz     | Betty     | 43500  |
| Roush    | Kathleen  | 38500  |

### PostgreSQL Standard Implementation

```sql
SELECT last_name, first_name, salary FROM teachers;
```

### Implementation Analysis

- This approach improves performance with large datasets by retrieving only necessary columns
- You can specify columns in any order, regardless of their order in the table structure
- Column selection helps focus analysis on relevant data points
- For large tables with many columns, this reduces network traffic and resource usage
- This technique is especially valuable when working with tables containing BLOB or TEXT columns

[Back to Table of Contents](#table-of-contents)

## Using DISTINCT to Find Unique Values

The DISTINCT keyword eliminates duplicate values, showing only unique entries in a column or combination of columns:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Identifying all unique schools in the educational system.

```sql
-- Business context: School facility inventory
-- Predicate: ∀s1,s2 ∈ (π School (Teacher)): s1 = s2 → Same school
SELECT DISTINCT "School"
FROM "Education"."Teacher";
```

This returns each unique school name just once:

| School             |
|--------------------|
| F.D. Roosevelt HS  |
| Myers Middle School|

### Domain Driven Database Design Implementation with Multiple Columns

**Business Context:** Analyzing the range of salary levels at each school.

```sql
-- Business context: Salary structure analysis by school
-- Predicate: ∀(s1,p1),(s2,p2) ∈ Result: (s1=s2 ∧ p1=p2) → Same row (no duplicates)
SELECT DISTINCT "School", "Salary"
FROM "Education"."Teacher";
```

This returns each unique school-salary combination:

| School              | Salary |
|---------------------|--------|
| Myers Middle School | 43500  |
| Myers Middle School | 36200  |
| F.D. Roosevelt HS   | 65000  |
| F.D. Roosevelt HS   | 38500  |
| F.D. Roosevelt HS   | 36200  |

### PostgreSQL Standard Implementation

```sql
-- For single column
SELECT DISTINCT school
FROM teachers;

-- For multiple columns
SELECT DISTINCT school, salary
FROM teachers;
```

### Implementation Analysis

- DISTINCT is a powerful tool for understanding the range of values in a column
- For data quality assessment, it helps identify:
  - Inconsistent spelling (e.g., "Myers Middle School" vs "Myer's Middle School")
  - Unexpected values that may indicate data entry errors
  - The overall distribution of values across a dataset
- When used with multiple columns, DISTINCT helps answer questions like:
  - "For each school, what salary levels exist?"
  - "For each product, what colors are available?"
  - "For each department, which job titles are present?"
- The SQL projection operation (π) is represented through column selection

[Back to Table of Contents](#table-of-contents)

## Sorting Data with ORDER BY

The ORDER BY clause arranges query results in a specified sequence, making patterns more apparent:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Identifying highest-paid teachers for budget review.

```sql
-- Business context: Compensation review process
-- Predicate: ∀t1,t2 ∈ Result: Position(t1) < Position(t2) → t1.Salary > t2.Salary
SELECT "FirstName", "LastName", "Salary"
FROM "Education"."Teacher"
ORDER BY "Salary" DESC;
```

The result shows teachers sorted from highest to lowest salary:

| FirstName | LastName | Salary |
|-----------|----------|--------|
| Lee       | Reynolds | 65000  |
| Samuel    | Cole     | 43500  |
| Betty     | Diaz     | 43500  |
| Kathleen  | Roush    | 38500  |
| Janet     | Smith    | 36200  |
| Samantha  | Bush     | 36200  |

### Domain Driven Database Design Implementation with Multiple Sort Columns

**Business Context:** Displaying newest teachers at each school.

```sql
-- Business context: Recent hire analysis by school
-- Predicates: 
-- Primary sort: ∀t1,t2 ∈ Result: Position(t1) < Position(t2) → t1.School < t2.School (alphabetically)
-- Secondary sort: ∀t1,t2 ∈ Result: (t1.School = t2.School ∧ Position(t1) < Position(t2)) → t1.HireDate > t2.HireDate
SELECT "LastName", "School", "HireDate"
FROM "Education"."Teacher"
ORDER BY "School" ASC, "HireDate" DESC;
```

This groups teachers by school (alphabetically) and then sorts them by hire date (newest first):

| LastName | School              | HireDate   |
|----------|---------------------|------------|
| Smith    | F.D. Roosevelt HS   | 2011-10-30 |
| Roush    | F.D. Roosevelt HS   | 2010-10-22 |
| Reynolds | F.D. Roosevelt HS   | 1993-05-22 |
| Bush     | Myers Middle School | 2011-10-30 |
| Diaz     | Myers Middle School | 2005-08-30 |
| Cole     | Myers Middle School | 2005-08-01 |

### PostgreSQL Standard Implementation

```sql
-- Single column sorting
SELECT first_name, last_name, salary
FROM teachers
ORDER BY salary DESC;

-- Multiple column sorting
SELECT last_name, school, hire_date
FROM teachers
ORDER BY school ASC, hire_date DESC;
```

### Implementation Analysis

- The ORDER BY clause does not change the original table, only the presentation of query results
- Default sort direction is ascending (ASC); descending requires explicit DESC keyword
- Multiple column sorting creates logical groupings with ordered values within each group
- Text sorting depends on collation settings of the database (locale and character set)
- In UTF-8 sorting, characters generally sort in this order:
  1. Punctuation marks (quotes, parentheses, math operators)
  2. Numbers 0-9
  3. Additional punctuation (question mark)
  4. Capital letters A-Z
  5. More punctuation (brackets, underscore)
  6. Lowercase letters a-z
  7. Additional punctuation, special characters, extended alphabet
- Best practice: Limit ORDER BY to 2-3 columns for understandable results
- For complex analysis, run multiple focused queries rather than a single complex one

[Back to Table of Contents](#table-of-contents)

## Filtering Rows with WHERE

The WHERE clause allows you to limit query results to only those rows matching specific criteria:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Identifying teachers at a specific school location.

```sql
-- Business context: School-specific staffing report
-- Predicate: ∀t ∈ Result: t.School = 'Myers Middle School'
SELECT "LastName", "School", "HireDate" 
FROM "Education"."Teacher"
WHERE "School" = 'Myers Middle School';
```

This returns only teachers from Myers Middle School:

| LastName | School              | HireDate   |
|----------|---------------------|------------|
| Cole     | Myers Middle School | 2005-08-01 |
| Bush     | Myers Middle School | 2011-10-30 |
| Diaz     | Myers Middle School | 2005-08-30 |

### PostgreSQL Standard Implementation

```sql
SELECT last_name, school, hire_date 
FROM teachers
WHERE school = 'Myers Middle School';
```

### Comparison Operators

PostgreSQL provides various comparison operators for filtering:

| Operator | Function | Example |
|----------|----------|---------|
| = | Equal to | `WHERE school = 'Baker Middle'` |
| <> or != | Not equal to | `WHERE school <> 'Baker Middle'` |
| > | Greater than | `WHERE salary > 20000` |
| < | Less than | `WHERE salary < 60500` |
| >= | Greater than or equal to | `WHERE salary >= 20000` |
| <= | Less than or equal to | `WHERE salary <= 60500` |
| BETWEEN | Within a range | `WHERE salary BETWEEN 20000 AND 40000` |
| IN | Match one of a set of values | `WHERE last_name IN ('Bush', 'Roush')` |
| LIKE | Match a pattern (case sensitive) | `WHERE first_name LIKE 'Sam%'` |
| ILIKE | Match a pattern (case insensitive) | `WHERE first_name ILIKE 'sam%'` |
| NOT | Negates a condition | `WHERE first_name NOT ILIKE 'sam%'` |

### Domain Driven Database Design Implementation Examples

**Example 1: Finding teachers named Janet**

```sql
-- Business context: Locating specific employee records
-- Predicate: ∀t ∈ Result: t.FirstName = 'Janet'
SELECT "FirstName", "LastName", "School"
FROM "Education"."Teacher"
WHERE "FirstName" = 'Janet';
```

**Example 2: Excluding a specific school**

```sql
-- Business context: Reporting on non-high school teachers
-- Predicate: ∀t ∈ Result: t.School ≠ 'F.D. Roosevelt HS'
SELECT "School"
FROM "Education"."Teacher"
WHERE "School" != 'F.D. Roosevelt HS';
```

**Example 3: Teachers hired before 2000**

```sql
-- Business context: Long-term employee analysis
-- Predicate: ∀t ∈ Result: t.HireDate < '2000-01-01'
SELECT "FirstName", "LastName", "HireDate"
FROM "Education"."Teacher"
WHERE "HireDate" < '2000-01-01';
```

**Example 4: Teachers earning $43,500 or more**

```sql
-- Business context: Upper salary band review
-- Predicate: ∀t ∈ Result: t.Salary ≥ 43500
SELECT "FirstName", "LastName", "Salary"
FROM "Education"."Teacher"
WHERE "Salary" >= 43500;
```

**Example 5: Salary range analysis**

```sql
-- Business context: Middle salary band analysis
-- Predicate: ∀t ∈ Result: t.Salary ≥ 40000 ∧ t.Salary ≤ 65000
SELECT "FirstName", "LastName", "School", "Salary"
FROM "Education"."Teacher"
WHERE "Salary" BETWEEN 40000 AND 65000;
```

### PostgreSQL Standard Implementation Examples

```sql
-- Finding teachers named Janet
SELECT first_name, last_name, school
FROM teachers
WHERE first_name = 'Janet';

-- Excluding a specific school
SELECT school
FROM teachers
WHERE school != 'F.D. Roosevelt HS';

-- Teachers hired before 2000
SELECT first_name, last_name, hire_date
FROM teachers
WHERE hire_date < '2000-01-01';

-- Teachers earning $43,500 or more
SELECT first_name, last_name, salary
FROM teachers
WHERE salary >= 43500;

-- Salary range analysis
SELECT first_name, last_name, school, salary
FROM teachers
WHERE salary BETWEEN 40000 AND 65000;
```

### Implementation Analysis

- The WHERE clause acts as a filter, removing rows that don't match specified criteria
- In predicate logic, each WHERE condition is a predicate that evaluates to true or false for each row
- Comparison operators work with various data types:
  - Strings are typically compared lexicographically (dictionary order)
  - Dates compare chronologically
  - Numbers compare numerically
- BETWEEN is inclusive, including values at both ends of the range
- The equals operator requires exact matches including character case
- Comparison with dates requires proper date format (PostgreSQL accepts ISO date format yyyy-mm-dd)

[Back to Table of Contents](#table-of-contents)

## Using LIKE and ILIKE with WHERE

LIKE and ILIKE operators provide pattern matching for text fields using wildcards:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Finding teachers with similar first names for a database consolidation project.

```sql
-- Business context: Name standardization in employee records
-- Predicate (case sensitive): ∀t ∈ Result: t.FirstName matches pattern 'sam*'
SELECT "FirstName"
FROM "Education"."Teacher"
WHERE "FirstName" LIKE 'Sam%';

-- Predicate (case insensitive): ∀t ∈ Result: t.FirstName matches pattern 'sam*' (any case)
SELECT "FirstName"
FROM "Education"."Teacher"
WHERE "FirstName" ILIKE 'sam%';
```

The case-insensitive query (ILIKE) returns:

| FirstName |
|-----------|
| Samuel    |
| Samantha  |

### PostgreSQL Standard Implementation

```sql
-- Case sensitive (returns no results)
SELECT first_name
FROM teachers
WHERE first_name LIKE 'sam%';

-- Case insensitive (returns Samuel and Samantha)
SELECT first_name
FROM teachers
WHERE first_name ILIKE 'sam%';
```

### Pattern Matching Wildcards

- **Percent sign (%)**: Matches zero or more characters
- **Underscore (_)**: Matches exactly one character

Examples for matching the word "baker":
- `LIKE 'b%'` (starts with b)
- `LIKE '%ak%'` (contains "ak")
- `LIKE '_aker'` (any single character followed by "aker")
- `LIKE 'ba_er'` (ba, then any single character, then er)

### Implementation Analysis

- LIKE is ANSI SQL standard and is case-sensitive
- ILIKE is PostgreSQL-specific and is case-insensitive
- Using ILIKE is recommended for text searches to avoid missing results due to case variations
- Case-insensitive searching helps identify data quality issues with inconsistent capitalization
- Pattern matching can be performance-intensive on large datasets; indexes can improve performance
- Wildcards at the beginning of patterns (e.g., '%abc') typically cannot utilize standard B-tree indexes

[Back to Table of Contents](#table-of-contents)

## Combining Operators with AND and OR

Multiple conditions can be combined with AND and OR operators, along with parentheses for complex logic:

### Domain Driven Database Design Implementation with Predicate Logic

**Example 1: Teachers at specific school with salary below threshold**

```sql
-- Business context: Budget constraints at specific location
-- Predicate: ∀t ∈ Result: t.School = 'Myers Middle School' ∧ t.Salary < 40000
SELECT *
FROM "Education"."Teacher"
WHERE "School" = 'Myers Middle School'
AND "Salary" < 40000;
```

**Example 2: Teachers with specific last names**

```sql
-- Business context: Follow-up on specific employees
-- Predicate: ∀t ∈ Result: t.LastName = 'Cole' ∨ t.LastName = 'Bush'
SELECT *
FROM "Education"."Teacher"
WHERE "LastName" = 'Cole'
OR "LastName" = 'Bush';
```

**Example 3: Complex condition with parentheses**

```sql
-- Business context: Identifying salary outliers at specific school
-- Predicate: ∀t ∈ Result: t.School = 'F.D. Roosevelt HS' ∧ (t.Salary < 38000 ∨ t.Salary > 40000)
SELECT *
FROM "Education"."Teacher"
WHERE "School" = 'F.D. Roosevelt HS'
AND ("Salary" < 38000 OR "Salary" > 40000);
```

### PostgreSQL Standard Implementation

```sql
-- Example 1: Teachers at specific school with salary below threshold
SELECT *
FROM teachers
WHERE school = 'Myers Middle School'
AND salary < 40000;

-- Example 2: Teachers with specific last names
SELECT *
FROM teachers
WHERE last_name = 'Cole'
OR last_name = 'Bush';

-- Example 3: Complex condition with parentheses
SELECT *
FROM teachers
WHERE school = 'F.D. Roosevelt HS'
AND (salary < 38000 OR salary > 40000);
```

### Implementation Analysis

- The AND operator requires both conditions to be true
- The OR operator requires at least one condition to be true
- Parentheses control the order of evaluation and establish logical grouping
- In predicate logic:
  - AND corresponds to logical conjunction (∧)
  - OR corresponds to logical disjunction (∨)
  - Parenthesized expressions are evaluated first
- Without parentheses, SQL follows standard operator precedence (AND before OR)
- Complex conditions can express sophisticated business rules and data filtering requirements

[Back to Table of Contents](#table-of-contents)

## Putting It All Together

SQL has a specific structure for queries, with keywords appearing in a particular order:

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Analyzing teacher hiring and compensation trends at Roosevelt High School.

```sql
-- Business context: Temporal analysis of teacher compensation at specific school
-- Predicates:
-- Filter: ∀t ∈ Result: t.School contains 'Roos'
-- Sort: ∀t1,t2 ∈ Result: Position(t1) < Position(t2) → t1.HireDate > t2.HireDate
SELECT "FirstName", "LastName", "School", "HireDate", "Salary"
FROM "Education"."Teacher"
WHERE "School" LIKE '%Roos%'
ORDER BY "HireDate" DESC;
```

This returns teachers at Roosevelt High School, ordered from newest hire to earliest:

| FirstName | LastName  | School            | HireDate   | Salary |
|-----------|-----------|-------------------|------------|--------|
| Janet     | Smith     | F.D. Roosevelt HS | 2011-10-30 | 36200  |
| Kathleen  | Roush     | F.D. Roosevelt HS | 2010-10-22 | 38500  |
| Lee       | Reynolds  | F.D. Roosevelt HS | 1993-05-22 | 65000  |

### PostgreSQL Standard Implementation

```sql
SELECT first_name, last_name, school, hire_date, salary
FROM teachers
WHERE school LIKE '%Roos%'
ORDER BY hire_date DESC;
```

### SQL Statement Structure

```
SELECT column_names
FROM table_name
WHERE criteria
ORDER BY column_names;
```

### Implementation Analysis

- The SELECT statement has a required sequence of clauses:
  1. SELECT clause (required): Specifies which columns to return
  2. FROM clause (required): Identifies the data source (table)
  3. WHERE clause (optional): Filters rows based on conditions
  4. ORDER BY clause (optional): Sorts the result set
- The results reveal a correlation between hire date and salary level (earlier hires generally have higher salaries)
- This query demonstrates how combining selection, filtering, and sorting can answer specific business questions
- The pattern-matching filter (%Roos%) would also match variations like "Roosevelt" or "Roosville"

[Back to Table of Contents](#table-of-contents)

## Try It Yourself Exercises

### Exercise 1: Teacher List by School

**Business Context:** The school district superintendent needs a teacher directory organized by school.

```sql
-- Business context: District-wide teacher directory
-- Predicates:
-- Primary sort: ∀t1,t2 ∈ Result: Position(t1) < Position(t2) → t1.School < t2.School (alphabetically)
-- Secondary sort: ∀t1,t2 ∈ Result: (t1.School = t2.School ∧ Position(t1) < Position(t2)) → t1.LastName < t2.LastName
SELECT "School", "FirstName", "LastName"
FROM "Education"."Teacher"
ORDER BY "School" ASC, "LastName" ASC;
```

### Exercise 2: Finding Specific Teacher

**Business Context:** Identifying high-earning teachers with names beginning with a specific letter.

```sql
-- Business context: Targeted compensation review
-- Predicate: ∀t ∈ Result: t.FirstName LIKE 'S%' ∧ t.Salary > 40000
SELECT *
FROM "Education"."Teacher"
WHERE "FirstName" LIKE 'S%'
AND "Salary" > 40000;
```

### Exercise 3: Ranking Recent Hires by Salary

**Business Context:** Analyzing compensation structure for recently hired teachers.

```sql
-- Business context: Recent hire compensation analysis
-- Predicates:
-- Filter: ∀t ∈ Result: t.HireDate ≥ '2010-01-01'
-- Sort: ∀t1,t2 ∈ Result: Position(t1) < Position(t2) → t1.Salary > t2.Salary
SELECT "FirstName", "LastName", "Salary", "HireDate"
FROM "Education"."Teacher"
WHERE "HireDate" >= '2010-01-01'
ORDER BY "Salary" DESC;
```

### PostgreSQL Standard Implementations

```sql
-- Exercise 1
SELECT school, first_name, last_name
FROM teachers
ORDER BY school ASC, last_name ASC;

-- Exercise 2
SELECT *
FROM teachers
WHERE first_name LIKE 'S%'
AND salary > 40000;

-- Exercise 3
SELECT first_name, last_name, salary, hire_date
FROM teachers
WHERE hire_date >= '2010-01-01'
ORDER BY salary DESC;
```

### Implementation Analysis

These exercises demonstrate practical applications of the SELECT statement concepts:

- Exercise 1 creates a hierarchical report with primary and secondary sorting
- Exercise 2 combines pattern matching and numeric comparison to find specific records
- Exercise 3 shows how temporal filtering and sorting can reveal compensation patterns
- Each query answers a specific business question through the methodical application of SQL clauses

[Back to Table of Contents](#table-of-contents)