Create a reusable prompt for future use on other chapters
Make sure that each chapter follows the convention  Chapter #  Chapter Title  Best Practices ( PostgreSQL 17)
Please create snake case and PascalCase code in all examples in the Markdown Language. even single word are to be represented in PascalCase for SchemaNames, TableNames, and ColumnNames Note instead of pkey and fkey use pky and fky. Always Choose meaningful names: Avoid cryptic abbreviations Use singular nouns for table names since it represents table definition of metadata for each instance of the rows that are being stored. PascalCase requires double quotes to preserve case

# Create a comprehensive Markdown guide for Chapter [NUMBER] - [CHAPTER TITLE] Best Practices (PostgreSQL 17).  Build upon the existing template to enhance it with better rules for creating new mark down language
# Create a comprehensive Markdown guide for Chapter [NUMBER] - [CHAPTER TITLE] Best Practices (PostgreSQL 17)

Please produce a step-by-step guide with detailed explanations in a Markdown document, including the following elements:
Note: split the Markdown into two major topical sections (PostgreSQL Standard Conventions(snake_case)
and Domain Driven Database Design Standard Conventions(PascalCase))provide grid of all of the unique objects 
and their generalized format.

1. Create a well-structured table of contents with clickable anchors
2. Include clear section headings with appropriate emoji icons (📚, 🏷️, 🔒, etc.)
3. For ALL code examples, provide BOTH PostgreSQL Standard (snake_case)  and Domain Driven Database Design(PascalCase)
   versions:
   - snake_case should use all lowercase with underscores
   - PascalCase should use initial capitals and be enclosed in double quotes to preserve case
   - For constraint naming conventions, use pky instead of pkey and fky instead of fkey
   - Use singular nouns for table names (representing metadata definition)
   - Use fully qualified names with schema prefixes

4. For each section:
   - Provide conceptual explanation with real-world analogies where appropriate
   - Include practical code examples in both naming conventions
   - Emphasize PostgreSQL 17 specific features and improvements
   - Highlight best practices and common pitfalls to avoid

5. Include a comprehensive "Best Practices Summary" section with bullet points
6. End with hands-on exercises that demonstrate the chapter concepts

Important formatting notes:
- Use "Back to TOC" navigation links at the end of each major section
- Ensure proper syntax highlighting for SQL code blocks
- Use meaningful names throughout all examples (avoid cryptic abbreviations)
- Emphasize PostgreSQL 17-specific features (GENERATED ALWAYS AS IDENTITY, schema organization, etc.)
- Format the title as: "Chapter [NUMBER] [CHAPTER TITLE] Best Practices (PostgreSQL 17)"
# Below make have some overlap but needs to be used to enhance the comprehensive Markdown guide for Chapter [NUMBER] - [CHAPTER TITLE] Best Practices (PostgreSQL 17).  Build upon the existing template to enhance it with better rules for creating new mark down language

1. Create two  use cases for PostgreSQL in snake case called "PostgreSQL Standard Conventions" and Microsoft T-SQL in PascalCase called "Enterprise heterogenous Database Standard Conventions"  for same PostgreSQL 17 code. 
2. Microsoft T-SQL 2022 naming conventions for all of their indexes, defaults, check constraints for columns, table, and database level which  multiple cross table object validation, and must be delimited to preserve the "PascalCase"
3.  All Create Domains\ Type must  be singular name with a SQL datatype avoid and eliminate any complex Domains\Types.  Just like the SOLID Design Principle SRP (Single Responsibility Principle) 

In both PostgreSQL and Microsoft T-SQL, naming conventions for indexes, defaults, check constraints, and other database objects generally follow these principles: **using descriptive names, adhering to SQL standards (letters, numbers, underscores), and avoiding reserved keywords**. PostgreSQL tends to favor lowercase names and underscores for spaces, while T-SQL allows uppercase and lowercase, and often uses underscores for spaces. 
** "PostgreSQL Standard Conventions" :**
* **Table and Column Names:**
Lowercase letters, numbers, and underscores. First character must be a lowercase letter. 
* **Index Names:**
`{schemaname}_{tablename}_{columnname}_{suffix}`, where the suffix can be:
   * `pkey` for Primary Key. 
   * `key` for Unique. 
   * `excl` for Exclusion. 
   * `idx` for any other index. 
   * `fkey` for Foreign Key. 
   * `check` for Check constraint. 
* **Default Constraints:**
The `DEFAULT` keyword is used, and a name can be assigned to the constraint. Example: `CONSTRAINT {schemaname}_{tablename}_positive_price CHECK (price > 0)`. 
* **Check Constraints:**
`CHECK` keyword followed by a constraint expression. 
* **Database Objects:**
All database objects (like tables, views, functions) follow the same naming rules as table names. 
**"Enterprise heterogenous Database Standard Conventions":**
* **Table and Column Names:** Follows SQL standards for identifiers (letters, numbers, underscores). Uppercase or lowercase allowed. 
* **Index Names:** `IX_SchemaName_TabbleName_ColumnName` or similar. 
* **Default Constraints:** `CONSTRAINT DF_SchemaName_TabbleName_ColumnName` DEFAULT <value>`. 
* **Check Column Constraints:** `CONSTRAINT CHK_SchemaName_TabbleName_ColumnName CHECK (condition)`. 
* **Check Single Table Constraints:** `CONSTRAINT CHK_SchemaName_TabbleName_TableValidationName CHECK (condition)`. 
* **Check Multi-Table Constraints:** A multi-table constraint in T-SQL, such as a CHECK constraint, can be implemented using a table-valued function (TVF) to verify conditions across multiple tables. The TVF should return a single value that indicates whether the 
constraint is met, and the CHECK constraint can then reference this TVF. 

"Enterprise heterogenous Database Standard Conventions", PascalCase which starts with a first character must be a UpperCase letter. 
* **Index Names:**
`{PostFix}_{SchemaName}_{TabbleName}_{ColumnName}`, where the suffix can be:
   * `PK_SchemaName_TabbleName` for Primary Key. 
   * `UK_SchemaName_TabbleName` for Unique. 
   * `excl` for Exclusion is a `IX_SchemaName_TabbleName_ColumnName` for any other index. with an addition component with an INCLUDE (Email, Address) is optional, but it can be beneficial to include other columns that are frequently accessed in queries using the index to avoid table scans. 
   * `IX_SchemaName_TabbleName_ColumnName` for any other index. 
   * `FK_ChildSchemaName_TabbleName_ParentSchemaName_TabbleName_ColumnName` for Foreign Key. 


* **Database Objects:** Names must be unique within a database, but not necessarily across databases. Validate that these conventions which have some extensions will always produce unique objects without a numeric post-fix to 
**Key Differences and Considerations:**
* **Case Sensitivity:** "PostgreSQL Standard Conventions" is case-insensitive for unquoted identifiers, but case-sensitive for quoted identifiers. T-SQL is generally case-insensitive. 
* **Reserved Words:** Avoid using reserved words for object names in both systems, but reserved words can be used delimited to notify it is used not as a reserve word. Example: Sales."Order" or sales."order"
* **Naming Conventions:** Consistency is key. Establish a convention for your project and follow it consistently. 
* **Column Constraints vs. Table Constraints:** Both systems allow for column constraints (directly on the column definition) and table constraints (defined separately). Table constraints can reference multiple columns. 
* **Unique Constraints:** Both systems have unique constraints, which enforce that values in a column or set of columns are unique. A unique constraint automatically creates an index to enforce the constraint. 
* **Database Level Objects:** Table and column names are unique within the database schema.