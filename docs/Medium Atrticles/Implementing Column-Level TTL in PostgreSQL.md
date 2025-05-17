# Implementing Column-Level TTL in PostgreSQL

## Table of Contents

- [Introduction](#introduction)
- [Core Concept](#core-concept)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [1. Schema Design](#1-schema-design)
  - [2. Create the Partitioned Table](#2-create-the-partitioned-table)
  - [3. Set Up pg_partman](#3-set-up-pg_partman)
  - [4. Create the Combined View](#4-create-the-combined-view)
  - [5. Automated Maintenance](#5-automated-maintenance)
- [Example Implementation](#example-implementation)
- [Benefits and Considerations](#benefits-and-considerations)
  - [Benefits](#benefits)
  - [Limitations and Considerations](#limitations-and-considerations)
- [Conclusion](#conclusion)

## Introduction

Time-to-Live (TTL) mechanisms are essential for data management in databases, especially when dealing with regulatory compliance, storage optimization, and performance improvements. While PostgreSQL doesn't natively support column-level TTL, this guide presents an efficient approach using partitioning and views to achieve this functionality.

Traditional row-level deletion based on timestamps can severely impact performance when dealing with millions of records. The method outlined here leverages PostgreSQL's partitioning capabilities to make data expiration much more efficient.

## Core Concept

The fundamental approach involves:

1. **Separation of Data**: Dividing your schema into core (non-expiring) and expiring attributes
2. **Table Partitioning**: Partitioning the tables containing expiring data based on time ranges
3. **Views**: Creating unified views that combine core and non-expired data
4. **Automated Maintenance**: Using pg_partman for automatic partition management

Rather than deleting individual rows, this approach drops entire partitions of expired data, which is significantly faster and more efficient for large datasets.

## Step-by-Step Implementation

### 1. Schema Design

Start by separating your data into two categories:

**Main Table (Non-Expiring Data)**

```sql
CREATE TABLE main_table (
    id SERIAL PRIMARY KEY,  -- id INTEGER always generated restrict as identity  ????
    user_id INTEGER NOT NULL,
    created_at TIMESTAMP NOT NULL,
    -- Other non-expiring columns
);
```

**Expiring Table (Attributes that need TTL)**

```sql
CREATE TABLE expiring_table_parent (
    id INTEGER PRIMARY KEY, 
    expires_at TIMESTAMP NOT NULL,  -- Build in a default value like NOW() + INTERVAL '6 months' and   
                                    -- a check constraint expires_at is not null which  
                                    -- will force the expires_at at insertion as the defaultvalue
    sensitive_data TEXT,
    -- Other columns that need to expire
    FOREIGN KEY (id) REFERENCES main_table(id)
) PARTITION BY RANGE (expires_at);
```

### 2. Create the Partitioned Table

Next, create the initial partitions for your expiring data:

```sql
-- Create partitions for the current month and the next month
CREATE TABLE expiring_table_202505 
    PARTITION OF expiring_table_parent
    FOR VALUES FROM ('2025-05-01') TO ('2025-06-01');

CREATE TABLE expiring_table_202506 
    PARTITION OF expiring_table_parent
    FOR VALUES FROM ('2025-06-01') TO ('2025-07-01');
```

### 3. Set Up pg_partman

Install and configure pg_partman to automate partition management:

```sql
-- Install the extension (if not already installed)
CREATE EXTENSION pg_partman;

-- Configure pg_partman for the expiring_table_parent
SELECT create_parent(
    p_parent_table => 'public.expiring_table_parent',
    p_control => 'expires_at',
    p_type => 'native',
    p_interval => 'monthly',
    p_premake => 3
);
```

This configuration:

- Sets up automatic partition management for `expiring_table_parent`
- Uses `expires_at` as the partition control column
- Creates monthly partitions
- Pre-creates 3 future partitions in advance

### 4. Create the Combined View

Create a view that joins the main table with the non-expired data from the partitioned table:

```sql
CREATE VIEW data_view AS
SELECT
    m.*,
    e.sensitive_data,
    -- Include other columns from expiring_table
    e.expires_at
FROM
    main_table m
LEFT JOIN
    expiring_table_parent e ON m.id = e.id
WHERE
    e.expires_at > NOW() OR e.expires_at IS NULL;
```

This view will dynamically filter out any data that has expired, presenting only current valid data to applications.

### 5. Automated Maintenance

Set up a scheduled maintenance job to automatically manage partitions:

Using pg_cron:

```sql
-- Install pg_cron if not already available
CREATE EXTENSION pg_cron;

-- Schedule daily maintenance at 3 AM
SELECT cron.schedule('0 3 * * *', $$
    SELECT run_maintenance(p_parent_table := 'public.expiring_table_parent');
$$);
```

Alternatively, set up a system cron job with a SQL script:

```bash
# /etc/cron.d/pg_partman_maintenance
0 3 * * * postgres psql -d mydatabase -c "SELECT run_maintenance(p_parent_table := 'public.expiring_table_parent');"
```

The maintenance job will:

- Create new partitions as needed
- Drop partitions that have completely expired
- Perform other necessary maintenance tasks

## Example Implementation

Here's a complete example of implementing column-level TTL for a user data system with sensitive information:

```sql
-- Create the main users table
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Create the partitioned table for sensitive user data
CREATE TABLE user_sensitive_data (
    user_id INTEGER PRIMARY KEY,
    expires_at TIMESTAMP NOT NULL,
    payment_info TEXT,
    address TEXT,
    phone_number VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
) PARTITION BY RANGE (expires_at);

-- Create initial partitions
CREATE TABLE user_sensitive_data_202505 
    PARTITION OF user_sensitive_data
    FOR VALUES FROM ('2025-05-01') TO ('2025-06-01');

CREATE TABLE user_sensitive_data_202506 
    PARTITION OF user_sensitive_data
    FOR VALUES FROM ('2025-06-01') TO ('2025-07-01');

-- Set up pg_partman
SELECT create_parent(
    p_parent_table => 'public.user_sensitive_data',
    p_control => 'expires_at',
    p_type => 'native',
    p_interval => 'monthly',
    p_premake => 3
);

-- Create the unified view
CREATE VIEW active_users AS
SELECT
    u.user_id,
    u.username,
    u.email,
    u.created_at,
    s.payment_info,
    s.address,
    s.phone_number,
    s.expires_at
FROM
    users u
LEFT JOIN
    user_sensitive_data s ON u.user_id = s.user_id
WHERE
    s.expires_at > NOW() OR s.expires_at IS NULL;

-- Schedule maintenance
SELECT cron.schedule('0 3 * * *', $$
    SELECT run_maintenance(p_parent_table := 'public.user_sensitive_data');
$$);

-- To insert data with TTL
INSERT INTO users (username, email) 
VALUES ('johndoe', 'john@example.com');

INSERT INTO user_sensitive_data (user_id, payment_info, address, phone_number, expires_at)
VALUES (
    1, 
    'Card ending in 1234', 
    '123 Main St, Anytown', 
    '555-123-4567',
    NOW() + INTERVAL '6 months'  -- Data will expire in 6 months
);
```

## Benefits and Considerations

### Benefits

1. **Performance**: Dropping partitions is significantly faster than deleting millions of individual rows
2. **Scalability**: Efficiently manages large datasets by isolating expiring data
3. **Data Integrity**: Core data remains intact while only expiring attributes are removed
4. **Compliance**: Makes it easier to meet data retention requirements
5. **Storage Optimization**: Physically removes expired data, reducing storage costs
6. **Transparent to Applications**: Using views makes the TTL mechanism transparent to applications

### Limitations and Considerations

1. **Schema Complexity**: Requires more complex schema design than simple row-level TTL
2. **Planning Required**: Needs careful upfront planning for partitioning strategy
3. **Partitioning Key**: The partitioning key must be based on the expiration timestamp
4. **Primary/Unique Keys**: Primary and unique keys on partitioned tables must include the partition key
5. **Join Overhead**: There's a slight performance impact from the JOIN in the view
6. **Initial Setup**: More complex initial setup compared to simpler approaches

## Conclusion

Column-level TTL in PostgreSQL using partitioning provides a highly efficient approach for managing data expiration in large-scale applications. By leveraging PostgreSQL's native partitioning capabilities and tools like pg_partman, you can implement a robust, scalable solution that outperforms traditional row-deletion methods.

This approach is particularly valuable for applications dealing with:

- Regulatory compliance requirements
- Personal data protection laws (GDPR, CCPA)
- Large datasets with specific expiration needs
- Performance-critical systems

By physically removing expired data through partition dropping rather than row deletion, you can achieve both compliance and performance benefits while maintaining your core data integrity.
