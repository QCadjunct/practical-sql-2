# Chapter 1: Creating Your First Database and Table

## Table of Contents

- [Introduction](#introduction)
- [Understanding Tables in Databases](#understanding-tables-in-databases)
  - [What is a Table?](#what-is-a-table)
  - [Example of Tables and Relationships](#example-of-tables-and-relationships)
- [Creating a Database](#creating-a-database)
  - [Basic Database Creation](#basic-database-creation)
  - [Executing SQL in pgAdmin](#executing-sql-in-pgadmin)
- [Connecting to Your Database](#connecting-to-your-database)
- [Creating a Table](#creating-a-table)
  - [Table Structure and Data Types](#table-structure-and-data-types)
  - [Creating the Table in pgAdmin](#creating-the-table-in-pgadmin)
- [Inserting Rows into a Table](#inserting-rows-into-a-table)
  - [The INSERT Statement](#the-insert-statement)
  - [Viewing the Data](#viewing-the-data)
- [When Code Goes Bad](#when-code-goes-bad)
- [SQL Formatting Best Practices](#sql-formatting-best-practices)
- [Try It Yourself Exercises](#try-it-yourself-exercises)
- [Summary](#summary)

## Introduction

SQL serves a dual purpose: it extracts knowledge from data and defines the structures that hold data, organizing relationships within it. This chapter introduces fundamental SQL concepts by walking through the creation of a database, defining tables, and adding data. These foundational skills form the building blocks of database management and data analysis.

[Back to Table of Contents](#table-of-contents)

## Understanding Tables in Databases

### What is a Table?

A table forms the basic structure for organizing data in a database:

- Tables consist of rows and columns that store data 
- Columns contain data of specific types (numbers, characters, dates)
- SQL defines table structure and relationships between tables
- SQL also extracts (queries) data from tables

When analyzing a database, understanding table structure provides insight into:

- Data types and content
- Relationships between tables
- Database complexity and organization

### Example of Tables and Relationships

The chapter presents a hypothetical database for school class enrollment:

#### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** This represents a student enrollment system where students can register for multiple classes. The system tracks which students enroll in which courses, maintaining relationships between students and their classes.

```sql
-- Business context: Student enrollment system tracking which students are registered for which classes
-- Predicate: Domain(StudentEnrollment) → StudentEnrollment ∈ School
CREATE TABLE "School"."StudentEnrollment" (
    "StudentEnrollmentId" SERIAL,
    "StudentId" VARCHAR(10),
    "ClassId" VARCHAR(10),
    "ClassSection" INTEGER,
    "Semester" VARCHAR(20),
    
    -- Standard metadata fields
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ StudentEnrollment: (x.StudentEnrollmentId = y.StudentEnrollmentId) → (x = y)
    CONSTRAINT "PK_School_StudentEnrollment" PRIMARY KEY ("StudentEnrollmentId"),
    
    -- Foreign Key predicates: 
    -- ∀x ∈ StudentEnrollment: ∃y ∈ Student: (x.StudentId = y.StudentId)
    CONSTRAINT "FK_School_StudentEnrollment_School_Student" FOREIGN KEY ("StudentId")
        REFERENCES "School"."Student"("StudentId"),
    -- ∀x ∈ StudentEnrollment: ∃y ∈ Class: (x.ClassId = y.ClassId)
    CONSTRAINT "FK_School_StudentEnrollment_School_Class" FOREIGN KEY ("ClassId")
        REFERENCES "School"."Class"("ClassId"),
        
    -- Not null predicates:
    -- ∀x ∈ StudentEnrollment: x.StudentId ≠ NULL
    CONSTRAINT "CHK_School_StudentEnrollment_StudentId" CHECK ("StudentId" IS NOT NULL),
    -- ∀x ∈ StudentEnrollment: x.ClassId ≠ NULL
    CONSTRAINT "CHK_School_StudentEnrollment_ClassId" CHECK ("ClassId" IS NOT NULL),
    -- ∀x ∈ StudentEnrollment: x.ClassSection ≠ NULL
    CONSTRAINT "CHK_School_StudentEnrollment_ClassSection" CHECK ("ClassSection" IS NOT NULL),
    -- ∀x ∈ StudentEnrollment: x.Semester ≠ NULL
    CONSTRAINT "CHK_School_StudentEnrollment_Semester" CHECK ("Semester" IS NOT NULL),
    
    -- Default constraints
    -- ∀x ∈ StudentEnrollment: (x.StudentId = NULL) → (x.StudentId := "Missing StudentId")
    CONSTRAINT "DF_School_StudentEnrollment_StudentId" DEFAULT 'Missing StudentId' FOR "StudentId",
    -- ∀x ∈ StudentEnrollment: (x.ClassId = NULL) → (x.ClassId := "Missing ClassId")
    CONSTRAINT "DF_School_StudentEnrollment_ClassId" DEFAULT 'Missing ClassId' FOR "ClassId",
    -- ∀x ∈ StudentEnrollment: (x.ClassSection = NULL) → (x.ClassSection := -9999)
    CONSTRAINT "DF_School_StudentEnrollment_ClassSection" DEFAULT -9999 FOR "ClassSection",
    -- ∀x ∈ StudentEnrollment: (x.Semester = NULL) → (x.Semester := "Missing Semester")
    CONSTRAINT "DF_School_StudentEnrollment_Semester" DEFAULT 'Missing Semester' FOR "Semester"
);
```

```sql
-- Business context: Storing core student personal information
-- Predicate: Domain(Student) → Student ∈ School
CREATE TABLE "School"."Student" (
    "StudentId" VARCHAR(10),
    "FirstName" VARCHAR(50),
    "LastName" VARCHAR(50),
    "DateOfBirth" DATE,
    
    -- Standard metadata fields
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ Student: (x.StudentId = y.StudentId) → (x = y)
    CONSTRAINT "PK_School_Student" PRIMARY KEY ("StudentId"),
    
    -- Not null predicates
    -- ∀x ∈ Student: x.FirstName ≠ NULL
    CONSTRAINT "CHK_School_Student_FirstName" CHECK ("FirstName" IS NOT NULL),
    -- ∀x ∈ Student: x.LastName ≠ NULL
    CONSTRAINT "CHK_School_Student_LastName" CHECK("LastName" IS NOT NULL),
    -- ∀x ∈ Student: x.DateOfBirth ≠ NULL
    CONSTRAINT "CHK_School_Student_DateOfBirth" CHECK ("DateOfBirth" IS NOT NULL),
    
    -- Default constraints
    -- ∀x ∈ Student: (x.FirstName = NULL) → (x.FirstName := "Missing FirstName")
    CONSTRAINT "DF_School_Student_FirstName" DEFAULT 'Missing FirstName' FOR "FirstName",
    -- ∀x ∈ Student: (x.LastName = NULL) → (x.LastName := "Missing LastName")
    CONSTRAINT "DF_School_Student_LastName" DEFAULT 'Missing LastName' FOR "LastName",
    -- ∀x ∈ Student: (x.DateOfBirth = NULL) → (x.DateOfBirth := "1900-01-01")
    CONSTRAINT "DF_School_Student_DateOfBirth" DEFAULT '1900-01-01' FOR "DateOfBirth"
);
```

```sql
-- Business context: Storing class catalog information
-- Predicate: Domain(Class) → Class ∈ School
CREATE TABLE "School"."Class" (
    "ClassId" VARCHAR(10),
    "ClassName" VARCHAR(100),
    "Department" VARCHAR(50),
    "Credits" INTEGER,
    
    -- Standard metadata fields
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ Class: (x.ClassId = y.ClassId) → (x = y)
    CONSTRAINT "PK_School_Class" PRIMARY KEY ("ClassId"),
    
    -- Not null predicates
    -- ∀x ∈ Class: x.ClassName ≠ NULL
    CONSTRAINT "CHK_School_Class_ClassName" CHECK ("ClassName" IS NOT NULL),
    -- ∀x ∈ Class: x.Department ≠ NULL
    CONSTRAINT "CHK_School_Class_Department" CHECK ("Department" IS NOT NULL),
    -- ∀x ∈ Class: x.Credits ≠ NULL
    CONSTRAINT "CHK_School_Class_Credits" CHECK ("Credits" IS NOT NULL),
    
    -- Default constraints
    -- ∀x ∈ Class: (x.ClassName = NULL) → (x.ClassName := "Missing Class Name")
    CONSTRAINT "DF_School_Class_ClassName" DEFAULT 'Missing Class Name' FOR "ClassName",
    -- ∀x ∈ Class: (x.Department = NULL) → (x.Department := "Unassigned Department")
    CONSTRAINT "DF_School_Class_Department" DEFAULT 'Unassigned Department' FOR "Department",
    -- ∀x ∈ Class: (x.Credits = NULL) → (x.Credits := 0)
    CONSTRAINT "DF_School_Class_Credits" DEFAULT 0 FOR "Credits"
);
```

#### PostgreSQL Standard Implementation

```sql
CREATE TABLE student_enrollment (
    student_id VARCHAR(10),
    class_id VARCHAR(10),
    class_section INTEGER,
    semester VARCHAR(20)
);

CREATE TABLE students (
    student_id VARCHAR(10) PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    dob DATE
);

CREATE TABLE classes (
    class_id VARCHAR(10) PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    credits INTEGER
);
```

Example data in these tables:

```markdown
-- Data in student_enrollment table
student_id    class_id        class_section    semester
CHRISPA004    COMPSCI101      3                Fall 2017
DAVISHE010    COMPSCI101      3                Fall 2017
ABRILDA002    ENG101          40               Fall 2017
DAVISHE010    ENG101          40               Fall 2017
RILEYPH002    ENG101          40               Fall 2017

-- Data in students table
student_id    first_name    last_name       dob
ABRILDA002    Abril         Davis           1999-01-10
CHRISPA004    Chris         Park            1996-04-10
DAVISHE010    Davis         Hernandez       1987-09-14
RILEYPH002    Riley         Phelps          1996-06-15
```

#### Implementation Analysis

- The Domain Driven Design approach with predicate logic offers significant advantages:
  - **Two-valued Logic**: By providing explicit defaults for all columns, we eliminate NULL values and ensure every column has exactly two states: user-provided or system-default
  - **Data Quality Tracking**: The "IsDataMissing" flag explicitly tracks records with incomplete data
  - **Complete Metadata**: Every record contains audit information about who created it and when
  - **Business Semantics**: The schema and constraint names directly express the business domain and rules
  - **Self-documenting Code**: Naming conventions make the database structure easily understandable for both technical and business users

- The relational model connects information across multiple tables:
  - The `"School"."StudentEnrollment"` table connects students to classes, establishing a many-to-many relationship
  - The `"School"."Student"` table stores core student demographic information
  - The `"School"."Class"` table stores course catalog information

This design reduces redundancy by storing information once and using unique identifiers to reference it from other tables, demonstrating the power of relational database modeling.

[Back to Table of Contents](#table-of-contents)

## Creating a Database

### Basic Database Creation

#### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Creating a database for data analysis purposes that will contain multiple related tables for analytical processing.

```sql
-- Business context: Creating an analysis database to store related data for analytical purposes
-- Predicate: Database("Analysis") represents a coherent domain for analytical data
CREATE DATABASE "Analysis";
```

#### PostgreSQL Standard Implementation

```sql
CREATE DATABASE analysis;
```

The statement consists of SQL keywords (`CREATE DATABASE`) followed by the database name and ending with a semicolon. The semicolon is part of the ANSI SQL standard and signals the end of a command.

### Executing SQL in pgAdmin

1. Ensure PostgreSQL is running (on Windows, it should automatically launch at startup; on macOS, double-click Postgres.app)
2. Launch pgAdmin 
3. Connect to the default server in the object browser
4. Select the default `postgres` database
5. Open Query Tool from the Tools menu
6. Enter the CREATE DATABASE statement  
7. Execute by clicking the lightning bolt icon
8. Refresh the Databases node to see your new database

#### Implementation Analysis
Creating a new database for each project is a best practice as it:
1. Organizes related tables together
2. Prevents namespace collisions between projects
3. Simplifies backup and restoration operations
4. Allows for better security and permission controls
5. Enables independent evolution of different applications

[Back to Table of Contents](#table-of-contents)

## Connecting to Your Database

After creating a database, you need to connect to it before creating any objects:

1. Close the Query Tool by clicking the X at the top right of the tool
2. In the object browser, click once on the analysis database (or "Analysis" in DDD)
3. Reopen Query Tool by choosing Tools ► Query Tool
4. Verify the correct database connection at the top of the Query Tool window (you should see "analysis" or "Analysis")

Any SQL code you execute now will apply to the selected database.

[Back to Table of Contents](#table-of-contents)

## Creating a Table

### Table Structure and Data Types

Tables consist of columns with defined data types that control what values they accept. The `CREATE TABLE` statement defines the table structure.

#### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Creating a table to store information about teachers, including their personal details, workplace information, and salary details for educational administration and payroll.

```sql
-- Business context: Tracking teacher information including workplace and compensation details
-- Predicate: Domain(Teacher) → Teacher ∈ Education
CREATE TABLE "Education"."Teacher" (
    "TeacherId" BIGSERIAL,     -- Auto-incrementing unique identifier
    "FirstName" VARCHAR(25),   -- Teacher's first/given name
    "LastName" VARCHAR(50),    -- Teacher's last/family name
    "SchoolName" VARCHAR(50),  -- Name of the school where teacher works
    "HireDate" DATE,           -- Date when teacher was hired
    "Salary" NUMERIC,          -- Teacher's annual compensation
    
    -- Standard metadata fields for two-valued predicate logic
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ Teacher: (x.TeacherId = y.TeacherId) → (x = y)
    CONSTRAINT "PK_Education_Teacher" PRIMARY KEY ("TeacherId"),
    
    -- Not null predicates
    -- ∀x ∈ Teacher: x.FirstName ≠ NULL
    CONSTRAINT "CHK_Education_Teacher_FirstName" CHECK ("FirstName" IS NOT NULL),
    -- ∀x ∈ Teacher: x.LastName ≠ NULL
    CONSTRAINT "CHK_Education_Teacher_LastName" CHECK ("LastName" IS NOT NULL),
    -- ∀x ∈ Teacher: x.SchoolName ≠ NULL
    CONSTRAINT "CHK_Education_Teacher_SchoolName" CHECK ("SchoolName" IS NOT NULL),
    -- ∀x ∈ Teacher: x.HireDate ≠ NULL
    CONSTRAINT "CHK_Education_Teacher_HireDate" CHECK ("HireDate" IS NOT NULL),
    -- ∀x ∈ Teacher: x.Salary ≠ NULL
    CONSTRAINT "CHK_Education_Teacher_Salary" CHECK ("Salary" IS NOT NULL),
    
    -- Business rule: Salary must be positive
    -- Predicate: ∀x ∈ Teacher: x.Salary > 0
    CONSTRAINT "CHK_Education_Teacher_SalaryPositive" CHECK ("Salary" > 0),
    
    -- Default constraints
    -- ∀x ∈ Teacher: (x.FirstName = NULL) → (x.FirstName := "Missing First Name")
    CONSTRAINT "DF_Education_Teacher_FirstName" DEFAULT 'Missing First Name' FOR "FirstName",
    -- ∀x ∈ Teacher: (x.LastName = NULL) → (x.LastName := "Missing Last Name")
    CONSTRAINT "DF_Education_Teacher_LastName" DEFAULT 'Missing Last Name' FOR "LastName",
    -- ∀x ∈ Teacher: (x.SchoolName = NULL) → (x.SchoolName := "No School Assigned")
    CONSTRAINT "DF_Education_Teacher_SchoolName" DEFAULT 'No School Assigned' FOR "SchoolName",
    -- ∀x ∈ Teacher: (x.HireDate = NULL) → (x.HireDate := "1900-01-01")
    CONSTRAINT "DF_Education_Teacher_HireDate" DEFAULT '1900-01-01' FOR "HireDate",
    -- ∀x ∈ Teacher: (x.Salary = NULL) → (x.Salary := 0)
    CONSTRAINT "DF_Education_Teacher_Salary" DEFAULT 0 FOR "Salary"
);
```

#### PostgreSQL Standard Implementation

```sql
CREATE TABLE teachers (
    id bigserial,
    first_name varchar(25),
    last_name varchar(50),
    school varchar(50),
    hire_date date,
    salary numeric
);
```

#### Implementation Analysis
Key elements of this table definition:
- `id`/`"TeacherId"`: Uses `bigserial` data type for auto-incrementing ID values
- Text columns use `varchar` with length limits to optimize storage
- `hire_date`/`"HireDate"`: Uses `date` data type for employment start date
- `salary`/`"Salary"`: Uses `numeric` data type for precise decimal values

The DDDD implementation adds:
- **Explicit NOT NULL constraints** expressed as formal predicates
- **Standard metadata fields** for tracking data quality, ownership, and modifications
- **Business rule enforcement** such as requiring salary to be positive
- **Meaningful default values** with standard prefixes like "Missing First Name"
- **Two-valued logic** by ensuring no NULL values can exist
- **Sentinel values** for dates (1900-01-01) and numeric fields (0)
- **Self-documenting naming** that clearly expresses business meaning

### Creating the Table in pgAdmin

1. Ensure you're connected to the correct database
2. Enter the CREATE TABLE statement in the Query Tool
3. Execute the statement by clicking the lightning bolt icon
4. Refresh the database to see the new table in the object browser
5. Expand the table node to examine column details

[Back to Table of Contents](#table-of-contents)

## Inserting Rows into a Table

### The INSERT Statement

After creating a table, you can add data using the `INSERT INTO` statement.

#### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Adding teacher records with their employment details to populate the teacher information system for staff management and payroll.

```sql
-- Business context: Populating the teacher database with staff records including hire dates and salaries
-- Predicate: ∀x in inserted rows: x ∈ Teacher
INSERT INTO "Education"."Teacher" ("FirstName", "LastName", "SchoolName", "HireDate", "Salary")
VALUES 
    ('Janet', 'Smith', 'F.D. Roosevelt HS', '2011-10-30', 36200),
    ('Lee', 'Reynolds', 'F.D. Roosevelt HS', '1993-05-22', 65000),
    ('Samuel', 'Cole', 'Myers Middle School', '2005-08-01', 43500),
    ('Samantha', 'Bush', 'Myers Middle School', '2011-10-30', 36200),
    ('Betty', 'Diaz', 'Myers Middle School', '2005-08-30', 43500),
    ('Kathleen', 'Roush', 'F.D. Roosevelt HS', '2010-10-22', 38500);
```

#### PostgreSQL Standard Implementation

```sql
INSERT INTO teachers (first_name, last_name, school, hire_date, salary)
VALUES 
    ('Janet', 'Smith', 'F.D. Roosevelt HS', '2011-10-30', 36200),
    ('Lee', 'Reynolds', 'F.D. Roosevelt HS', '1993-05-22', 65000),
    ('Samuel', 'Cole', 'Myers Middle School', '2005-08-01', 43500),
    ('Samantha', 'Bush', 'Myers Middle School', '2011-10-30', 36200),
    ('Betty', 'Diaz', 'Myers Middle School', '2005-08-30', 43500),
    ('Kathleen', 'Roush', 'F.D. Roosevelt HS', '2010-10-22', 38500);
```

#### Implementation Analysis
Important syntax rules for INSERT statements:
1. List the target columns after the table name using proper quoting/casing convention
2. Use the `VALUES` keyword before the data
3. Enclose each row's data in parentheses and separate rows with commas
4. Inside each row's parentheses, separate column values with commas
5. **Data type quoting rules:**
   - Text values: Must be enclosed in single quotes
   - Date values: Must be enclosed in single quotes and use ISO 8601 format (YYYY-MM-DD)
   - Numeric values (integers and decimals): No quotes needed
6. The order of values must match the order of columns specified

The `bigserial` column (`id` or `"TeacherId"`) automatically gets assigned values, so it's not included in the INSERT statement. Similarly, the standard metadata fields in the DDDD approach are automatically populated with their default values.

### Viewing the Data

#### Domain Driven Database Design Output

```
TeacherId | FirstName | LastName  | SchoolName         | HireDate   | Salary | IsDataMissing | UserAuthorizationId | DateAdded           | DateOfLastUpdate
----------+-----------+-----------+--------------------+------------+--------+---------------+---------------------+---------------------+-------------------
1         | Janet     | Smith     | F.D. Roosevelt HS  | 2011-10-30 | 36200  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
2         | Lee       | Reynolds  | F.D. Roosevelt HS  | 1993-05-22 | 65000  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
3         | Samuel    | Cole      | Myers Middle School| 2005-08-01 | 43500  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
4         | Samantha  | Bush      | Myers Middle School| 2011-10-30 | 36200  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
5         | Betty     | Diaz      | Myers Middle School| 2005-08-30 | 43500  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
6         | Kathleen  | Roush     | F.D. Roosevelt HS  | 2010-10-22 | 38500  | FALSE         | 1                   | 2023-05-15 10:30:00 | 2023-05-15 10:30:00
```

Note that:
- The "IsDataMissing" flag is FALSE since all required data was provided
- The "UserAuthorizationId" shows the default system user (ID 1)
- The "DateAdded" and "DateOfLastUpdate" fields show when the record was created
- The "TeacherId" was automatically generated by the BIGSERIAL type

#### PostgreSQL Standard Output

```
id | first_name | last_name | school             | hire_date  | salary
---+------------+-----------+--------------------+------------+-------
1  | Janet      | Smith     | F.D. Roosevelt HS  | 2011-10-30 | 36200
2  | Lee        | Reynolds  | F.D. Roosevelt HS  | 1993-05-22 | 65000
3  | Samuel     | Cole      | Myers Middle School| 2005-08-01 | 43500
4  | Samantha   | Bush      | Myers Middle School| 2011-10-30 | 36200
5  | Betty      | Diaz      | Myers Middle School| 2005-08-30 | 43500
6  | Kathleen   | Roush     | F.D. Roosevelt HS  | 2010-10-22 | 38500
```

You can view this data in pgAdmin by right-clicking the table and selecting "View/Edit Data" > "All Rows".

[Back to Table of Contents](#table-of-contents)

## When Code Goes Bad

⚠️ The chapter notes that syntax errors are common when writing SQL. For example, forgetting a comma between value sets would produce:

```
ERROR: syntax error at or near "("
LINE 5: ('Samuel', 'Cole', 'Myers Middle School', '2005-08-01', 43...
                                                                  ^
```

Error messages typically hint at the location and nature of the problem. When encountering obscure errors, searching online for the error message often helps.

[Back to Table of Contents](#table-of-contents)

## SQL Formatting Best Practices

💡 Best practices for SQL formatting include:
- Uppercase SQL keywords (SELECT, INSERT, CREATE)
- Use lowercase_and_underscores for object names in PostgreSQL standard
- Use PascalCase and double quotes for object names in Domain Driven Design
- Indent clauses and code blocks with 2-4 spaces for readability

These conventions enhance readability and help maintain consistency in codebases, especially when multiple people work with the same code.

[Back to Table of Contents](#table-of-contents)

## Try It Yourself Exercises

🔍 Exercise 1: Design a database for a zoo with tables for animal types and individual animals

#### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Creating a database system to track zoo animals, their species types, and conservation status for zoo management.

```sql
-- Business context: Tracking animal species information at a zoo
-- Predicate: Domain(AnimalType) → AnimalType ∈ Zoo
CREATE TABLE "Zoo"."AnimalType" (
    "AnimalTypeId" SERIAL,
    "Name" VARCHAR(50),
    "Habitat" VARCHAR(50),
    "Diet" VARCHAR(30),
    
    -- Standard metadata fields
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ AnimalType: (x.AnimalTypeId = y.AnimalTypeId) → (x = y)
    CONSTRAINT "PK_Zoo_AnimalType" PRIMARY KEY ("AnimalTypeId"),
    
    -- Not null predicates
    -- ∀x ∈ AnimalType: x.Name ≠ NULL
    CONSTRAINT "CHK_Zoo_AnimalType_Name" CHECK ("Name" IS NOT NULL),
    -- ∀x ∈ AnimalType: x.Habitat ≠ NULL
    CONSTRAINT "CHK_Zoo_AnimalType_Habitat" CHECK ("Habitat" IS NOT NULL),
    -- ∀x ∈ AnimalType: x.Diet ≠ NULL
    CONSTRAINT "CHK_Zoo_AnimalType_Diet" CHECK ("Diet" IS NOT NULL),
    
    -- Uniqueness predicate: ∀x,y ∈ AnimalType: (x.Name = y.Name) → (x = y)
    -- No two animal types can have the same name
    CONSTRAINT "UQ_Zoo_AnimalType_Name" UNIQUE ("Name"),
    
    -- Default constraints
    -- ∀x ∈ AnimalType: (x.Name = NULL) → (x.Name := "Unspecified Species")
    CONSTRAINT "DF_Zoo_AnimalType_Name" DEFAULT 'Unspecified Species' FOR "Name",
    -- ∀x ∈ AnimalType: (x.Habitat = NULL) → (x.Habitat := "Unknown")
    CONSTRAINT "DF_Zoo_AnimalType_Habitat" DEFAULT 'Unknown Habitat' FOR "Habitat",
    -- ∀x ∈ AnimalType: (x.Diet = NULL) → (x.Diet := "Unspecified")
    CONSTRAINT "DF_Zoo_AnimalType_Diet" DEFAULT 'Unspecified Diet' FOR "Diet"
);

-- Business context: Tracking individual animals in the zoo's care
-- Predicate: Domain(Animal) → Animal ∈ Zoo
CREATE TABLE "Zoo"."Animal" (
    "AnimalId" SERIAL,
    "Name" VARCHAR(50),
    "AnimalTypeId" INTEGER,
    "BirthDate" DATE,
    "IsEndangered" BOOLEAN DEFAULT FALSE,
    
    -- Standard metadata fields
    "IsDataMissing" BOOLEAN DEFAULT FALSE,
    "UserAuthorizationId" INTEGER DEFAULT 1,
    "DateAdded" TIMESTAMPTZ DEFAULT NOW(),
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW(),
    
    -- Primary Key predicate: ∀x,y ∈ Animal: (x.AnimalId = y.AnimalId) → (x = y)
    CONSTRAINT "PK_Zoo_Animal" PRIMARY KEY ("AnimalId"),
    
    -- Foreign key predicate: ∀x ∈ Animal: ∃y ∈ AnimalType: (x.AnimalTypeId = y.AnimalTypeId)
    -- Every animal must be of a known animal type
    CONSTRAINT "FK_Zoo_Animal_Zoo_AnimalType" FOREIGN KEY ("AnimalTypeId") 
        REFERENCES "Zoo"."AnimalType"("AnimalTypeId"),

    -- Not null predicates
    -- ∀x ∈ Animal: x.Name ≠ NULL
    CONSTRAINT "CHK_Zoo_Animal_Name" CHECK ("Name" IS NOT NULL),

    ```sql
    -- ∀x ∈ Animal: x.AnimalTypeId ≠ NULL
    CONSTRAINT "CHK_Zoo_Animal_AnimalTypeId" CHECK ("AnimalTypeId" IS NOT NULL),
    
    -- Default constraints
    -- ∀x ∈ Animal: (x.Name = NULL) → (x.Name := "Unnamed Animal")
    CONSTRAINT "DF_Zoo_Animal_Name" DEFAULT 'Unnamed Animal' FOR "Name",
    -- ∀x ∈ Animal: (x.AnimalTypeId = NULL) → (x.AnimalTypeId := -9999)
    CONSTRAINT "DF_Zoo_Animal_AnimalTypeId" DEFAULT -9999 FOR "AnimalTypeId",
    -- ∀x ∈ Animal: (x.BirthDate = NULL) → (x.BirthDate := "1900-01-01")
    CONSTRAINT "DF_Zoo_Animal_BirthDate" DEFAULT '1900-01-01' FOR "BirthDate"
);
```

#### PostgreSQL Standard Implementation

```sql
-- Animal types table
CREATE TABLE animal_types (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    habitat VARCHAR(50) NOT NULL,
    diet VARCHAR(30) NOT NULL
);

-- Individual animals table
CREATE TABLE animals (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    type_id INTEGER NOT NULL REFERENCES animal_types(id),
    birth_date DATE,
    is_endangered BOOLEAN DEFAULT FALSE
);
```

🔍 Exercise 2: Create INSERT statements to load sample data into the tables

#### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Business context: Populating the zoo database with species information
-- Predicate: ∀x in inserted rows: x ∈ AnimalType
INSERT INTO "Zoo"."AnimalType" ("Name", "Habitat", "Diet")
VALUES
    ('Lion', 'Savannah', 'Carnivore'),
    ('Elephant', 'Grassland', 'Herbivore');

-- Business context: Adding individual animals to the database
-- Predicate: ∀x in inserted rows: x ∈ Animal
INSERT INTO "Zoo"."Animal" ("Name", "AnimalTypeId", "BirthDate", "IsEndangered")
VALUES
    ('Leo', 1, '2015-05-12', FALSE),
    ('Dumbo', 2, '2010-02-28', TRUE);
```

#### PostgreSQL Standard Implementation

```sql
-- Insert sample animal types
INSERT INTO animal_types (name, habitat, diet)
VALUES
    ('Lion', 'Savannah', 'Carnivore'),
    ('Elephant', 'Grassland', 'Herbivore');

-- Insert sample animals
INSERT INTO animals (name, type_id, birth_date, is_endangered)
VALUES
    ('Leo', 1, '2015-05-12', FALSE),
    ('Dumbo', 2, '2010-02-28', TRUE);
```

⚠️ If you purposely introduce an error, like omitting a comma between value sets:

```sql
-- Incorrect syntax example
INSERT INTO "Zoo"."Animal" ("Name", "AnimalTypeId", "BirthDate", "IsEndangered")
VALUES
    ('Leo', 1, '2015-05-12', FALSE)  -- Missing comma here!
    ('Dumbo', 2, '2010-02-28', TRUE);
```

You would receive an error message similar to:

```markdown
ERROR: syntax error at or near "("
LINE 4:     ('Dumbo', 2, '2010-02-28', TRUE);
            ^
```

This error message indicates the problem is near the opening parenthesis of the second value set, and the actual issue is the missing comma between the value sets.

#### Implementation Analysis

- The Domain Driven Design approach with predicate logic creates self-documenting code through:
  - Formalized business rules expressed as logical predicates
  - Clear expression of entity relationships and constraints
  - Business-domain oriented schema and naming conventions
  - Constraint naming that explicitly states business purpose
- Every constraint directly maps to a business rule expressed as a formal logical predicate
- The addition of metadata fields creates a more robust data tracking system
- Two-value predicate logic eliminates NULL-related ambiguities through default values and explicit constraints
- The standard metadata fields provide audit trail, data quality tracking, and user accountability

[Back to Table of Contents](#table-of-contents)

## Summary

In this first chapter, you've learned the foundational skills for working with PostgreSQL:

1. **Database Creation**: You created a database to organize related tables and data
2. **Table Design**: You learned how to define tables with columns of specific data types
3. **Data Insertion**: You added rows of data to your tables using the INSERT statement
4. **Basic Syntax**: You experienced SQL syntax requirements, including semicolons and quoting rules

The Domain Driven Design approach with predicate logic demonstrated how these basic concepts can be enhanced with:

- Business domain organization through schema names
- Self-documenting table and constraint names
- Comprehensive metadata tracking
- Formal expression of business rules as logical predicates
- Elimination of NULL-related ambiguities through default values

These foundational skills set the stage for learning more complex database operations and queries in the chapters to come. In the next chapter, you'll learn how to query data using the SELECT statement, allowing you to extract insights from your tables.

[Back to Table of Contents](#table-of-contents)