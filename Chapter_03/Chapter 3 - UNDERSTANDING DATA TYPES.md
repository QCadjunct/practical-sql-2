# Chapter 3: UNDERSTANDING DATA TYPES

## Table of Contents
- [Introduction](#introduction)
- [Characters](#characters)
  - [char(n)](#charn)
  - [varchar(n)](#varcharn)
  - [text](#text)
  - [Character Types in Action](#character-types-in-action)
- [Numbers](#numbers)
  - [Integers](#integers)
  - [Auto-Incrementing Integers](#auto-incrementing-integers)
  - [Decimal Numbers](#decimal-numbers)
  - [Fixed-Point Numbers](#fixed-point-numbers)
  - [Floating-Point Types](#floating-point-types)
  - [Number Data Types in Action](#number-data-types-in-action)
  - [Trouble with Floating-Point Math](#trouble-with-floating-point-math)
  - [Choosing Your Number Data Type](#choosing-your-number-data-type)
- [Dates and Times](#dates-and-times)
  - [Date and Time Types in Action](#date-and-time-types-in-action)
  - [Using the interval Data Type in Calculations](#using-the-interval-data-type-in-calculations)
- [Miscellaneous Types](#miscellaneous-types)
- [Transforming Values with CAST](#transforming-values-with-cast)
- [Try It Yourself Exercises](#try-it-yourself-exercises)
- [Summary](#summary)

## Introduction

Understanding data types is fundamental to building effective databases and performing accurate analysis. In PostgreSQL, each column in a table must have exactly one data type, which is defined during table creation. Proper data type selection affects storage efficiency, data integrity, and calculation accuracy.

The three main categories of data types covered in this chapter are:

1. **Characters**: For storing text, numbers, and symbols as strings
2. **Numbers**: For storing numeric values and performing calculations 
3. **Dates and times**: For handling temporal information

### Domain Driven Database Design Implementation with Predicate Logic

**Business Context:** Wildlife conservation program tracking eagle observations to monitor population health

```sql
-- Business context: Wildlife conservation monitoring program
-- Domain predicate: Domain(EagleWatch) → EagleWatch ∈ Wildlife
CREATE TABLE "Wildlife"."EagleWatch" (
    "EagleWatchId" SERIAL, -- Unique observation identifier
    "ObservedDate" DATE, -- Date when eagles were observed
    "EaglesSeen" INTEGER, -- Count of eagles observed
    
    -- Standard metadata fields tracking data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ EagleWatch: (x.EagleWatchId = y.EagleWatchId) → (x = y)
    CONSTRAINT "PK_Wildlife_EagleWatch" PRIMARY KEY ("EagleWatchId"),
    
    -- Not null predicates:
    -- ∀x ∈ EagleWatch: x.ObservedDate ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_ObservedDate" CHECK ("ObservedDate" IS NOT NULL),
    -- ∀x ∈ EagleWatch: x.EaglesSeen ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_EaglesSeen" CHECK ("EaglesSeen" IS NOT NULL),
    -- ∀x ∈ EagleWatch: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ EagleWatch: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ EagleWatch: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ EagleWatch: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Wildlife_EagleWatch_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Data validation predicate: ∀x ∈ EagleWatch: x.EaglesSeen ≥ 0
    CONSTRAINT "CHK_Wildlife_EagleWatch_MinimumEagles" CHECK ("EaglesSeen" >= 0),
    
    -- Default constraint predicates:
    -- ∀x ∈ EagleWatch: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Wildlife_EagleWatch_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ EagleWatch: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Wildlife_EagleWatch_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ EagleWatch: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Wildlife_EagleWatch_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ EagleWatch: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Wildlife_EagleWatch_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE eagle_watch (
    observed_date date,
    eagles_seen integer
);
```

### Implementation Analysis
- The DDDD approach makes the business domain explicit through schema naming and constraints
- Every column has explicit NOT NULL checks and appropriate default values
- Standard metadata fields provide quality tracking and audit capabilities
- Business rules are formalized through predicates (e.g., eagles seen must be non-negative)
- The PostgreSQL standard implementation is minimalistic but misses data validation and audit capability

[Back to Table of Contents](#table-of-contents)

## Characters

Character string types are general-purpose types suitable for storing any combination of text, numbers, and symbols.

### char(n)

A fixed-length column where the character length is specified by `n`. Values shorter than `n` are padded with spaces to reach the fixed length. This type is part of standard SQL and is also known as `character(n)`, though it's used infrequently in modern applications.

### varchar(n)

A variable-length column with maximum length `n`. If values are shorter than the maximum, only the actual characters consume storage space, saving considerable space in large databases. This type is included in standard SQL and can also be specified as `character varying(n)`.

### text

A PostgreSQL-specific variable-length column with unlimited length (up to approximately 1 gigabyte). While not part of the SQL standard, similar implementations exist in other database systems like Microsoft SQL Server and MySQL.

### Character Types in Action

**Business Context:** Testing character data storage behavior to optimize storage efficiency in a text processing system

```sql
-- Domain predicate: Domain(CharacterTest) → CharacterTest ∈ DataAnalysis
CREATE TABLE "DataAnalysis"."CharacterTest" (
    "CharacterTestId" SERIAL, -- Unique test identifier
    "VarcharColumn" VARCHAR(10), -- Variable-length, max 10 characters
    "CharColumn" CHAR(10), -- Fixed-length, always 10 characters
    "TextColumn" TEXT, -- Unlimited length text
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ CharacterTest: (x.CharacterTestId = y.CharacterTestId) → (x = y)
    CONSTRAINT "PK_DataAnalysis_CharacterTest" PRIMARY KEY ("CharacterTestId"),
    
    -- Not null predicates:
    -- ∀x ∈ CharacterTest: x.VarcharColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_VarcharColumn" CHECK ("VarcharColumn" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.CharColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_CharColumn" CHECK ("CharColumn" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.TextColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_TextColumn" CHECK ("TextColumn" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ CharacterTest: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_CharacterTest_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ CharacterTest: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_DataAnalysis_CharacterTest_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ CharacterTest: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_DataAnalysis_CharacterTest_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ CharacterTest: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_DataAnalysis_CharacterTest_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ CharacterTest: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_DataAnalysis_CharacterTest_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);

-- Data insertion for testing character type behavior
INSERT INTO "DataAnalysis"."CharacterTest" 
    ("VarcharColumn", "CharColumn", "TextColumn")
VALUES
    ('abc', 'abc', 'abc'),
    ('defghi', 'defghi', 'defghi');

-- Export to examine space padding effect for CHAR type
COPY "DataAnalysis"."CharacterTest" 
    ("VarcharColumn", "CharColumn", "TextColumn")
TO 'C:\YourDirectory\typetest.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE char_data_types (
    varchar_column varchar(10),
    char_column char(10),
    text_column text
);

INSERT INTO char_data_types
VALUES
    ('abc', 'abc', 'abc'),
    ('defghi', 'defghi', 'defghi');

COPY char_data_types TO 'C:\YourDirectory\typetest.txt'
WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

### Implementation Analysis
- The exported file shows that `char` types pad spaces to maintain fixed length:
```
varchar_column|char_column|text_column
abc|abc       |abc
defghi|defghi    |defghi
```
- This demonstrates the storage efficiency differences between fixed and variable length types
- While PostgreSQL documentation indicates no substantial performance difference among the types, the space efficiency of `varchar` and `text` make them preferable for most use cases
- The DDDD implementation adds rigorous data validation and audit capabilities lacking in the standard approach

[Back to Table of Contents](#table-of-contents)

## Numbers

Number data types enable storage and mathematical operations on numeric values. Unlike numbers stored as character strings, proper numeric types allow calculations and sort in numerical order.

### Integers

Integer types store whole numbers (positive, negative, and zero) and come in three sizes with different storage requirements and value ranges:

| Type     | Storage | Range |
|----------|---------|-------|
| smallint | 2 bytes | -32,768 to +32,767 |
| integer  | 4 bytes | -2,147,483,648 to +2,147,483,647 |
| bigint   | 8 bytes | -9,223,372,036,854,775,808 to +9,223,372,036,854,775,807 |

**Business Context:** Product inventory management system with varying quantity scales

```sql
-- Domain predicate: Domain(Product) → Product ∈ Inventory
CREATE TABLE "Inventory"."Product" (
    "ProductId" BIGINT, -- Large product identifier range
    "StoreQuantity" INTEGER, -- Medium range inventory count
    "ShelfPosition" SMALLINT, -- Small range position identifier
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ Product: (x.ProductId = y.ProductId) → (x = y)
    CONSTRAINT "PK_Inventory_Product" PRIMARY KEY ("ProductId"),
    
    -- Not null predicates:
    -- ∀x ∈ Product: x.ProductId ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_ProductId" CHECK ("ProductId" IS NOT NULL),
    -- ∀x ∈ Product: x.StoreQuantity ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_StoreQuantity" CHECK ("StoreQuantity" IS NOT NULL),
    -- ∀x ∈ Product: x.ShelfPosition ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_ShelfPosition" CHECK ("ShelfPosition" IS NOT NULL),
    -- ∀x ∈ Product: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ Product: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ Product: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ Product: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Inventory_Product_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Data validation predicates:
    -- ∀x ∈ Product: x.StoreQuantity ≥ 0
    CONSTRAINT "CHK_Inventory_Product_NonNegativeQuantity" CHECK ("StoreQuantity" >= 0),
    -- ∀x ∈ Product: x.ShelfPosition > 0 AND x.ShelfPosition <= 100
    CONSTRAINT "CHK_Inventory_Product_ShelfPositionRange" CHECK ("ShelfPosition" > 0 AND "ShelfPosition" <= 100),
    
    -- Default constraint predicates:
    -- ∀x ∈ Product: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Inventory_Product_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ Product: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Inventory_Product_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ Product: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Inventory_Product_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ Product: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Inventory_Product_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE inventory.product (
    product_id bigint,
    store_quantity integer,
    shelf_position smallint
);
```

### Auto-Incrementing Integers

PostgreSQL provides special serial types for creating auto-incrementing identity columns, commonly used for primary keys:

| Type | Storage | Range |
|------|---------|-------|
| smallserial | 2 bytes | 1 to 32,767 |
| serial | 4 bytes | 1 to 2,147,483,647 |
| bigserial | 8 bytes | 1 to 9,223,372,036,854,775,807 |

**Business Context:** Person identification system with automatic ID assignment

```sql
-- Domain predicate: Domain(Person) → Person ∈ Contact
CREATE TABLE "Contact"."Person" (
    "PersonId" SERIAL, -- Auto-incrementing identifier
    "PersonName" VARCHAR(100), -- Person's full name
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ Person: (x.PersonId = y.PersonId) → (x = y)
    CONSTRAINT "PK_Contact_Person" PRIMARY KEY ("PersonId"),
    
    -- Not null predicates:
    -- ∀x ∈ Person: x.PersonName ≠ NULL
    CONSTRAINT "CHK_Contact_Person_PersonName" CHECK ("PersonName" IS NOT NULL),
    -- ∀x ∈ Person: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Contact_Person_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ Person: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Contact_Person_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ Person: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Contact_Person_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ Person: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Contact_Person_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ Person: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Contact_Person_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ Person: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Contact_Person_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ Person: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Contact_Person_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ Person: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Contact_Person_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE people (
    id serial,
    person_name varchar(100)
);
```

### Decimal Numbers

PostgreSQL offers two approaches for storing decimal numbers: fixed-point and floating-point types.

### Fixed-Point Numbers

Fixed-point numbers (also called arbitrary precision types) use the `numeric(precision,scale)` or `decimal(precision,scale)` format:
- `precision`: Total number of digits on both sides of the decimal point
- `scale`: Number of digits to the right of the decimal point

**Business Context:** Financial transaction system requiring exact decimal precision for monetary amounts

```sql
-- Domain predicate: Domain(Transaction) → Transaction ∈ Finance
CREATE TABLE "Finance"."Transaction" (
    "TransactionId" SERIAL, -- Unique identifier for transactions
    "Amount" NUMERIC(10,2), -- Dollar amount with 2 decimal places
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ Transaction: (x.TransactionId = y.TransactionId) → (x = y)
    CONSTRAINT "PK_Finance_Transaction" PRIMARY KEY ("TransactionId"),
    
    -- Not null predicates:
    -- ∀x ∈ Transaction: x.Amount ≠ NULL
    CONSTRAINT "CHK_Finance_Transaction_Amount" CHECK ("Amount" IS NOT NULL),
    -- ∀x ∈ Transaction: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Finance_Transaction_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ Transaction: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Finance_Transaction_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ Transaction: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Finance_Transaction_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ Transaction: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Finance_Transaction_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ Transaction: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Finance_Transaction_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ Transaction: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Finance_Transaction_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ Transaction: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Finance_Transaction_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ Transaction: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Finance_Transaction_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE finance.transaction (
    transaction_id serial,
    amount numeric(10,2)  -- Can store up to 8 digits before decimal, 2 after
);
```

### Floating-Point Types

PostgreSQL offers two floating-point types with different precision levels:

| Type | Storage | Precision |
|------|---------|-----------|
| real | 4 bytes | 6 decimal digits |
| double precision | 8 bytes | 15 decimal digits |

**Business Context:** Scientific measurement system tracking different precision levels

```sql
-- Domain predicate: Domain(Measurement) → Measurement ∈ Science
CREATE TABLE "Science"."Measurement" (
    "MeasurementId" SERIAL, -- Unique identifier for measurements
    "Temperature" REAL, -- Standard precision temperature recording
    "PreciseCalculation" DOUBLE PRECISION, -- High precision scientific calculation
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ Measurement: (x.MeasurementId = y.MeasurementId) → (x = y)
    CONSTRAINT "PK_Science_Measurement" PRIMARY KEY ("MeasurementId"),
    
    -- Not null predicates:
    -- ∀x ∈ Measurement: x.Temperature ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_Temperature" CHECK ("Temperature" IS NOT NULL),
    -- ∀x ∈ Measurement: x.PreciseCalculation ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_PreciseCalculation" CHECK ("PreciseCalculation" IS NOT NULL),
    -- ∀x ∈ Measurement: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ Measurement: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ Measurement: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ Measurement: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Science_Measurement_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ Measurement: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Science_Measurement_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ Measurement: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Science_Measurement_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ Measurement: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Science_Measurement_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ Measurement: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Science_Measurement_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE science.measurement (
    point_id serial,
    temperature real,
    precise_calculation double precision
);
```

### Number Data Types in Action

**Business Context:** Testing behavior of different numeric types for storage and precision requirements

```sql
-- Domain predicate: Domain(NumberTest) → NumberTest ∈ DataAnalysis
CREATE TABLE "DataAnalysis"."NumberTest" (
    "NumberTestId" SERIAL, -- Unique test identifier
    "NumericColumn" NUMERIC(20,5), -- Fixed precision with 5 decimal places
    "RealColumn" REAL, -- Standard floating-point (6 digits precision)
    "DoubleColumn" DOUBLE PRECISION, -- High precision floating-point (15 digits)
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ NumberTest: (x.NumberTestId = y.NumberTestId) → (x = y)
    CONSTRAINT "PK_DataAnalysis_NumberTest" PRIMARY KEY ("NumberTestId"),
    
    -- Not null predicates:
    -- ∀x ∈ NumberTest: x.NumericColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_NumericColumn" CHECK ("NumericColumn" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.RealColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_RealColumn" CHECK ("RealColumn" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.DoubleColumn ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_DoubleColumn" CHECK ("DoubleColumn" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ NumberTest: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_DataAnalysis_NumberTest_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ NumberTest: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_DataAnalysis_NumberTest_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ NumberTest: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_DataAnalysis_NumberTest_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ NumberTest: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_DataAnalysis_NumberTest_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ NumberTest: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_DataAnalysis_NumberTest_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);

# Chapter 3: UNDERSTANDING DATA TYPES (continued)

```sql
-- Insert test data to demonstrate precision
INSERT INTO "DataAnalysis"."NumberTest" 
    ("NumericColumn", "RealColumn", "DoubleColumn")
VALUES
    (.7, .7, .7),
    (2.13579, 2.13579, 2.13579),
    (2.1357987654, 2.1357987654, 2.1357987654);

-- Query to show precision differences
SELECT 
    "NumericColumn", 
    "RealColumn", 
    "DoubleColumn" 
FROM "DataAnalysis"."NumberTest";
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE number_data_types (
    numeric_column numeric(20,5),
    real_column real,
    double_column double precision
);

INSERT INTO number_data_types
VALUES
    (.7, .7, .7),
    (2.13579, 2.13579, 2.13579),
    (2.1357987654, 2.1357987654, 2.1357987654);

SELECT * FROM number_data_types;
```

#### Query Results
```
numeric_column | real_column | double_column
---------------+-------------+---------------
0.70000        | 0.7         | 0.7
2.13579        | 2.13579     | 2.13579
2.13580        | 2.1358      | 2.1357987654
```

### Implementation Analysis

- The DDDD implementation adds rigorous validation through explicit constraints
- Result differences between numeric types are evident:
  - `numeric` always stores exactly 5 decimal places as defined (padding with zeros if needed)
  - `real` provides approximately 6 digits of precision, rounding or truncating longer values
  - `double precision` can store the full precision of the values in this example
- These differences have implications for calculation accuracy and storage requirements

### Trouble with Floating-Point Math

**Business Context:** Financial calculations requiring exact precision to avoid rounding errors

```sql
-- Domain predicate: Domain(PrecisionTest) → PrecisionTest ∈ Finance
-- Query to demonstrate floating-point calculation issues
SELECT
    "NumericColumn" * 10000000 AS "Fixed",
    "RealColumn" * 10000000 AS "Float"
FROM "DataAnalysis"."NumberTest"
WHERE "NumericColumn" = .7;
```

### PostgreSQL Standard Implementation

```sql
SELECT
    numeric_column * 10000000 AS "Fixed",
    real_column * 10000000 AS "Float"
FROM number_data_types
WHERE numeric_column = .7;
```

#### Query Results

```
Fixed           | Float
----------------+------------------
7000000.00000   | 6999999.88079071
```

### Implementation Analysis

- This demonstrates a critical limitation of floating-point types: inexact representation
- The fixed-point calculation produces the mathematically correct result (7,000,000)
- The floating-point calculation shows a small but significant error
- This error occurs because computers cannot precisely represent some decimal values in binary floating-point format
- These small errors can compound in complex calculations, making floating-point types unsuitable for financial applications or other cases requiring exact precision

### Choosing Your Number Data Type

Based on the examples above, here are key guidelines for selecting appropriate number types:

1. **Use integers when possible**
   - If data doesn't require decimals, integer types provide the most efficient storage and exact calculations
   - Select the appropriate integer size based on the expected range of values

2. **For decimal data requiring exact calculations:**
   - Choose `numeric` or `decimal` for financial calculations, scientific measurements requiring exact precision, or any application where calculation exactness is critical
   - Be aware that these types consume more storage space than floating-point alternatives

3. **For decimal data where approximation is acceptable:**
   - Use `real` or `double precision` when speed and storage efficiency are more important than absolute precision
   - Appropriate for scientific applications where minor rounding is acceptable
   - Always consider the potential impact of floating-point errors when designing calculations

4. **Choose an appropriate size:**
   - Select a type that can accommodate your maximum expected values
   - For numeric/decimal types, specify precision large enough for digits on both sides of the decimal point

[Back to Table of Contents](#table-of-contents)

## Dates and Times

PostgreSQL provides robust support for temporal data with specialized types for dates, times, and intervals.

### Date and Time Types

| Type      | Storage | Description   | Range                 |
|-----------|---------|---------------|-----------------------|
| timestamp | 8 bytes | Date and time | 4713 BC to 294276 AD  |
| date      | 4 bytes | Date only     | 4713 BC to 5874897 AD |
| time      | 8 bytes | Time only     | 00:00:00 to 24:00:00  |
| interval  | 16 bytes| Time period   | +/- 178,000,000 years |

The `timestamp` type can include time zone information when specified with `with time zone` (or shorthand `timestamptz`), which is critical for comparing times recorded in different geographical locations.

**Business Context:** Event scheduling system tracking timestamps across different time zones

```sql
-- Domain predicate: Domain(TimeTest) → TimeTest ∈ Event
CREATE TABLE "Event"."TimeTest" (
    "TimeTestId" SERIAL, -- Unique test identifier
    "TimestampColumn" TIMESTAMP WITH TIME ZONE, -- Date and time with time zone
    "IntervalColumn" INTERVAL, -- Time interval
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ TimeTest: (x.TimeTestId = y.TimeTestId) → (x = y)
    CONSTRAINT "PK_Event_TimeTest" PRIMARY KEY ("TimeTestId"),
    
    -- Not null predicates:
    -- ∀x ∈ TimeTest: x.TimestampColumn ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_TimestampColumn" CHECK ("TimestampColumn" IS NOT NULL),
    -- ∀x ∈ TimeTest: x.IntervalColumn ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_IntervalColumn" CHECK ("IntervalColumn" IS NOT NULL),
    -- ∀x ∈ TimeTest: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ TimeTest: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ TimeTest: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ TimeTest: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Event_TimeTest_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ TimeTest: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Event_TimeTest_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ TimeTest: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Event_TimeTest_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ TimeTest: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Event_TimeTest_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ TimeTest: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Event_TimeTest_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);

-- Insert test data with different time zone formats
INSERT INTO "Event"."TimeTest"
    ("TimestampColumn", "IntervalColumn")
VALUES
    ('2018-12-31, 01:00 EST', '2 days'),
    ('2018-12-31, 01:00 -8', '1 month'),
    ('2018-12-31 01:00 Australia/Melbourne', '1 century'),
    (NOW(), '1 week');
```

### PostgreSQL Standard Implementation

```sql
CREATE TABLE date_time_types (
    timestamp_column timestamp with time zone,
    interval_column interval
);

INSERT INTO date_time_types
VALUES
    ('2018-12-31 01:00 EST', '2 days'),
    ('2018-12-31 01:00 -8', '1 month'),
    ('2018-12-31 01:00 Australia/Melbourne', '1 century'),
    (NOW(), '1 week');

SELECT * FROM date_time_types;
```

#### Sample Query Results

```makdown
timestamp_column               | interval_column
-------------------------------+----------------
2018-12-31 01:00:00-05         | 2 days
2018-12-31 04:00:00-05         | 1 mon
2018-12-30 09:00:00-05         | 100 years
2023-06-15 14:32:11.437812-04  | 7 days
```

### Implementation Analysis

- The DDDD implementation provides explicit constraints and metadata tracking
- Times are automatically converted to the local time zone of the database client
- Three different methods for specifying time zones are demonstrated:
  - Named time zone abbreviation: 'EST' (Eastern Standard Time)
  - UTC offset: '-8' (8 hours behind UTC, Pacific time)
  - Area/location format: 'Australia/Melbourne'
- PostgreSQL standardizes interval display formats (e.g., '1 century' becomes '100 years')
- The `now()` function captures the current transaction time

### Using the interval Data Type in Calculations

**Business Context:** Event planning system calculating relative dates for scheduling

```sql
-- Domain predicate: Domain(TimeCalculation) → TimeCalculation ∈ Event
-- Query to demonstrate interval calculations with timestamps
SELECT
    "TimestampColumn",
    "IntervalColumn",
    "TimestampColumn" - "IntervalColumn" AS "NewDate"
FROM "Event"."TimeTest";
```

### PostgreSQL Standard Implementation

```sql
SELECT
    timestamp_column,
    interval_column,
    timestamp_column - interval_column AS new_date
FROM date_time_types;
```

#### Sample Query Results

```markdown
timestamp_column              | interval_column | new_date
------------------------------+-----------------+------------------------------
2018-12-31 01:00:00-05        | 2 days          | 2018-12-29 01:00:00-05
2018-12-31 04:00:00-05        | 1 mon           | 2018-11-30 04:00:00-05
2018-12-30 09:00:00-05        | 100 years       | 1918-12-30 09:00:00-05
2023-06-15 14:32:11.437812-04 | 7 days          | 2023-06-08 14:32:11.437812-04
```

### Implementation Analysis

- The `interval` type enables intuitive date arithmetic by subtracting time periods from timestamps
- This example calculates dates in the past by subtracting intervals
- The result maintains the appropriate time zone from the original timestamp
- This functionality simplifies common business logic involving scheduling, deadlines, or historical analysis
- The formal predicate logic for this operation can be expressed as:
  `∀x ∈ TimeTest: HasNewDate(x, x.TimestampColumn - x.IntervalColumn)`

[Back to Table of Contents](#table-of-contents)

## Miscellaneous Types

PostgreSQL supports many additional specialized data types beyond character, number, and date/time categories, including:

- **Boolean**: Stores `true` or `false` values
- **Geometric types**: For points, lines, circles, and other two-dimensional objects
- **Network address types**: For IP or MAC addresses
- **UUID**: Universal Unique Identifier type for globally unique values
- **XML and JSON**: For storing structured data in these formats

These specialized types enable more precise domain modeling and optimization for specific data storage needs.

[Back to Table of Contents](#table-of-contents)

## Transforming Values with CAST

The `CAST()` function converts values between compatible data types, enabling flexible data manipulation and display.

**Business Context:** Data integration system requiring type conversions for reporting

```sql
-- Domain predicate: Domain(Conversion) → Conversion ∈ DataIntegration
-- Timestamp to character conversion
SELECT 
    "TimestampColumn", 
    CAST("TimestampColumn" AS VARCHAR(10)) AS "DateString"
FROM "Event"."TimeTest";

-- Numeric conversion examples
SELECT 
    "NumericColumn",
    CAST("NumericColumn" AS INTEGER) AS "RoundedInteger",
    CAST("NumericColumn" AS VARCHAR(6)) AS "NumericString"
FROM "DataAnalysis"."NumberTest";

-- This will fail - letters cannot be converted to numbers
SELECT CAST("CharColumn" AS INTEGER) FROM "DataAnalysis"."CharacterTest";
```

### PostgreSQL Standard Implementation

```sql
-- Convert timestamp to varchar
SELECT 
    timestamp_column, 
    CAST(timestamp_column AS varchar(10))
FROM date_time_types;

-- Convert numeric values to integer and varchar
SELECT 
    numeric_column,
    CAST(numeric_column AS integer),
    CAST(numeric_column AS varchar(6))
FROM number_data_types;

-- This fail with an error
SELECT CAST(char_column AS integer) FROM char_data_types;
```

#### Sample Query Results

First query:

```markdown
timestamp_column              | datestring
------------------------------+------------
2018-12-31 01:00:00-05        | 2018-12-31
```

Second query:

```markdown
numeric_column | roundedinteger | numericstring
---------------+----------------+--------------
0.70000        | 1              | 0.7000
2.13579        | 2              | 2.1357
2.13580        | 2              | 2.1358
```

Third query:

```markdown
ERROR:  invalid input syntax for type integer: "abc       "
```

### PostgreSQL Alternative Shorthand

PostgreSQL also offers a less-verbose "double colon" notation as a shortcut for `CAST()`:

```sql
-- Double colon shorthand for casting
SELECT timestamp_column::varchar(10) FROM date_time_types;
```

### Implementation Analysis

- The CAST function enables conversion between compatible types
- When casting to a type with less precision, data may be truncated:
  - CAST to INTEGER rounds decimal values
  - CAST to VARCHAR(n) truncates at n characters
- Incompatible conversions (like text with letters to numbers) fail with an error
- Type conversion is essential for:
  - Reporting and data presentation
  - Combining values of different types
  - Data migration between systems
- The predicate logic for type conversion can be expressed as:
  `∀x ∈ Source: HasConvertedValue(x, ConvertType(x.Value, TargetType))`

[Back to Table of Contents](#table-of-contents)

## Try It Yourself Exercises

The chapter concludes with practical exercises to reinforce understanding of data types:

### Exercise 1: Selecting an appropriate data type for tracking driver mileage

**Business Context:** Delivery company tracking driver mileage to a tenth of a mile with a maximum daily distance of 999 miles

```sql
-- Domain predicate: Domain(DriverMileage) → DriverMileage ∈ Logistics
CREATE TABLE "Logistics"."DriverMileage" (
    "DriverMileageId" SERIAL,
    "DriverId" INTEGER,
    "TravelDate" DATE,
    "MilesDriven" NUMERIC(5,1), -- Up to 999.9 miles with 0.1 precision
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ DriverMileage: (x.DriverMileageId = y.DriverMileageId) → (x = y)
    CONSTRAINT "PK_Logistics_DriverMileage" PRIMARY KEY ("DriverMileageId"),
    
    -- Not null predicates:
    -- ∀x ∈ DriverMileage: x.DriverId ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_DriverId" CHECK ("DriverId" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.TravelDate ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_TravelDate" CHECK ("TravelDate" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.MilesDriven ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_MilesDriven" CHECK ("MilesDriven" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ DriverMileage: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Logistics_DriverMileage_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Data validation predicates:
    -- ∀x ∈ DriverMileage: x.MilesDriven ≥ 0
    CONSTRAINT "CHK_Logistics_DriverMileage_NonNegative" CHECK ("MilesDriven" >= 0),
    -- ∀x ∈ DriverMileage: x.MilesDriven <= 999.9
    CONSTRAINT "CHK_Logistics_DriverMileage_MaximumDistance" CHECK ("MilesDriven" <= 999.9),
    
    -- Default constraint predicates:
    -- ∀x ∈ DriverMileage: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Logistics_DriverMileage_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ DriverMileage: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Logistics_DriverMileage_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ DriverMileage: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Logistics_DriverMileage_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ DriverMileage: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Logistics_DriverMileage_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

**Answer Analysis:**

- `NUMERIC(5,1)` is the most appropriate type for this scenario because:
  - It allows tracking to a tenth of a mile as required (one decimal place)
  - Can store values up to 999.9, meeting the maximum daily travel requirement
  - Provides exact decimal arithmetic, ensuring accurate mileage calculations
  - Fixed-point representation avoids floating-point imprecision issues

### Exercise 2: Appropriate data types for driver names

**Business Context:** Driver management system with proper name handling requirements

```sql
-- Domain predicate: Domain(Driver) → Driver ∈ Logistics
CREATE TABLE "Logistics"."Driver" (
    "DriverId" SERIAL,
    "FirstName" VARCHAR(50),
    "LastName" VARCHAR(50),
    
    -- Standard metadata fields for data quality and audit trail
    "IsDataMissing" BOOLEAN,
    "UserAuthorizationId" INTEGER,
    "DateAdded" TIMESTAMPTZ,
    "DateOfLastUpdate" TIMESTAMPTZ,
    
    -- Primary key predicate: ∀x,y ∈ Driver: (x.DriverId = y.DriverId) → (x = y)
    CONSTRAINT "PK_Logistics_Driver" PRIMARY KEY ("DriverId"),
    
    -- Not null predicates:
    -- ∀x ∈ Driver: x.FirstName ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_FirstName" CHECK ("FirstName" IS NOT NULL),
    -- ∀x ∈ Driver: x.LastName ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_LastName" CHECK ("LastName" IS NOT NULL),
    -- ∀x ∈ Driver: x.IsDataMissing ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_IsDataMissing" CHECK ("IsDataMissing" IS NOT NULL),
    -- ∀x ∈ Driver: x.UserAuthorizationId ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_UserAuthorizationId" CHECK ("UserAuthorizationId" IS NOT NULL),
    -- ∀x ∈ Driver: x.DateAdded ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_DateAdded" CHECK ("DateAdded" IS NOT NULL),
    -- ∀x ∈ Driver: x.DateOfLastUpdate ≠ NULL
    CONSTRAINT "CHK_Logistics_Driver_DateOfLastUpdate" CHECK ("DateOfLastUpdate" IS NOT NULL),
    
    -- Default constraint predicates:
    -- ∀x ∈ Driver: (x.IsDataMissing = NULL) → (x.IsDataMissing := FALSE)
    CONSTRAINT "DF_Logistics_Driver_IsDataMissing" DEFAULT FALSE FOR "IsDataMissing",
    -- ∀x ∈ Driver: (x.UserAuthorizationId = NULL) → (x.UserAuthorizationId := 1)
    CONSTRAINT "DF_Logistics_Driver_UserAuthorizationId" DEFAULT 1 FOR "UserAuthorizationId",
    -- ∀x ∈ Driver: (x.DateAdded = NULL) → (x.DateAdded := NOW())
    CONSTRAINT "DF_Logistics_Driver_DateAdded" DEFAULT NOW() FOR "DateAdded",
    -- ∀x ∈ Driver: (x.DateOfLastUpdate = NULL) → (x.DateOfLastUpdate := NOW())
    CONSTRAINT "DF_Logistics_Driver_DateOfLastUpdate" DEFAULT NOW() FOR "DateOfLastUpdate"
);
```

**Answer Analysis:**

- `VARCHAR` is appropriate for name fields because:
  - Names vary in length, making variable-length storage more efficient
  - 50 characters provides ample space for most names while limiting excessive storage
- Separating first and last names provides several advantages:
  - Enables sorting by last name in reports and queries
  - Supports targeted searches on either name component
  - Allows appropriate formatting for different outputs (formal vs. casual)
  - Facilitates compliance with cultural naming conventions
  - Supports more granular data validation and business rules

### Exercise 3: Converting malformatted date strings

**Business Context:** Data import process with potentially malformatted date strings

```sql
-- Testing conversion of malformatted date string
SELECT CAST('4//2017' AS TIMESTAMP);
```

**Answer Analysis:**

- Attempting to convert '4//2017' to a timestamp will fail with an error:
  - PostgreSQL will report "invalid input syntax for type timestamp"
  - The string doesn't conform to any recognized date format
  - The double slash is not a valid date separator
  - The string lacks a day value
- To successfully convert date strings, they must conform to recognizable formats, such as:
  - ISO format: '2017-04-01'
  - With explicit day: '04/01/2017'
  - With month name: 'April 1, 2017'

## Summary

Chapter 3 explores the fundamental data types available in PostgreSQL, providing guidance for making appropriate type selections based on business requirements and data characteristics.

Key takeaways include:

1. **Character Types:**
   - `char(n)`: Fixed-length with space padding
   - `varchar(n)`: Variable-length with maximum size
   - `text`: Unlimited length for large text

2. **Number Types:**
   - Integer types (`smallint`, `integer`, `bigint`) for whole numbers
   - Fixed-point types (`numeric`, `decimal`) for exact decimal calculations
   - Floating-point types (`real`, `double precision`) for approximate decimal values

3. **Date and Time Types:**
   - `timestamp` for date and time values, preferably with time zone
   - `date` for date-only values
   - `time` for time-only values
   - `interval` for time periods used in date arithmetic

4. **Type Selection Considerations:**
   - Storage efficiency vs precision requirements
   - Mathematical operation requirements
   - Range limitations
   - Compatibility with application needs
   - Data validation and business rule enforcement

5. **Type Conversion:**
   - The `CAST()` function converts between compatible types
   - Type conversion may result in precision loss
   - Incompatible conversions will fail

By understanding data types and applying Domain Driven Database Design principles with formal predicate logic, you can create robust databases that accurately model business domains while enforcing data integrity through explicit constraints.

[Back to Table of Contents](#table-of-contents)