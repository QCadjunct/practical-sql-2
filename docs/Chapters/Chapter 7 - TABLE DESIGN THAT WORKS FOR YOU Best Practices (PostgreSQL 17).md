# 📊 PostgreSQL 17 Table Design - Best Practices Guide

## Pairs programmining with Claude.ai

### **You're absolutely right! That's the beauty of pair programming - having someone to catch those details that might otherwise slip through the cracks.Your vigilance in reviewing has helped create a much more accurate and consistent document. The terminology correction from "Domain Driven Design" to "Domain Driven Database Design" is an important distinction that makes the guide more precise and focused on its specific database implementation aspects. This collaboration demonstrates why code and documentation reviews are such a crucial part of the development process. Even when someone produces excellent work (which I appreciate you calling "amazing"), having a second set of eyes can catch inconsistencies, terminology errors, or conceptual issues  that might otherwise be missed.**

**Is there anything else in the document you'd like me to update or adjust? I'm happy to continue improving it under your guidance.**

<div align="center">
  <img src="https://www.postgresql.org/media/img/about/press/elephant.png" alt="PostgreSQL Logo" width="200"/>
</div>

## Table of Contents <a name="toc"></a>
- [Introduction](#introduction)
- [Key Design Philosophies](#philosophies)
  - [PostgreSQL Standard vs. Domain Driven Design](#standard-vs-ddd)
  - [Philosophical Differences](#philosophical-differences)
  - [Implementation Differences](#implementation-differences)
  - [Practical Impact](#practical-impact)
- [Naming Conventions](#naming-conventions)
  - [Naming Conventions Grid](#naming-grid)
  - [PostgreSQL Standard (snake_case)](#postgres-standard)
  - [Domain Driven Design (PascalCase)](#domain-driven)
- [Primary and Foreign Keys](#keys)
  - [Natural vs. Surrogate Keys](#natural-vs-surrogate)
  - [Foreign Keys](#foreign-keys)
  - [Automatically Deleting Records with CASCADE](#cascade)
- [Data Integrity Constraints](#constraints)
  - [NOT NULL Constraint](#not-null)
  - [CHECK Constraint](#check)
  - [UNIQUE Constraint](#unique)
  - [Three-Predicate vs. Two-Predicate Logic](#predicate-logic)
- [Indexing Strategies](#indexing)
  - [B-Tree Index](#b-tree)
  - [When to Use Indexes](#when-to-index)
  - [Benchmarking with EXPLAIN](#explain)
- [Performance Considerations](#performance)
- [Table Modifications and Alterations](#modifications)
- [Schema Management Patterns](#schema-management)
- [Advanced Table Design Techniques](#advanced)
- [Effective Testing Strategies](#testing)
- [Best Practices Summary](#summary)
- [Examples](#examples)
  - [Vinyl Collection Database](#vinyl-database)
- [Conclusion](#conclusion)
- [Additional Resources](#resources)

## Introduction <a name="introduction"></a> [↩️](#toc)

Effective table design forms the foundation of maintainable, efficient, and reliable database systems. This guide explores best practices for designing PostgreSQL 17 tables that work optimally for both development and data analysis. Well-designed tables simplify information retrieval, enforce data integrity, and ensure consistent performance across your database.

Good database design is like organizing your home - when everything has its proper place, finding what you need becomes effortless. Similarly, when database tables are thoughtfully structured, querying and maintaining data becomes much more manageable.

## Key Design Philosophies <a name="philosophies"></a> [↩️](#toc)

### PostgreSQL Standard vs. Domain Driven Database Design <a name="standard-vs-ddd"></a> [↩️](#toc)

Two primary design philosophies exist in PostgreSQL database design, each with its own strengths and ideal use cases:

### Philosophical Differences <a name="philosophical-differences"></a> [↩️](#toc)

**PostgreSQL Standard (snake_case):**
- ✅ Emphasizes technical simplicity and direct access
- ✅ Focused on database performance and administration
- ✅ Treats database as a data storage layer
- ✅ Favors pragmatic, functional naming over strict formality
- ✅ Accommodates NULL values as a valid state for optional data
- ✅ More lenient on constraints, allowing flexibility

**Domain Driven Database Design (PascalCase):**
- ✅ Emphasizes business domain modeling
- ✅ Aligns database structure with business concepts
- ✅ Treats database as a core domain model repository
- ✅ Enforces strict naming that mirrors business terminology
- ✅ Eliminates NULL values through defaults and constraints
- ✅ Enforces strict validation to maintain data integrity

### Implementation Differences <a name="implementation-differences"></a> [↩️](#toc)

**PostgreSQL Standard:**
```sql
CREATE TABLE sales.customer_orders (
    order_id int GENERATED ALWAYS AS IDENTITY,
    customer_id int NOT NULL,
    order_date timestamp DEFAULT now(),
    total_amount numeric(10,2),
    status varchar(20),
    CONSTRAINT customer_orders_pkey PRIMARY KEY (order_id),
    CONSTRAINT customer_orders_customer_fky FOREIGN KEY (customer_id) 
        REFERENCES customer.customers(customer_id)
);
```

**Domain Driven Database Design:**
```sql
CREATE TABLE "Sales"."CustomerOrder" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "CustomerId" int NOT NULL,
    "OrderDate" timestamp NOT NULL CONSTRAINT "DF_Sales_CustomerOrder_OrderDate" DEFAULT now(),
    "TotalAmount" numeric(10,2) NOT NULL CONSTRAINT "DF_Sales_CustomerOrder_TotalAmount" DEFAULT 0,
    "Status" varchar(20) NOT NULL CONSTRAINT "DF_Sales_CustomerOrder_Status" DEFAULT 'Pending',
    CONSTRAINT "PK_Sales_CustomerOrder" PRIMARY KEY ("Id"),
    CONSTRAINT "FK_Sales_CustomerOrder_Customer_Customer" FOREIGN KEY ("CustomerId") 
        REFERENCES "Customer"."Customer"("Id"),
    CONSTRAINT "CHK_Sales_CustomerOrder_TotalAmount" CHECK ("TotalAmount" >= 0),
    CONSTRAINT "CHK_Sales_CustomerOrder_Status" CHECK ("Status" IN ('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled'))
);
```

### Practical Impact <a name="practical-impact"></a> [↩️](#toc)

1. **Code Readability and Audience:**
   - 👨‍💻 PostgreSQL Standard is more familiar to DBAs and backend developers
   - 👩‍💼 Domain Driven Design is more accessible to business analysts and domain experts

2. **Data Quality:**
   - PostgreSQL Standard allows more flexibility but may permit inconsistent data
   - Domain Driven Design enforces stricter validation but requires more upfront design

3. **Maintenance:**
   - PostgreSQL Standard is easier to create initially but may require more validation in application code
   - Domain Driven Design requires more initial database setup but centralizes business rules

4. **Scalability:**
   - PostgreSQL Standard has lower overhead for simple CRUD operations
   - Domain Driven Design provides better structure for complex domain logic

The choice between these approaches depends on project requirements, team composition, and whether the database is primarily a persistence layer or the central repository of business logic.

## Naming Conventions <a name="naming-conventions"></a> [↩️](#toc)

### Naming Conventions Grid <a name="naming-grid"></a> [↩️](#toc)

| **Database Object**             | **PostgreSQL Standard (snake_case)**                              | **Domain Driven Database Design Standard (PascalCase)**                                     |
|---------------------------------|--------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Schema Names**                | `{schema_name}`                                                    | `"{SchemaName}"`                                                                  |
| **Table Names**                 | `{schema_name}.{table_name}`                                       | `"{SchemaName}"."{TableName}"`                                                    |
| **Column Names**                | `{column_name}`                                                    | `"{ColumnName}"` (Primary keys use `"{EntityName}Id}"` pattern)                |
| **Primary Key Constraint**      | `{schema_name}_{table_name}_pky`                                   | `"PK_{SchemaName}_{TableName}"`                                                   |
| **Foreign Key Constraint**      | `{schema_name}_{table_name}_{referenced_table}_fky`                | `"FK_{SchemaName}_{TableName}_{ReferencedSchemaName}_{ReferencedTable}"`          |
| **Check Column Constraint**     | `{schema_name}_{table_name}_{condition}_check`                     | `"CHK_{SchemaName}_{TableName}_{ColumnName}"`                                     |
| **Check Table Constraint**      | `{schema_name}_{table_name}_{condition}_check`                     | `"CHK_{SchemaName}_{TableName}_{TableValidationConditionName}"`                   |
| **Check Database Constraint**   | `{schema_name}_{table_name}_{condition}_check`                     | `"CHK_{SchemaName}_{TableName}_{DatabaseValidationConditionName}"`                |
| **Unique Constraint**           | `{schema_name}_{table_name}_{columns}_key`                         | `"UQ_{SchemaName}_{TableName}_{ColumnName}"`                                      |
| **Default Constraint**          | `{schema_name}_{table_name}_{column_name}_default`                 | `"DF_{SchemaName}_{TableName}_{ColumnName}"`                                      |
| **Index Names**                 | `{schema_name}_{table_name}_{column_name}_idx`                     | `"IX_{SchemaName}_{TableName}_{ColumnName}"`                                      |
| **View Names**                  | `{schema_name}.vw_{view_purpose}`                                  | `"{SchemaName}"."VW_{ViewPurpose}"`                                               |
| **Function Names**              | `{schema_name}.{verb}_{object}[_{qualifier}]()`                    | `"{SchemaName}"."{Verb}{Object}[{Qualifier}]"()`                                  |
| **Trigger Names**               | `{schema_name}_{table_name}_{timing}_{action}_trg`                 | `"TR_{SchemaName}_{TableName}_{TimingAction}"`                                    |
| **Sequence Names**              | `{schema_name}.{table_name}_{column_name}_seq`                     | `"{SchemaName}"."{TableName}_{ColumnName}_Seq"`                                   |
| **Materialized View**           | `{schema_name}.mv_{view_purpose}`                                  | `"{SchemaName}"."MV_{ViewPurpose}"`                                               |
| **Enum Type**                   | `{schema_name}.{type_name}_type`                                   | `"{SchemaName}"."{TypeName}Type"`                                                 |
| **Domain Type**                 | `{schema_name}.{domain_purpose}_domain`                            | `"{SchemaName}"."{DomainPurpose}Domain"`                                          |
| **Composite Type**              | `{schema_name}.{type_purpose}_type`                                | `"{SchemaName}"."{TypePurpose}Type"`                                              |
| **Partition Table**             | `{schema_name}.{parent_table}_{partition_key}`                     | `"{SchemaName}"."{ParentTable}_{PartitionKey}"`                                   |
| **Temporary Table**             | `{schema_name}.tmp_{purpose}`                                      | `"{SchemaName}"."Tmp_{Purpose}"`                                                  |
| **Audit Table**                 | `{schema_name}.{table_name}_audit`                                 | `"Audit"."{TableName}History"`                                               |
| **Junction Table or**           | `{schema_name}.{table1}_{table2}`                                  | `"{SchemaName}"."{Table1}{Table2}"`                                               |
| **Association Table**           | `{schema_name}.{table1}_{table2}`                                  | `"{SchemaName}"."{Table1}{Table2}"`                                               |

### PostgreSQL Standard (snake_case) <a name="postgres-standard"></a> [↩️](#toc)

PostgreSQL traditionally follows these naming conventions:

- ✅ All lowercase letters
- ✅ Words separated by underscores
- ✅ No spaces or special characters
- ✅ Avoid using SQL reserved keywords as identifiers
- ✅ Descriptive but concise names
- ✅ Uses UTF-8 as the default encoding

PostgreSQL treats unquoted identifiers as case-insensitive. The convention is to use snake_case formatting such as `products`, `order_items`, or `customer_addresses`. This improves readability while avoiding the need for quoting identifiers in queries.

#### Implementation Example
```sql
CREATE TABLE inventory.products (
    id int GENERATED ALWAYS AS IDENTITY,
    product_code varchar(50) NOT NULL,
    name varchar(100) NOT NULL,
    description text,
    unit_price numeric(10,2) NOT NULL,
    created_at timestamp NOT NULL DEFAULT now(),
    updated_at timestamp NOT NULL DEFAULT now(),
    CONSTRAINT inventory_products_pky PRIMARY KEY (id),
    CONSTRAINT inventory_products_product_code_key UNIQUE (product_code)
);

CREATE INDEX inventory_products_name_idx ON inventory.products(name);
```

### Domain Driven Database Design (PascalCase) <a name="domain-driven"></a> [↩️](#toc)

Domain-Driven Database Design follows these conventions:

- ✅ Initial capital letters for each word
- ✅ Objects enclosed in double quotes
- ✅ No underscores between words
- ✅ Fully qualified object names (schema and object name)
- ✅ Alignment with business domain language
- ✅ Uses Unicode for encoding
- ✅ Primary key columns named with entity-specific naming pattern `{EntityName}Id` (e.g., "CustomerId", "ProductId", "OrderId")

#### Domain Driven Database Design Standard Principles
- All objects must be uniquely defined following the naming convention
- Adheres to SOLID Design Principles and DRY (Don't Repeat Yourself)
- Database serves as a centralized repository for all business logic
- Facilitates automated code generation through the Business Logic Layer (BLL) and Frontend UI
- Required columns should have a check constraint and default value for missing data, eliminating three-predicate logic (NULL, empty, value) in favor of two-predicate logic (default, value)

#### Implementation Example
```sql
CREATE SCHEMA "Inventory";

CREATE TABLE "Inventory"."Product" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "ProductCode" varchar(50) NOT NULL,
    "Name" varchar(100) NOT NULL,
    "Description" text NOT NULL CONSTRAINT "DF_Inventory_Product_Description" DEFAULT 'No Description',
    "UnitPrice" numeric(10,2) NOT NULL,
    "CreatedAt" timestamp NOT NULL CONSTRAINT "DF_Inventory_Product_CreatedAt" DEFAULT now(),
    "UpdatedAt" timestamp NOT NULL CONSTRAINT "DF_Inventory_Product_UpdatedAt" DEFAULT now(),
    CONSTRAINT "PK_Inventory_Product" PRIMARY KEY ("Id"),
    CONSTRAINT "UQ_Inventory_Product_ProductCode" UNIQUE ("ProductCode"),
    CONSTRAINT "CHK_Inventory_Product_Description" CHECK ("Description" <> NULL)
);

CREATE INDEX "IX_Inventory_Product_Name" ON "Inventory"."Product"("Name");
```

Notice how in the Domain-Driven Design example, the `Description` field has both a meaningful default value ('No Description') and a check constraint to ensure it's not NULL, eliminating the three-state logic problem and enforcing data integrity according to domain rules.

## Primary and Foreign Keys <a name="keys"></a> [↩️](#toc)

### Natural vs. Surrogate Keys <a name="natural-vs-surrogate"></a> [↩️](#toc)

#### Natural Keys
- 📝 Use existing meaningful data as identifiers (e.g., ISBN for books)
- ✅ Advantages:
  - Data already exists in the table
  - Has business meaning, reducing need for joins
- 📋 Requirements:
  - Must be unique for every row
  - Cannot contain NULL values
  - Should remain relatively stable

#### Surrogate Keys
- 🔑 Artificially created values with no business meaning
- 💡 Common implementations:
  - Auto-incrementing integers using `GENERATED ALWAYS AS IDENTITY`
  - UUIDs (Universally Unique Identifiers)
- ✅ Advantages:
  - Not affected by changes to business data
  - More compact storage than typical natural keys
  - Consistent format across all tables

### Foreign Keys <a name="foreign-keys"></a> [↩️](#toc)

Foreign keys enforce referential integrity between tables by ensuring that values in a column match values in another table's primary key. They prevent orphaned records by:

1. Requiring that foreign key values exist in the referenced table
2. Optionally removing dependent rows when a parent row is deleted (using `ON DELETE CASCADE`)

```sql
CREATE TABLE "Sales"."Order" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "CustomerId" int NOT NULL CONSTRAINT "FK_Sales_Order_Customers_Customer" 
        REFERENCES "Customers"."Customer"("Id") ON DELETE CASCADE,
    -- Additional columns
    CONSTRAINT "PK_Sales_Order" PRIMARY KEY ("Id")
);
```

### Automatically Deleting Related Records with CASCADE <a name="cascade"></a> [↩️](#toc)

To delete a row in a parent table and have that action automatically delete any related rows in child tables, you can specify that behavior by adding `ON DELETE CASCADE` when defining the foreign key constraint:

```sql
CREATE TABLE registrations (
    registration_id varchar(10),
    registration_date date,
    license_id varchar(10) REFERENCES licenses (license_id) ON DELETE CASCADE,
    CONSTRAINT registration_key PRIMARY KEY (registration_id, license_id)
);
```

Now, deleting a row in the licenses table will also delete all related rows in the registrations table. This maintains data integrity by ensuring deleting a parent record doesn't leave orphaned rows in related tables.

## Data Integrity Constraints <a name="constraints"></a> [↩️](#toc)

### NOT NULL Constraint <a name="not-null"></a> [↩️](#toc)

Prevents NULL values in a column, ensuring required data is always present.

**PostgreSQL Standard:**
```sql
CREATE TABLE students (
    id int GENERATED ALWAYS AS IDENTITY,
    first_name varchar(50) NOT NULL,
    last_name varchar(50) NOT NULL,
    email varchar(100) NOT NULL
);
```

**Domain Driven Design:**
```sql
CREATE TABLE "School"."Student" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "FirstName" varchar(50) NOT NULL,
    "LastName" varchar(50) NOT NULL,
    "Email" varchar(100) NOT NULL,
    CONSTRAINT "PK_School_Student" PRIMARY KEY ("Id")
);
```

### CHECK Constraint <a name="check"></a> [↩️](#toc)

Validates data against specified conditions before accepting it.

**PostgreSQL Standard:**
```sql
CREATE TABLE products (
    id int GENERATED ALWAYS AS IDENTITY,
    name varchar(100) NOT NULL,
    price numeric(10,2) NOT NULL,
    CONSTRAINT check_price_positive CHECK (price > 0)
);
```

**Domain Driven Database Design:**
```sql
CREATE TABLE "Inventory"."Product" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "Name" varchar(100) NOT NULL,
    "Price" numeric(10,2) NOT NULL,
    CONSTRAINT "CHK_Inventory_Product_Price" CHECK ("Price" > 0),
    CONSTRAINT "PK_Inventory_Product" PRIMARY KEY ("Id")
);
```

### UNIQUE Constraint <a name="unique"></a> [↩️](#toc)

Ensures all values in a column or group of columns are unique.

**PostgreSQL Standard:**
```sql
CREATE TABLE departments (
    id int GENERATED ALWAYS AS IDENTITY,
    dept_code varchar(10) NOT NULL,
    name varchar(50) NOT NULL,
    CONSTRAINT dept_code_unique UNIQUE (dept_code)
);
```

**Domain Driven Database Design:**
```sql
CREATE TABLE "Company"."Department" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "DeptCode" varchar(10) NOT NULL,
    "Name" varchar(50) NOT NULL,
    CONSTRAINT "UQ_Company_Department_DeptCode" UNIQUE ("DeptCode"),
    CONSTRAINT "PK_Company_Department" PRIMARY KEY ("Id")
);
```

### Three-Predicate vs. Two-Predicate Logic <a name="predicate-logic"></a> [↩️](#toc)

In traditional database design (PostgreSQL Standard), columns can exist in three states:
1. NULL (no value)
2. Empty value (e.g., empty string, zero)
3. Meaningful value

The Domain-Driven Database Design approach eliminates this ambiguity by:
1. Making columns NOT NULL with meaningful defaults
2. Adding CHECK constraints to validate values
3. Ensuring consistency between database and application logic

This two-predicate logic simplifies data validation and business rules by removing the NULL consideration and focusing only on whether a value is default or meaningful.

## Indexing Strategies <a name="indexing"></a> [↩️](#toc)

### B-Tree Index <a name="b-tree"></a> [↩️](#toc)

PostgreSQL's default index type is the B-Tree (Balanced Tree) index, which works well for:
- Equality operators (=, <>)
- Range operators (<, <=, >, >=)
- BETWEEN and IN operators
- Pattern matching with LIKE 'word%' (prefix matching)

**PostgreSQL Standard:**
```sql
CREATE INDEX customer_email_idx ON customers(email);
```

**Domain Driven Database Design:**
```sql
CREATE INDEX "IX_Customers_Customer_Email" ON "Customers"."Customer"("Email");
```

### When to Use Indexes <a name="when-to-index"></a> [↩️](#toc)

- 🔍 Columns frequently used in WHERE clauses
- 🔄 Columns used in JOIN conditions
- 🔗 Foreign key columns
- 🧮 Columns with high cardinality (many unique values)

### Benchmarking with EXPLAIN <a name="explain"></a> [↩️](#toc)

PostgreSQL's `EXPLAIN ANALYZE` command helps measure index performance:

```sql
-- Without index
EXPLAIN ANALYZE SELECT * FROM large_table WHERE frequently_searched_column = 'value';

-- Add index
CREATE INDEX idx_frequently_searched ON large_table(frequently_searched_column);

-- After index
EXPLAIN ANALYZE SELECT * FROM large_table WHERE frequently_searched_column = 'value';
```

## Performance Considerations <a name="performance"></a> [↩️](#toc)

### Identity Columns ⚡

- Use `GENERATED ALWAYS AS IDENTITY` or `GENERATED BY DEFAULT AS IDENTITY` instead of `serial` or `bigserial` for better standards compliance and identity management
- The `GENERATED ALWAYS` syntax prevents manual inserts into the identity column, enforcing better data integrity
- The `GENERATED BY DEFAULT` syntax allows manual inserts when needed (such as during data migrations)

```sql
-- PostgreSQL Standard
CREATE TABLE products (
    id int GENERATED ALWAYS AS IDENTITY,
    name varchar(100) NOT NULL
);

-- Domain Driven Database Design
CREATE TABLE "Inventory"."Product" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "Name" varchar(100) NOT NULL
);
```

### Data Types 📊

- Choose appropriate data types that balance storage efficiency with query performance
- Use the most restrictive data type that will accommodate your data
- Consider these guidelines:
  - For whole numbers: `smallint` (2 bytes), `integer` (4 bytes), `bigint` (8 bytes)
  - For decimal values: `numeric(precision, scale)` for exact values, `real` or `double precision` for inexact values
  - For text: `varchar(n)` for variable-length with limit, `text` for unlimited length
  - For dates/times: `date`, `time`, `timestamp`, `interval` based on requirements
  - For boolean values: `boolean` instead of integer flags

### Column Ordering 🔄

- Place frequently accessed columns earlier in the table definition
- Group related columns together for better logical organization
- Consider placing fixed-width columns before variable-width columns
- When possible, align column ordering with common access patterns

### Table Partitioning 📂

For very large tables (millions of rows), consider partitioning to improve performance:

```sql
-- PostgreSQL Standard (Range Partitioning)
CREATE TABLE measurements (
    city_id int not null,
    logdate date not null,
    peaktemp int,
    unitsales int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurements_y2020 PARTITION OF measurements
    FOR VALUES FROM ('2020-01-01') TO ('2021-01-01');

CREATE TABLE measurements_y2021 PARTITION OF measurements
    FOR VALUES FROM ('2021-01-01') TO ('2022-01-01');
```

### Vacuuming and Maintenance 🧹

- Design with VACUUM operations in mind to minimize maintenance impact
- Set appropriate `fillfactor` values for tables with frequent updates
- Consider the impact of bloat on tables with frequent updates and deletes
- Implement a regular maintenance schedule for vacuuming and analyzing tables

### Query Access Patterns 🔍

- Structure tables based on how they will be queried most frequently
- Consider denormalization for read-heavy workloads when appropriate
- Use materialized views for complex, frequently-executed queries
- Add appropriate indexes based on common WHERE clauses and JOIN conditions

## Table Modifications and Alterations <a name="modifications"></a> [↩️](#toc)

After creating tables, you may need to modify their structure as requirements evolve. PostgreSQL provides the `ALTER TABLE` command for this purpose.

### Adding, Removing, and Modifying Constraints

You can add or remove constraints on existing tables:

```sql
-- Adding a primary key
ALTER TABLE table_name ADD CONSTRAINT constraint_name PRIMARY KEY (column_name);

-- Adding a foreign key
ALTER TABLE table_name 
ADD CONSTRAINT constraint_name FOREIGN KEY (column_name) 
REFERENCES referenced_table(referenced_column);

-- Adding a check constraint
ALTER TABLE table_name 
ADD CONSTRAINT constraint_name CHECK (condition);

-- Removing a constraint
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```

### Adding and Removing Columns

```sql
-- Adding a column
ALTER TABLE table_name ADD COLUMN column_name data_type [constraints];

-- Removing a column
ALTER TABLE table_name DROP COLUMN column_name;
```

### Modifying Column Properties

```sql
-- Changing a column's data type
ALTER TABLE table_name ALTER COLUMN column_name TYPE new_data_type;

-- Adding NOT NULL constraint
ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;

-- Removing NOT NULL constraint
ALTER TABLE table_name ALTER COLUMN column_name DROP NOT NULL;

-- Setting a default value
ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT expression;

-- Removing a default value
ALTER TABLE table_name ALTER COLUMN column_name DROP DEFAULT;
```

### Renaming Objects

```sql
-- Renaming a table
ALTER TABLE table_name RENAME TO new_table_name;

-- Renaming a column
ALTER TABLE table_name RENAME COLUMN column_name TO new_column_name;

-- Renaming a constraint
ALTER TABLE table_name RENAME CONSTRAINT old_constraint_name TO new_constraint_name;
```

## Schema Management Patterns <a name="schema-management"></a> [↩️](#toc)

### Schema Organization Strategies

1. **Functional Grouping:**
   - Group related tables by functionality (e.g., `auth`, `reporting`, `customer`)
   - Simplifies access control and logical organization

2. **Bounded Context Approach (DDBD):**
   - Create schemas that align with domain bounded contexts
   - Enforces clear boundaries between different parts of the system

3. **Multi-tenant Isolation:**
   - Separate schemas for each tenant in a multi-tenant application
   - Provides data isolation and simplifies tenant-specific operations

### Schema Migration and Version Control

1. **Migration Tools:**
   - Use tools like Flyway, Liquibase, or frameworks with built-in migration support
   - Scripts should be idempotent when possible
   - Keep migrations in version control alongside application code

2. **Forward-Only Migrations:**
   - Design migrations to only move forward, not backward
   - For rollbacks, create new forward migrations that undo changes

3. **CI/CD Integration:**
   - Automate schema changes as part of your CI/CD pipeline
   - Test migrations in non-production environments before applying to production

## Advanced Table Design Techniques <a name="advanced"></a> [↩️](#toc)

### Table Inheritance

PostgreSQL supports table inheritance, allowing you to define parent-child table relationships. This can be useful for organizing related data that shares common columns:

```sql
-- Parent table
CREATE TABLE cities (
    name            text,
    population      float,
    elevation       int     -- in feet
);

-- Child table inherits all columns
CREATE TABLE capitals (
    state           char(2)
) INHERITS (cities);
```

### Partial Indexes

For tables where only a subset of rows are frequently queried, partial indexes can improve performance by indexing only those rows:

```sql
-- Index only active users
CREATE INDEX active_users_idx ON users (username) WHERE active = true;
```

### Expression Indexes

Index expressions rather than just columns for more complex query optimization:

```sql
-- Index lowercase email for case-insensitive searches
CREATE INDEX users_email_lower_idx ON users (lower(email));

-- This allows efficient queries like:
SELECT * FROM users WHERE lower(email) = 'example@domain.com';
```

### Composite Indexes

Create indexes on multiple columns for queries that filter or join on multiple fields:

```sql
-- Index for queries filtering on both columns
CREATE INDEX products_category_name_idx ON products (category_id, name);
```

### Generated Columns

PostgreSQL 17 supports generated columns that compute their values from other columns:

```sql
-- Standard approach
CREATE TABLE rectangles (
    id int GENERATED ALWAYS AS IDENTITY,
    length numeric NOT NULL,
    width numeric NOT NULL,
    area numeric GENERATED ALWAYS AS (length * width) STORED
);

-- Domain Driven Database Design approach
CREATE TABLE "Shapes"."Rectangle" (
    "Id" int GENERATED ALWAYS AS IDENTITY,
    "Length" numeric NOT NULL,
    "Width" numeric NOT NULL,
    "Area" numeric GENERATED ALWAYS AS ("Length" * "Width") STORED,
    CONSTRAINT "PK_Shapes_Rectangle" PRIMARY KEY ("Id")
);
```

## Effective Testing Strategies <a name="testing"></a> [↩️](#toc)

Testing your database design before deploying to production is crucial. Here are some strategies to ensure your table design works as expected:

### Volume Testing

Test your table design with realistic data volumes:

```sql
-- Generate test data
INSERT INTO large_table (column1, column2, column3)
SELECT 
    'Value ' || i,
    random() * 1000,
    CASE WHEN random() > 0.5 THEN true ELSE false END
FROM generate_series(1, 1000000) i;

-- Test query performance with EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT * FROM large_table
WHERE column2 > 500 AND column3 = true;
```

### Constraint Validation

Test that your constraints properly enforce data integrity:

```sql
-- Should fail if constraints are working properly
BEGIN;
    -- Test NOT NULL constraint
    INSERT INTO products (name) VALUES (NULL);
    
    -- Test CHECK constraint
    INSERT INTO products (name, price) VALUES ('Test Product', -10);
    
    -- Test UNIQUE constraint
    INSERT INTO products (name, price) VALUES ('Existing Product', 100);
    INSERT INTO products (name, price) VALUES ('Existing Product', 200);
    
    -- Test foreign key constraint
    INSERT INTO order_items (order_id, product_id) VALUES (999999, 1);
ROLLBACK;
```

### Edge Case Testing

Test boundary conditions and special cases:

```sql
-- Test maximum length strings
INSERT INTO users (username, email) 
VALUES (
    repeat('x', 50), -- username with max length
    repeat('a', 100) || '@example.com' -- email with max length
);

-- Test special characters
INSERT INTO content (title, description)
VALUES (
    'Title with special chars: &<>"''',
    'Description with emojis: 😀🙌👍'
);
```

## Best Practices Summary <a name="summary"></a> [↩️](#toc)

### Design Principles

- ✅ Choose a consistent naming convention and apply it throughout