# Chapter 7 - TABLE DESIGN THAT WORKS FOR YOU : PostgreSQL 17

## Table of Contents

- [Introduction](#introduction)
- [Key Concepts](#key-concepts)
- [PostgreSQL Standard Conventions](#postgresql-standard-conventions)
  - [Convention Principles](#convention-principles)
  - [Naming Patterns](#naming-patterns)
  - [Constraint Naming](#constraint-naming)
  - [Index and Key Management](#index-and-key-management)
- [Enterprise Heterogeneous Database Standard Conventions](#enterprise-heterogeneous-database-standard-conventions)

  - [Convention Principles](#enterprise-convention-principles)
  - [Naming Patterns](#enterprise-naming-patterns)
  - [Cross-Database Compatibility Considerations](#cross-database-compatibility-considerations)
- [Schema Organization](#schema-organization)
- [Table Design](#table-design)
- [Column Design](#column-design)
- [Constraint Management](#constraint-management)
- [Index Strategies](#index-strategies)
- [Domain and Type Management](#domain-and-type-management)
- [Performance Considerations](#performance-considerations)
- [PostgreSQL 17 Specific Features](#postgresql-17-specific-features)
- [Best Practices Summary](#best-practices-summary)
- [Hands-on Exercises](#hands-on-exercises)

---

## Introduction

This chapter presents best practices for table design in PostgreSQL 17, focusing on how to create efficient and maintainable database structures. We explore various principles of Domain Driven Database Design (DDDD) and integrate two-valued predicate logic to ensure that every database object expresses clear business rules and relationships.

[Back to Table of Contents](#table-of-contents)

## Key Concepts

Effective table design ensures that data is organized logically, which facilitates efficient data retrieval and manipulation. This chapter emphasizes the importance of clarity, consistency, and adherence to business rules in database design.

[Back to Table of Contents](#table-of-contents)

## PostgreSQL Standard Conventions

### Convention Principles

In PostgreSQL, using consistent naming conventions enhances readability and maintainability. The recommended practice is to use snake_case for identifiers.

### Naming Patterns

- **Tables**: Use plural nouns (e.g., `users`, `orders`).
- **Columns**: Use descriptive names (e.g., `user_id`, `order_date`).
- **Constraints**: Use clear prefixes (e.g., `pky` for primary keys, `fky` for foreign keys).

#### Example

**Business Context:** This table will store information about products available in the inventory.

#### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Business context: Inventory management
-- Predicate: Domain(Product) → Product ∈ Inventory
CREATE TABLE "Inventory"."Product" (
    "ProductId" BIGINT GENERATED ALWAYS AS IDENTITY CONSTRAINT "PK_Inventory_Product" PRIMARY KEY,
    "ProductName" VARCHAR(100) NOT NULL CONSTRAINT "CHK_Inventory_Product_ProductName" CHECK ("ProductName" IS NOT NULL),
    "CreatedAt" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Inventory_Product_CreatedAt",
    "IsDataMissing" BOOLEAN DEFAULT FALSE CONSTRAINT "DF_Inventory_Product_IsDataMissing",
    "UserAuthorizationId" INTEGER DEFAULT 1 CONSTRAINT "DF_Inventory_Product_UserAuthorizationId",
    "DateAdded" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Inventory_Product_DateAdded",
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Inventory_Product_DateOfLastUpdate"
    -- Check constraint for not null: ∀x ∈ Product: x.ProductName ≠ NULL
);
```

#### PostgreSQL Standard Implementation

```sql
CREATE TABLE inventory.products (
    product_id serial PRIMARY KEY,
    product_name varchar(100) NOT NULL,
    created_at timestamp with time zone DEFAULT current_timestamp
);
```

#### Implementation Analysis

- The DDDD approach formalizes business rules as logical predicates.
- By using explicit default values and constraints, we ensure data integrity and clarity.
- The standard PostgreSQL implementation lacks self-documenting constraint names and does not capture business rules as effectively.

[Back to Table of Contents](#table-of-contents)


## Enterprise Heterogeneous Database Standard Conventions

### Convention Principles

In enterprise environments, the use of PascalCase with double quotes preserves case sensitivity for identifiers.

### Naming Patterns

- **Tables**: Use singular nouns (e.g., `"Product"`).
- **Columns**: Use meaningful names with PascalCase (e.g., `"ProductId"`).

#### Example

**Business Context:** This table will hold customer information for the sales department.

#### Domain Driven Database Design Implementation with Predicate Logic

```sql
-- Business context: Customer management
-- Predicate: Domain(Customer) → Customer ∈ Sales
CREATE TABLE "Sales"."Customer" (
    "CustomerId" BIGINT GENERATED ALWAYS AS IDENTITY CONSTRAINT "PK_Sales_Customer" PRIMARY KEY,
    "FirstName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_Sales_Customer_FirstName" CHECK ("FirstName" IS NOT NULL),
    "LastName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_Sales_Customer_LastName" CHECK ("LastName" IS NOT NULL),
    "Email" VARCHAR(100) UNIQUE CONSTRAINT "UQ_Sales_Customer_Email",
    "IsDataMissing" BOOLEAN DEFAULT FALSE CONSTRAINT "DF_Sales_Customer_IsDataMissing",
    "UserAuthorizationId" INTEGER DEFAULT 1 CONSTRAINT "DF_Sales_Customer_UserAuthorizationId",
    "DateAdded" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Sales_Customer_DateAdded",
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Sales_Customer_DateOfLastUpdate"
    -- Primary Key predicate: ∀x,y ∈ Customer: (x.CustomerId = y.CustomerId) → (x = y)
);
```

#### PostgreSQL Standard Implementation

```sql
CREATE TABLE sales.customers (
    customer_id serial PRIMARY KEY,
    first_name varchar(50) NOT NULL,
    last_name varchar(50) NOT NULL,
    email varchar(100) UNIQUE
);
```

#### Implementation Analysis

- The DDDD version provides comprehensive constraints and metadata fields, enhancing data integrity.
- The standard implementation lacks self-documenting constraint names and does not express business rules as clearly.

---

## Schema Organization

### Best Practices for Schema Design

Organizing your schema effectively prevents confusion and enhances security. Consider the following:

1. **Separation of Concerns**: Group related tables into schemas.
2. **Access Control**: Implement role-based access at the schema level.

[Back to Table of Contents](#table-of-contents)


## 🔍 Table Design

### Single Responsibility Principle

Each table should focus on a single entity or concept, reducing complexity and improving data integrity.

### Normalization Considerations

Aim for at least Third Normal Form (3NF) to eliminate redundancy while maintaining relationships between tables.

---

## Column Design

### Data Type Selection Best Practices

Choose the most appropriate data type for each column based on the nature of the data it will store.

### Default Values and Constraints

Set default values and constraints (e.g., NOT NULL) to ensure data integrity at the time of insertion.

[Back to Table of Contents](#table-of-contents)


## Constraint Management

### Primary Key Strategies

Use surrogate keys when natural keys are not available or appropriate, ensuring that they are unique and not null.

### Foreign Key Implementation

Foreign keys should be used to enforce referential integrity between related tables.

[Back to Table of Contents](#table-of-contents)


## Index Strategies

### Index Types and Selection Criteria

Familiarize yourself with different index types (e.g., B-Tree, GIN) and choose based on query patterns.

### Multi-Column Indexes

Consider creating multi-column indexes for queries that filter on multiple columns to enhance performance.

[Back to Table of Contents](#table-of-contents)


## Domain and Type Management

### Creating Custom Domains and Types

Define custom domains when specific data validation is needed beyond standard SQL types, ensuring consistency.

[Back to Table of Contents](#table-of-contents)


## Performance Considerations

### Topic-Specific Performance Optimizations

Conduct regular performance assessments and optimize queries based on actual execution plans.

[Back to Table of Contents](#table-of-contents)


## PostgreSQL 17 Specific Features

### New Features Relevant to the Chapter Topic

PostgreSQL 17 introduces several enhancements that can aid in effective table design:

- **Improved Partitioning**: New partitioning strategies allow for more efficient data management.
- **Enhanced Indexing Options**: Support for new index types can optimize performance for specific query patterns.
- **Data Type Enhancements**: New data types or modifications to existing types can provide better data integrity and storage efficiency.

[Back to Table of Contents](#table-of-contents)


## Best Practices Summary

- **Consistent Naming Conventions**: Use clear and consistent naming conventions across your database for all objects.
- **Enforce Constraints**: Apply necessary constraints to maintain data integrity and enforce business rules.
- **Strategic Indexing**: Utilize indexes judiciously to enhance query performance while considering storage implications.
- **Organized Schema Design**: Structure your schema logically to facilitate access control and data management.
- **Documentation and Monitoring**: Maintain thorough documentation of schema designs and regularly monitor performance metrics.

[Back to Table of Contents](#table-of-contents)


## Hands-on Exercises

### Exercise 1: Create a Table Following PostgreSQL Standard Conventions

**Task:** Create a table named `employees` with the following columns:

- `employee_id` (BIGINT, generated always as identity, primary key)
- `first_name` (VARCHAR(50), NOT NULL)
- `last_name` (VARCHAR(50), NOT NULL)
- `hire_date` (DATE, default to current date)

#### Solution:

```sql
-- snake_case example
CREATE TABLE hr.employees (
    employee_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    hire_date DATE DEFAULT CURRENT_DATE,
    CONSTRAINT hr_employees_pky PRIMARY KEY (employee_id)
);
```

```sql
-- PascalCase example
CREATE TABLE "HR"."Employee" (
    "EmployeeId" BIGINT GENERATED ALWAYS AS IDENTITY CONSTRAINT "PK_HR_Employee" PRIMARY KEY,
    "FirstName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_HR_Employee_FirstName" CHECK ("FirstName" IS NOT NULL),
    "LastName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_HR_Employee_LastName" CHECK ("LastName" IS NOT NULL),
    "HireDate" DATE DEFAULT NOW() CONSTRAINT "DF_HR_Employee_HireDate"
);
```

### Exercise 2: Implement Foreign Key Constraints Between Two Related Tables

**Task:** Create a `departments` table and establish a foreign key relationship with the `employees` table.

#### Solution:

```sql
-- snake_case example
CREATE TABLE hr.departments (
    department_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL
);

ALTER TABLE hr.employees ADD COLUMN department_id BIGINT;
ALTER TABLE hr.employees ADD CONSTRAINT hr_employees_fky FOREIGN KEY (department_id) REFERENCES hr.departments(department_id);
```

```sql
-- PascalCase example
CREATE TABLE "HR"."Department" (
    "DepartmentId" BIGINT GENERATED ALWAYS AS IDENTITY CONSTRAINT "PK_HR_Department" PRIMARY KEY,
    "DepartmentName" VARCHAR(100) NOT NULL CONSTRAINT "CHK_HR_Department_DepartmentName" CHECK ("DepartmentName" IS NOT NULL)
);

ALTER TABLE "HR"."Employees" ADD COLUMN "DepartmentId" BIGINT;
ALTER TABLE "HR"."Employees" ADD CONSTRAINT "FK_HR_Employees_Departments_HR_Department_DepartmentId"
FOREIGN KEY ("DepartmentId") REFERENCES "HR"."Department"("DepartmentId");
```

### Exercise 3: Analyze Query Performance Before and After Adding Indexes

**Task:** Create an index on the `last_name` column of the `employees` table and measure query performance.

#### Solution:

```sql
-- snake_case example
CREATE INDEX idx_last_name ON hr.employees (last_name);

-- Measure performance (example query)
EXPLAIN ANALYZE SELECT * FROM hr.employees WHERE last_name = 'Smith';
```
```sql
-- PascalCase example
CREATE INDEX "Idx_LastName" ON "HR"."Employees" ("LastName");

-- Measure performance (example query)
EXPLAIN ANALYZE SELECT * FROM "HR"."Employees" WHERE "LastName" = 'Smith';
```

### Solutions Provided with Explanation

1. **Exercise 1 Explanation**: The `employees` table is designed to maintain basic employee information, ensuring that each row has a unique identifier (`employee_id`) and mandatory name fields. The use of default values helps streamline data entry.

2. **Exercise 2 Explanation**: The foreign key constraint between `employees` and `departments` ensures that each employee is associated with a valid department, maintaining referential integrity across the database.

3. **Exercise 3 Explanation**: By creating an index on the `last_name` column, queries filtering by last name will execute faster due to the database bypassing a full table scan and directly accessing the indexed data.

[Back to Table of Contents](#table-of-contents)

This concludes Chapter 7 on Table Design Best Practices for PostgreSQL 17. By following these principles, you can create databases that are efficient, reliable, and easy to maintain. In the next chapter, we will delve into SQL aggregate functions, exploring how to assess data quality and derive meaningful insights from your datasets.