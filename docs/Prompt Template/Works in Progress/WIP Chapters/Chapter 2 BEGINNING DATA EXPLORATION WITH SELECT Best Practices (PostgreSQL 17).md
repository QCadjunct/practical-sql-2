
Here is the corrected Markdown:

```markdown
# Chapter 2 BEGINNING DATA EXPLORATION WITH SELECT Best Practices (PostgreSQL 17)

This guide covers the fundamentals of retrieving data from PostgreSQL tables using the `SELECT` statement, based on the concepts introduced in Chapter 2 of "Practical SQL". We'll emphasize best practices relevant to PostgreSQL 17, including naming conventions and schema organization.

## 📚 Table of Contents

*   [Introduction: Interviewing Your Data](#introduction-interviewing-your-data)
*   [Setting the Stage: Schema and Table Naming Conventions](#setting-the-stage-schema-and-table-naming-conventions)
*   [🏷️ Basic SELECT Syntax: Retrieving All Data](#️-basic-select-syntax-retrieving-all-data)
*   [🎯 Querying a Subset of Columns](#-querying-a-subset-of-columns)
*   [🔍 Using DISTINCT to Find Unique Values](#-using-distinct-to-find-unique-values)
*   [📊 Sorting Data with ORDER BY](#-sorting-data-with-order-by)
    *   [Understanding Text Sorting (Collation)](#understanding-text-sorting-collation)
*   [⚙️ Filtering Rows with WHERE](#️-filtering-rows-with-where)
    *   [Comparison Operators](#comparison-operators)
    *   [Pattern Matching with LIKE and ILIKE](#pattern-matching-with-like-and-ilike)
    *   [Combining Operators with AND and OR](#combining-operators-with-and-and-or)
*   [✨ Putting It All Together: The Standard SELECT Structure](#-putting-it-all-together-the-standard-select-structure)
*   [✅ Best Practices Summary](#-best-practices-summary)
*   [✍️ Hands-on Exercises](#️-hands-on-exercises)

## Introduction: Interviewing Your Data

Think of exploring data with SQL as conducting an interview. You start with broad questions to get a feel for the subject (your data) and then ask more specific questions to uncover details, verify information, and understand the story the data tells. The primary tool for this interview in SQL is the `SELECT` statement. It allows you to ask your database questions and retrieve specific pieces of information from your tables.

This initial exploration helps you understand data quality (Is it clean or dirty? Complete or missing values?), data range, and potential patterns or anomalies.

[Back to TOC](#-table-of-contents)

## Setting the Stage: Schema and Table Naming Conventions

Before we dive into `SELECT` statements, let's establish our environment and naming conventions, crucial for maintainable databases, especially in PostgreSQL 17.

**Schemas:** PostgreSQL uses schemas to organize database objects (like tables, functions, etc.) within a database. Using schemas avoids naming conflicts and groups related objects. We'll use a schema named `school_district` (snake_case) or `"SchoolDistrict"` (PascalCase).

**Table Naming:** Best practice dictates using singular nouns for table names, as a table defines the structure for a single entity type (e.g., `teacher`, not `teachers`).

**Naming Conventions:** We will demonstrate two common conventions:
1.  **snake_case:** All lowercase, words separated by underscores (e.g., `school_district`, `teacher`, `first_name`, `hire_date`). This is very common in the PostgreSQL community.
2.  **PascalCase:** Each word capitalized, no separators (e.g., `"SchoolDistrict"`, `"Teacher"`, `"FirstName"`, `"HireDate"`). Requires double quotes in PostgreSQL to preserve capitalization.

**Fully Qualified Names:** It's best practice to always refer to tables using their fully qualified name: `schema_name.table_name` or `"SchemaName"."TableName"`.

**Example Table Definition Context:**
While Chapter 2 focuses on *querying*, imagine the `teacher` table was created within the `school_district` schema like this (conceptual - showing relevant columns from the PDF):

*   **snake_case:**
    ```sql
    -- Conceptual Table Definition (snake_case)
    CREATE TABLE school_district.teacher (
        id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- PostgreSQL standard identity
        first_name text,
        last_name text,
        school text,
        hire_date date,
        salary numeric
        -- Constraint naming example: pky_teacher (implicitly named above)
        -- Or explicitly: CONSTRAINT pky_teacher PRIMARY KEY (id)
    );
    ```
*   **PascalCase:**
    ```sql
    -- Conceptual Table Definition (PascalCase)
    CREATE TABLE "SchoolDistrict"."Teacher" (
        "Id" bigint GENERATED ALWAYS AS IDENTITY CONSTRAINT "PkyTeacher" PRIMARY KEY, -- Explicit constraint name
        "FirstName" text,
        "LastName" text,
        "School" text,
        "HireDate" date,
        "Salary" numeric
    );
    ```
***Note:*** *PostgreSQL 17 encourages `GENERATED ALWAYS AS IDENTITY` over the older `bigserial` type for auto-incrementing primary keys.*

We will assume data similar to that shown in the PDF has been inserted into this `teacher` table within the appropriate schema for our examples.

[Back to TOC](#-table-of-contents)

## 🏷️ Basic SELECT Syntax: Retrieving All Data

The simplest way to start interviewing your data is to ask for everything in a table. The `SELECT *` statement retrieves all columns and all rows from the specified table.

**Syntax:**
```sql
SELECT * FROM schema_name.table_name;
```

**Analogy:** Asking the interviewee, "Tell me everything about your work history." It's broad but gives you a complete overview to start.

**Use Case:** Useful for initial exploration of small tables or when you genuinely need every piece of data. However, it's generally **not recommended** for production code or large tables due to performance implications (fetching unnecessary data) and potential issues if the table structure changes.

**Examples:**

*   **snake_case:**
    ```sql
    -- Retrieve all columns and rows from the teacher table
    SELECT * FROM school_district.teacher;
    ```
*   **PascalCase:**
    ```sql
    -- Retrieve all columns and rows from the Teacher table
    SELECT * FROM "SchoolDistrict"."Teacher";
    ```

**Output (Conceptual, based on PDF data):**
The output would show all columns (`id`, `first_name`, `last_name`, `school`, `hire_date`, `salary`) and all rows for the teachers. The `id` (or `Id`) column automatically populates with unique, sequential integers thanks to `GENERATED ALWAYS AS IDENTITY`.

[Back to TOC](#-table-of-contents)

## 🎯 Querying a Subset of Columns

Often, you only need specific pieces of information. Instead of the wildcard (`*`), list the column names you want, separated by commas.

**Syntax:**
```sql
SELECT column1, column2 FROM schema_name.table_name;
```

**Analogy:** Asking the interviewee, "Tell me specifically about your previous job title and salary."

**Use Case:** Standard practice for most queries. Improves clarity, reduces data transfer, and makes queries less likely to break if unrelated columns are added or removed from the table. You can also specify the order in which columns appear in the result set.

**Examples:**

*   **snake_case:**
    ```sql
    -- Retrieve only last name, first name, and salary
    SELECT last_name, first_name, salary
    FROM school_district.teacher;
    ```
*   **PascalCase:**
    ```sql
    -- Retrieve only LastName, FirstName, and Salary
    SELECT "LastName", "FirstName", "Salary"
    FROM "SchoolDistrict"."Teacher";
    ```

**Output (Conceptual):**
The result set would only contain the `last_name` (`LastName`), `first_name` (`FirstName`), and `salary` (`Salary`) columns for all teachers.

[Back to TOC](#-table-of-contents)

## 🔍 Using DISTINCT to Find Unique Values

The `DISTINCT` keyword eliminates duplicate rows from your result set, showing only unique values or unique combinations of values for the selected column(s).

**Syntax:**
```sql
SELECT DISTINCT column1 [, column2, ...] FROM schema_name.table_name;
```

**Analogy:** Asking for a list of all the *different* job titles held by employees in a department, rather than listing the title for every single employee.

**Use Case:** Identifying the unique entries in a column (e.g., all distinct school names, product categories, status codes). Essential for understanding the range of data and identifying potential inconsistencies (e.g., "F.D. Roosevelt HS" vs. "FD Roosevelt HS").

**Examples:**

1.  **Distinct values in a single column:**
    *   **snake_case:**
        ```sql
        -- Find the unique school names
        SELECT DISTINCT school
        FROM school_district.teacher;
        ```
    *   **PascalCase:**
        ```sql
        -- Find the unique school names
        SELECT DISTINCT "School"
        FROM "SchoolDistrict"."Teacher";
        ```
    *   **Output:** `F.D. Roosevelt HS`, `Myers Middle School`

2.  **Distinct combinations of values in multiple columns:**
    *   **snake_case:**
        ```sql
        -- Find the unique combinations of school and salary
        SELECT DISTINCT school, salary
        FROM school_district.teacher;
        ```
    *   **PascalCase:**
        ```sql
        -- Find the unique combinations of School and Salary
        SELECT DISTINCT "School", "Salary"
        FROM "SchoolDistrict"."Teacher";
        ```
    *   **Output:** Shows each unique pair. Since two teachers at Myers Middle School earn 43500, that pair appears only once. (Returns 5 rows instead of 6).

**Pitfall:** Using `DISTINCT` on columns with many unique values (like primary keys or timestamps) might not be meaningful and can impact performance on very large tables.

[Back to TOC](#-table-of-contents)

## 📊 Sorting Data with ORDER BY

The `ORDER BY` clause sorts the rows in your result set based on the values in one or more columns.

**Syntax:**
```sql
SELECT column1, column2
FROM schema_name.table_name
ORDER BY sort_column1 [ASC | DESC] [, sort_column2 [ASC | DESC], ...];
```

*   `ASC`: Ascending order (A-Z, 0-9). This is the default if omitted.
*   `DESC`: Descending order (Z-A, 9-0).

**Analogy:** Arranging a list of job applicants alphabetically by last name, or numerically by years of experience (highest to lowest).

**Use Case:** Presenting data in a meaningful sequence (e.g., highest salaries first, most recent hires first, alphabetical lists).

**Examples:**

1.  **Sorting by a single column (descending salary):**
    *   **snake_case:**
        ```sql
        -- List teachers ordered by salary, highest first
        SELECT first_name, last_name, salary
        FROM school_district.teacher
        ORDER BY salary DESC;
        ```
    *   **PascalCase:**
        ```sql
        -- List teachers ordered by Salary, highest first
        SELECT "FirstName", "LastName", "Salary"
        FROM "SchoolDistrict"."Teacher"
        ORDER BY "Salary" DESC;
        ```

2.  **Sorting by multiple columns (school ascending, hire date descending):**
    *   **snake_case:**
        ```sql
        -- List teachers grouped by school (A-Z), then by most recently hired within each school
        SELECT last_name, school, hire_date
        FROM school_district.teacher
        ORDER BY school ASC, hire_date DESC;
        ```
    *   **PascalCase:**
        ```sql
        -- List teachers grouped by School (A-Z), then by most recently hired within each school
        SELECT "LastName", "School", "HireDate"
        FROM "SchoolDistrict"."Teacher"
        ORDER BY "School" ASC, "HireDate" DESC;
        ```

**Best Practice:** Limit the number of columns in `ORDER BY` to what's necessary for clarity. Too many sorting levels can make the result hard to interpret.

### Understanding Text Sorting (Collation)

How text sorts (e.g., does 'a' come before 'A'?) depends on the database's **collation** settings, typically determined during PostgreSQL installation based on the operating system's locale (e.g., `en_US.UTF-8`).

You can check your server's setting:
```sql
SHOW lc_collate;
```

In a typical `UTF-8` collation:
*   Punctuation often comes first.
*   Numbers (as text) come before letters.
*   Uppercase letters come before lowercase letters (e.g., 'Z' before 'a').

This is why `'Ladybug'` might sort before `'ladybug'`. Be aware of this when sorting text columns containing mixed cases or special characters. For case-insensitive sorting needs, consider using functions like `LOWER()` in the `ORDER BY` clause (e.g., `ORDER BY LOWER(last_name)`), though this can impact index usage.

[Back to TOC](#-table-of-contents)

## ⚙️ Filtering Rows with WHERE

The `WHERE` clause filters rows, returning only those that meet specified criteria.

**Syntax:**
```sql
SELECT column1, column2 FROM schema_name.table_name WHERE condition;
```

**Analogy:** Asking the interviewee, "Tell me about jobs where your salary was *greater than* $50,000."

**Use Case:** Selecting specific data based on values, ranges, or patterns. This is fundamental to retrieving relevant information.

### Comparison Operators

These operators are used within the `WHERE` clause to define filter conditions.

| Operator  | Function                     | Example (snake_case)                   | Example (PascalCase)                      |
| :-------- | :--------------------------- | :------------------------------------- | :---------------------------------------- |
| `=`       | Equal to                     | `WHERE school = 'Myers Middle School'`   | `WHERE "School" = 'Myers Middle School'`   |
| `<>`      | Not equal to (Standard SQL)  | `WHERE school <> 'F.D. Roosevelt HS'`  | `WHERE "School" <> 'F.D. Roosevelt HS'`  |
| `!=`      | Not equal to (PostgreSQL)    | `WHERE school != 'F.D. Roosevelt HS'`  | `WHERE "School" != 'F.D. Roosevelt HS'`  |
| `>`       | Greater than                 | `WHERE salary > 50000`                 | `WHERE "Salary" > 50000`                  |
| `<`       | Less than                    | `WHERE hire_date < '2000-01-01'`       | `WHERE "HireDate" < '2000-01-01'`         |
| `>=`      | Greater than or equal to     | `WHERE salary >= 43500`                | `WHERE "Salary" >= 43500`                 |
| `<=`      | Less than or equal to        | `WHERE salary <= 65000`                | `WHERE "Salary" <= 65000`                 |
| `BETWEEN` | Within a range (inclusive)   | `WHERE salary BETWEEN 40000 AND 65000` | `WHERE "Salary" BETWEEN 40000 AND 65000`  |
| `IN`      | Matches value in a list      | `WHERE last_name IN ('Cole', 'Bush')`  | `WHERE "LastName" IN ('Cole', 'Bush')`   |
| `NOT`     | Negates a condition          | `WHERE school NOT IN (...)`            | `WHERE "School" NOT IN (...)`             |

**Examples:**

1.  **Filtering by exact match:**
    *   **snake_case:**
        ```sql
        -- Find teachers at Myers Middle School
        SELECT last_name, school, hire_date
        FROM school_district.teacher
        WHERE school = 'Myers Middle School';
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers at Myers Middle School
        SELECT "LastName", "School", "HireDate"
        FROM "SchoolDistrict"."Teacher"
        WHERE "School" = 'Myers Middle School';
        ```

2.  **Filtering by date range (less than):**
    *   **snake_case:**
        ```sql
        -- Find teachers hired before 2000
        SELECT first_name, last_name, hire_date
        FROM school_district.teacher
        WHERE hire_date < '2000-01-01'; -- Use ISO 8601 format (YYYY-MM-DD) for dates
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers hired before 2000
        SELECT "FirstName", "LastName", "HireDate"
        FROM "SchoolDistrict"."Teacher"
        WHERE "HireDate" < '2000-01-01';
        ```

3.  **Filtering using BETWEEN:**
    *   **snake_case:**
        ```sql
        -- Find teachers earning between $40k and $65k (inclusive)
        SELECT first_name, last_name, school, salary
        FROM school_district.teacher
        WHERE salary BETWEEN 40000 AND 65000;
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers earning between $40k and $65k (inclusive)
        SELECT "FirstName", "LastName", "School", "Salary"
        FROM "SchoolDistrict"."Teacher"
        WHERE "Salary" BETWEEN 40000 AND 65000;
        ```

### Pattern Matching with LIKE and ILIKE

These operators search for patterns within string (text) data.

*   `LIKE`: Case-sensitive pattern matching (Standard SQL).
*   `ILIKE`: Case-insensitive pattern matching (PostgreSQL specific, very useful!).

**Wildcards:**
*   `%`: Matches any sequence of zero or more characters.
*   `_`: Matches any single character.

**Examples:**

1.  **Finding names starting with 'sam' (case-insensitive):**
    *   **snake_case:**
        ```sql
        -- ILIKE ignores case: finds 'Samuel' and 'Samantha'
        SELECT first_name
        FROM school_district.teacher
        WHERE first_name ILIKE 'sam%';
        ```
    *   **PascalCase:**
        ```sql
        -- ILIKE ignores case: finds 'Samuel' and 'Samantha'
        SELECT "FirstName"
        FROM "SchoolDistrict"."Teacher"
        WHERE "FirstName" ILIKE 'sam%';
        ```

2.  **`LIKE` (case-sensitive) would likely return no results if the data is 'Samuel'/'Samantha':**
    *   **snake_case:**
        ```sql
        -- LIKE is case-sensitive: would only find 'samuel' if it existed exactly like that
        SELECT first_name
        FROM school_district.teacher
        WHERE first_name LIKE 'sam%'; -- Likely returns 0 rows
        ```
    *   **PascalCase:**
        ```sql
        -- LIKE is case-sensitive: would only find 'samuel' if it existed exactly like that
        SELECT "FirstName"
        FROM "SchoolDistrict"."Teacher"
        WHERE "FirstName" LIKE 'sam%'; -- Likely returns 0 rows
        ```

**Best Practice:** Prefer `ILIKE` for robustness when searching user-provided or potentially inconsistently cased text data, unless case sensitivity is explicitly required.

**Pitfall:** Queries using `LIKE` or `ILIKE` with a leading wildcard (`%text`) are often slow on large tables because they usually cannot use standard indexes effectively. Trailing wildcards (`text%`) are generally faster. We'll cover indexing later.

### Combining Operators with AND and OR

Combine multiple conditions in the `WHERE` clause using `AND` and `OR`. Use parentheses `()` to control the order of evaluation.

*   `AND`: BOTH conditions must be true for a row to be included.
*   `OR`: AT LEAST ONE of the conditions must be true for a row to be included.

**Examples:**

1.  **Using `AND`:**
    *   **snake_case:**
        ```sql
        -- Find teachers at Myers Middle School AND earning less than $40k
        SELECT *
        FROM school_district.teacher
        WHERE school = 'Myers Middle School' AND salary < 40000;
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers at Myers Middle School AND earning less than $40k
        SELECT *
        FROM "SchoolDistrict"."Teacher"
        WHERE "School" = 'Myers Middle School' AND "Salary" < 40000;
        ```

2.  **Using `OR`:**
    *   **snake_case:**
        ```sql
        -- Find teachers whose last name is Cole OR Bush
        SELECT *
        FROM school_district.teacher
        WHERE last_name = 'Cole' OR last_name = 'Bush';
        -- Alternative using IN: WHERE last_name IN ('Cole', 'Bush');
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers whose last name is Cole OR Bush
        SELECT *
        FROM "SchoolDistrict"."Teacher"
        WHERE "LastName" = 'Cole' OR "LastName" = 'Bush';
        -- Alternative using IN: WHERE "LastName" IN ('Cole', 'Bush');
        ```

3.  **Using `AND`, `OR`, and Parentheses:**
    *   **snake_case:**
        ```sql
        -- Find teachers at F.D. Roosevelt HS AND (salary is < $38k OR salary is > $40k)
        SELECT *
        FROM school_district.teacher
        WHERE school = 'F.D. Roosevelt HS'
          AND (salary < 38000 OR salary > 40000);
        ```
    *   **PascalCase:**
        ```sql
        -- Find teachers at F.D. Roosevelt HS AND (salary is < $38k OR salary is > $40k)
        SELECT *
        FROM "SchoolDistrict"."Teacher"
        WHERE "School" = 'F.D. Roosevelt HS'
          AND ("Salary" < 38000 OR "Salary" > 40000);
        ```
    *   Parentheses ensure the `OR` condition is evaluated first as a group.

[Back to TOC](#-table-of-contents)

## ✨ Putting It All Together: The Standard SELECT Structure

SQL requires clauses to be in a specific order. For the basic statements covered here, the order is:

1.  `SELECT` column_names
2.  `FROM` schema_name.table_name
3.  `WHERE` criteria (optional)
4.  `ORDER BY` column_names (optional)

**Example combining multiple clauses:**

*   **snake_case:**
    ```sql
    -- Select specific columns for teachers in schools containing 'Roos' in their name,
    -- ordered by the most recent hire date first.
    SELECT first_name, last_name, school, hire_date, salary
    FROM school_district.teacher
    WHERE school ILIKE '%Roos%' -- Case-insensitive search for 'Roos' anywhere in the name
    ORDER BY hire_date DESC;
    ```
*   **PascalCase:**
    ```sql
    -- Select specific columns for teachers in schools containing 'Roos' in their name,
    -- ordered by the most recent hire date first.
    SELECT "FirstName", "LastName", "School", "HireDate", "Salary"
    FROM "SchoolDistrict"."Teacher"
    WHERE "School" ILIKE '%Roos%' -- Case-insensitive search for 'Roos' anywhere in the name
    ORDER BY "HireDate" DESC;
    ```
This query demonstrates selecting specific columns, filtering rows based on a pattern (`ILIKE`), and ordering the results.

[Back to TOC](#-table-of-contents)

## ✅ Best Practices Summary

*   **Use Schemas:** Organize your database objects using schemas (e.g., `school_district`).
*   **Singular Table Names:** Name tables using singular nouns (e.g., `teacher`).
*   **Consistent Naming:** Choose either `snake_case` or `PascalCase` (requires quoting) and stick to it for tables, columns, constraints, etc.
*   **Fully Qualified Names:** Reference objects using `schema_name.table_name` (or quoted equivalent) for clarity and explicitness.
*   **Avoid `SELECT *` in application code:** Explicitly list the columns you need. It's clearer, more efficient, and less prone to breaking if the table structure changes. `SELECT *` is acceptable for quick interactive exploration.
*   **Use `WHERE` effectively:** Filter data as early as possible in your query process.
*   **Prefer `ILIKE` for case-insensitive searches:** Use PostgreSQL's `ILIKE` operator unless case matters.
*   **Be mindful of `LIKE`/`ILIKE` performance:** Leading wildcards (`%text`) can be slow.
*   **Use `ORDER BY` for presentation:** Sort data to make it understandable, but limit complexity.
*   **Use `DISTINCT` for unique values:** Useful for exploring data variety and finding inconsistencies.
*   **Use standard date formats:** Use the ISO 8601 format (`'YYYY-MM-DD'`) when filtering date columns for clarity and unambiguous results.
*   **Understand Collation:** Be aware of how text sorting works in your database environment.
*   **Use modern Identity Columns:** Prefer `GENERATED ALWAYS AS IDENTITY` over `serial` or `bigserial` in PostgreSQL 10+.

[Back to TOC](#-table-of-contents)

## ✍️ Hands-on Exercises

Use the concepts learned to write queries against the `school_district.teacher` (snake_case) or `"SchoolDistrict"."Teacher"` (PascalCase) table. Assume the table exists within the appropriate schema and contains the data shown in the PDF chapter.

**Reference Table Structure (Conceptual):**

*   **snake_case:** `school_district.teacher`
    *   `id` (bigint, primary key, auto-incrementing - e.g., `GENERATED ALWAYS AS IDENTITY`)
    *   `first_name` (text)
    *   `last_name` (text)
    *   `school` (text)
    *   `hire_date` (date)
    *   `salary` (numeric)
*   **PascalCase:** `"SchoolDistrict"."Teacher"  -- Note: Pky is {TableName}{Id or Key}`
    *   `"TeacherId"` (bigint, primary key, auto-incrementing - e.g., `GENERATED ALWAYS AS IDENTITY`)
    *   `"FirstName"` (text)
    *   `"LastName"` (text)
    *   `"School"` (text)
    *   `"HireDate"` (date)
    *   `"Salary"` (numeric)

**Data (Based on PDF):**

| id / "TeacherId" | first\_name / "FirstName" | last\_name / "LastName" | school / "School"     | hire\_date / "HireDate" | salary / "Salary" |
| :--------------- | :------------------------ | :---------------------- | :-------------------- | :---------------------- | ----------------: |
| 1                | Janet                     | Smith                   | F.D. Roosevelt HS     | 2011-10-30              |             36200 |
| 2                | Lee                       | Reynolds                | F.D. Roosevelt HS     | 1993-05-22              |             65000 |
| 3                | Samuel                    | Cole                    | Myers Middle School   | 2005-08-01              |             43500 |
| 4                | Samantha                  | Bush                    | Myers Middle School   | 2011-10-30              |             36200 |
| 5                | Betty                     | Diaz                    | Myers Middle School   | 2005-08-30              |             43500 |
| 6                | Kathleen                  | Roush                   | F.D. Roosevelt HS     | 2010-10-22              |             38500 |

---

**1. School Roster (Alphabetical)**

*   **Task:** The school district superintendent asks for a list of teachers in each school. Write a query that lists the schools in alphabetical order along with teachers ordered by last name A-Z within each school.
*   **Solution (snake_case):**
    ```sql
    SELECT school, last_name, first_name
    FROM school_district.teacher
    ORDER BY school ASC, last_name ASC;
    ```
*   **Solution (PascalCase):**
    ```sql
    SELECT "School", "LastName", "FirstName"
    FROM "SchoolDistrict"."Teacher"
    ORDER BY "School" ASC, "LastName" ASC;
    ```
*   **Expected Output (Conceptual):**
    ```
    school             last_name  first_name
    ------------------ ---------- ----------
    F.D. Roosevelt HS  Reynolds   Lee
    F.D. Roosevelt HS  Roush      Kathleen
    F.D. Roosevelt HS  Smith      Janet
    Myers Middle School Bush       Samantha
    Myers Middle School Cole       Samuel
    Myers Middle School Diaz       Betty
    ```

---

**2. Specific Teacher Search**

*   **Task:** Write a query that finds the one teacher whose first name starts with the letter S *and* who earns more than $40,000.
*   **Solution (snake_case):**
    ```sql
    -- Using ILIKE for robustness against case variations (e.g., 'samuel')
    -- Although LIKE 'S%' would work if names are consistently capitalized 'Samuel'
    SELECT first_name, last_name, salary
    FROM school_district.teacher
    WHERE first_name ILIKE 'S%' AND salary > 40000;
    ```
*   **Solution (PascalCase):**
    ```sql
    SELECT "FirstName", "LastName", "Salary"
    FROM "SchoolDistrict"."Teacher"
    WHERE "FirstName" ILIKE 'S%' AND "Salary" > 40000;
    ```
*   **Expected Output:**
    ```
    first_name  last_name   salary
    ----------- ----------- --------
    Samuel      Cole        43500
    ```

---

**3. Recent Hires Ranking**

*   **Task:** Rank teachers hired on or after January 1, 2010, ordered by highest salary to lowest.
*   **Solution (snake_case):**
    ```sql
    SELECT first_name, last_name, hire_date, salary
    FROM school_district.teacher
    WHERE hire_date >= '2010-01-01' -- Inclusive of the date
    ORDER BY salary DESC;
    ```

*   **Solution (PascalCase):**

    ```sql
    SELECT "FirstName", "LastName", "HireDate", "Salary"
    FROM "SchoolDistrict"."Teacher"
    WHERE "HireDate" >= '2010-01-01' -- Inclusive of the date
    ORDER BY "Salary" DESC;
    ```
*   **Expected Output:**
    ```
    first_name  last_name   hire_date   salary
    ----------- ----------- ----------- --------
    Kathleen    Roush       2010-10-22  38500
    Janet       Smith       2011-10-30  36200
    Samantha    Bush        2011-10-30  36200
    ```

[Back to TOC](#-table-of-contents)
```