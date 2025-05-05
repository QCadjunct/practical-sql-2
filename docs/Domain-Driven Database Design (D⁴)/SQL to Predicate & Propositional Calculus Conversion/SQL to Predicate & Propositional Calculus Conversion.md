# SQL to Predicate & Propositional Calculus Conversion

```sql
-- Business context: Customer information storage and management within the Sales domain
-- Domain predicate: Domain(Customer) → Customer ∈ Sales
CREATE TABLE "Sales"."Customer" (
    -- Column predicate: CustomerId: Customer → BIGINT
    -- Auto-generation predicate: ∀c ∈ Customer: CustomerId(c) is auto-generated
    -- Primary Key predicate: ∀x,y ∈ Customer: (x.CustomerId = y.CustomerId) → (x = y)
    -- Existence predicate: ∀x ∈ Customer: ∃!id (id = x.CustomerId)
    "CustomerId" BIGINT GENERATED ALWAYS AS IDENTITY CONSTRAINT "PK_Sales_Customer" PRIMARY KEY,
    
    -- Column predicate: FirstName: Customer → VARCHAR(50)
    -- Non-null predicate: ∀c ∈ Customer: FirstName(c) ≠ NULL
    -- Check constraint predicate: ∀c ∈ Customer: FirstName(c) ≠ NULL
    "FirstName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_Sales_Customer_FirstName" CHECK ("FirstName" IS NOT NULL),
    
    -- Column predicate: LastName: Customer → VARCHAR(50)
    -- Non-null predicate: ∀c ∈ Customer: LastName(c) ≠ NULL
    -- Check constraint predicate: ∀c ∈ Customer: LastName(c) ≠ NULL
    "LastName" VARCHAR(50) NOT NULL CONSTRAINT "CHK_Sales_Customer_LastName" CHECK ("LastName" IS NOT NULL),
    
    -- Column predicate: Email: Customer → VARCHAR(100)
    -- Unique constraint predicate: ∀c₁,c₂ ∈ Customer: (c₁ ≠ c₂) → (Email(c₁) ≠ Email(c₂) ∨ Email(c₁) = NULL ∨ Email(c₂) = NULL)
    "Email" VARCHAR(100) UNIQUE CONSTRAINT "UQ_Sales_Customer_Email",
    
    -- Column predicate: IsDataMissing: Customer → BOOLEAN
    -- Default value predicate: ∀c ∈ Customer: IsDataMissing(c) = NULL → IsDataMissing(c) := FALSE
    "IsDataMissing" BOOLEAN DEFAULT FALSE CONSTRAINT "DF_Sales_Customer_IsDataMissing",
    
    -- Column predicate: UserAuthorizationId: Customer → INTEGER
    -- Default value predicate: ∀c ∈ Customer: UserAuthorizationId(c) = NULL → UserAuthorizationId(c) := 1
    "UserAuthorizationId" INTEGER DEFAULT 1 CONSTRAINT "DF_Sales_Customer_UserAuthorizationId",
    
    -- Column predicate: DateAdded: Customer → TIMESTAMPTZ
    -- Default value predicate: ∀c ∈ Customer: DateAdded(c) = NULL → DateAdded(c) := NOW()
    "DateAdded" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Sales_Customer_DateAdded",
    
    -- Column predicate: DateOfLastUpdate: Customer → TIMESTAMPTZ
    -- Default value predicate: ∀c ∈ Customer: DateOfLastUpdate(c) = NULL → DateOfLastUpdate(c) := NOW()
    "DateOfLastUpdate" TIMESTAMPTZ DEFAULT NOW() CONSTRAINT "DF_Sales_Customer_DateOfLastUpdate"
    
    -- Complete entity structure predicate:
    -- ∀c ∈ Customer: 
    --   CustomerId(c) ∈ BIGINT ∧
    --   FirstName(c) ∈ VARCHAR(50) ∧ FirstName(c) ≠ NULL ∧
    --   LastName(c) ∈ VARCHAR(50) ∧ LastName(c) ≠ NULL ∧
    --   Email(c) ∈ VARCHAR(100) ∧
    --   IsDataMissing(c) ∈ BOOLEAN ∧
    --   UserAuthorizationId(c) ∈ INTEGER ∧
    --   DateAdded(c) ∈ TIMESTAMPTZ ∧
    --   DateOfLastUpdate(c) ∈ TIMESTAMPTZ
);
```

This conversion expresses the SQL table definition in terms of predicate and propositional calculus using:

1. Domain predicates to establish the relation between the Customer entity and the Sales schema
2. Column typing predicates that map each attribute to its appropriate data type
3. Constraint predicates expressing:
   - Auto-generation of the CustomerId
   - Primary key uniqueness and existence properties
   - Non-null requirements for FirstName and LastName
   - Uniqueness requirement for Email
   - Default value assignments for multiple fields
4. A comprehensive entity structure predicate that combines all attributes and constraints

The logical formulation provides a precise and unambiguous mathematical representation of the database schema, highlighting the constraints and relationships between entities in the Customer table.