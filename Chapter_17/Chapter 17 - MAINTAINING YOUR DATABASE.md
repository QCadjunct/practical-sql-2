# Chapter 17 - MAINTAINING YOUR DATABASE

## Introduction

This chapter covers essential maintenance tasks for PostgreSQL databases, including routine maintenance procedures, backup and recovery strategies, and database monitoring. Proper database maintenance is crucial for ensuring data integrity, optimal performance, and system reliability in production environments.

## 17.1 Routine Maintenance Tasks

### 17.1.1 VACUUM Processing

VACUUM is a critical PostgreSQL maintenance operation that reclaims storage occupied by dead tuples and makes it available for reuse.

#### How VACUUM Works

PostgreSQL's MVCC (Multiversion Concurrency Control) system creates new row versions when data is modified. Old versions, known as dead tuples, remain in the database until vacuumed.

```sql
-- Basic VACUUM syntax
VACUUM [FULL] [FREEZE] [VERBOSE] [ANALYZE] [table_name];

-- Example: Standard VACUUM on a specific table
VACUUM inventory.product;

-- Example: VACUUM with analysis to update statistics
VACUUM ANALYZE inventory.product;
```

When executed, VACUUM:
1. Reclaims space from dead tuples
2. Updates the visibility map
3. Updates free space maps
4. Optionally updates statistics (with ANALYZE)

#### Autovacuum Configuration

The autovacuum daemon automatically performs VACUUM operations based on configured parameters:

```sql
-- Checking autovacuum settings
SHOW autovacuum;
SHOW autovacuum_vacuum_threshold;
SHOW autovacuum_vacuum_scale_factor;

-- Example of configuring autovacuum for a specific table
ALTER TABLE inventory.product SET (
    autovacuum_vacuum_threshold = 100,
    autovacuum_vacuum_scale_factor = 0.1
);
```

Key autovacuum parameters to consider:

| Parameter | Description | Default |
|-----------|-------------|---------|
| autovacuum_vacuum_threshold | Minimum dead tuples before vacuum | 50 |
| autovacuum_vacuum_scale_factor | Fraction of table size triggering vacuum | 0.2 |
| autovacuum_analyze_threshold | Minimum modified tuples before analyze | 50 |
| autovacuum_analyze_scale_factor | Fraction of table size triggering analyze | 0.1 |

#### VACUUM FULL Operations

VACUUM FULL completely rebuilds the table, reclaiming maximum space but requires an exclusive lock.

```sql
-- VACUUM FULL operation - use with caution on production systems
VACUUM FULL inventory.product;
```

**Warning**: VACUUM FULL locks the table, blocks operations, and may impact application availability.

### 17.1.2 Database Statistics

PostgreSQL's query planner depends on up-to-date statistics to create efficient execution plans.

#### ANALYZE Command

```sql
-- Basic ANALYZE syntax
ANALYZE [VERBOSE] [table_name [(column_name [, ...])];

-- Example: Analyze a specific table
ANALYZE inventory.product;

-- Example: Analyze specific columns
ANALYZE inventory.product("ProductName", "StockQuantity");
```

ANALYZE:
1. Gathers statistics on data distribution
2. Updates the `pg_statistic` catalog
3. Helps the query planner make better execution decisions

#### Manual Statistics Management

For special cases, you can manually set statistics:

```sql
-- Set storage parameter for column statistics
ALTER TABLE inventory.product ALTER COLUMN "ProductName" SET STATISTICS 1000;
```

### 17.1.3 Managing Table Bloat

Table bloat occurs when tables contain large amounts of dead space, affecting performance.

```sql
-- Query to detect table bloat
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname || '.' || tablename)) as table_size,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename) - 
                  pg_relation_size(schemaname || '.' || tablename)) as bloat_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 20;

-- Addressing bloat with VACUUM FULL or pg_repack
VACUUM FULL inventory.product;
```

For production environments, consider using extensions like `pg_repack` to rebuild tables without exclusive locks.

## 17.2 Database Backup and Recovery

### 17.2.1 Logical Backups with pg_dump

`pg_dump` creates logical backups containing SQL statements to recreate the schema and data.

```bash
# Backing up a single database to a file
pg_dump -U username -d inventory -f inventory_backup.sql

# Creating a compressed backup
pg_dump -U username -d inventory | gzip > inventory_backup.sql.gz

# Custom format backup (allows parallel and selective restores)
pg_dump -U username -d inventory -Fc -f inventory_backup.dump

# Backing up selected tables
pg_dump -U username -d inventory -t inventory.product -t sales.order -f selected_tables.sql
```

#### Restoring Logical Backups

```bash
# Restoring a plain SQL backup
psql -U username -d inventory -f inventory_backup.sql

# Restoring a compressed backup
gunzip -c inventory_backup.sql.gz | psql -U username -d inventory

# Restoring a custom format backup
pg_restore -U username -d inventory inventory_backup.dump

# Parallel restore for faster processing
pg_restore -U username -d inventory -j 4 inventory_backup.dump
```

### 17.2.2 Physical Backups

Physical backups involve copying the database files directly, offering faster backups and restores for large databases.

#### Base Backups with pg_basebackup

```bash
# Create a base backup
pg_basebackup -h localhost -U postgres -D /backup/path -Ft -z -P

# Create a base backup with WAL streaming
pg_basebackup -h localhost -U postgres -D /backup/path -Ft -z -P -X stream
```

#### Point-in-Time Recovery (PITR)

Configure continuous WAL archiving in postgresql.conf:

```
wal_level = replica
archive_mode = on
archive_command = 'cp %p /path/to/archive/%f'
```

Restore process:

1. Restore the base backup
2. Create a recovery.conf file (PostgreSQL 12+ uses recovery.signal)
3. Configure recovery target (time, transaction ID, etc.)
4. Start PostgreSQL

```
# In postgresql.conf (for PostgreSQL 12+)
restore_command = 'cp /path/to/archive/%f %p'
recovery_target_time = '2023-10-15 14:30:00'
```

### 17.2.3 Backup Strategies and Planning

Effective backup strategies combine different approaches:

1. **Full backups**: Regular pg_dump or pg_basebackup operations
2. **Incremental backups**: WAL archiving for PITR
3. **Hybrid approach**: Weekly full backups with daily incremental backups

Key considerations:
- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Storage requirements
- Backup verification and testing

```sql
-- Create a smaller test database from a production backup for testing recovery
CREATE DATABASE inventory_test;
pg_restore -U username -d inventory_test --data-only inventory_backup.dump
```

## 17.3 Database Monitoring

### 17.3.1 Built-in Statistics Views

PostgreSQL provides several statistics views for monitoring database activity:

```sql
-- Query to check active sessions
SELECT pid, usename, application_name, client_addr, backend_start, state, query
FROM pg_stat_activity
WHERE state != 'idle';

-- Query to check table statistics
SELECT schemaname, relname, seq_scan, seq_tup_read, idx_scan, idx_tup_fetch, 
       n_tup_ins, n_tup_upd, n_tup_del, n_live_tup, n_dead_tup
FROM pg_stat_user_tables
WHERE schemaname = 'inventory';

-- Query to check index usage
SELECT schemaname, relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE schemaname = 'inventory';
```

#### Key Statistics Views

| View | Purpose |
|------|---------|
| pg_stat_activity | Active sessions and queries |
| pg_stat_user_tables | Table access statistics |
| pg_stat_user_indexes | Index usage statistics |
| pg_statio_user_tables | I/O statistics for tables |
| pg_stat_database | Database-wide statistics |

### 17.3.2 Query Performance Monitoring

Monitoring slow queries helps identify performance issues:

```sql
-- Enable query logging with execution time
ALTER SYSTEM SET log_min_duration_statement = '1000'; -- Log queries taking over 1 second
ALTER SYSTEM SET log_statement = 'none';
SELECT pg_reload_conf();

-- Query to find slow queries from pg_stat_statements (requires extension)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

SELECT query, calls, total_exec_time, mean_exec_time, 
       rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 17.3.3 Automated Maintenance with pg_cron

The pg_cron extension allows scheduling tasks directly within PostgreSQL:

```sql
-- Install pg_cron extension
CREATE EXTENSION pg_cron;

-- Schedule a VACUUM ANALYZE to run every day at 2 AM
SELECT cron.schedule('nightly-vacuum', '0 2 * * *', 'VACUUM ANALYZE inventory.product');

-- Schedule a weekly backup
SELECT cron.schedule('weekly-backup', '0 0 * * 0', 
                     'SELECT pg_dump_command(''inventory'', ''/backup/weekly/inventory_$date.dump'')');

-- List scheduled jobs
SELECT * FROM cron.job;
```

### 17.3.4 System Health Checks

Regular system health checks help maintain database reliability:

```sql
-- Check for bloated tables
SELECT
    schemaname || '.' || relname AS table_name,
    n_live_tup,
    n_dead_tup,
    round(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_tup_ratio
FROM pg_stat_user_tables
WHERE n_dead_tup > 100
ORDER BY n_dead_tup DESC;

-- Check for unused indexes
SELECT
    schemaname || '.' || relname AS table_name,
    indexrelname AS index_name,
    idx_scan,
    pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size
FROM pg_stat_user_indexes ui
JOIN pg_index i ON ui.indexrelid = i.indexrelid
WHERE NOT indisunique AND idx_scan < 50
AND pg_relation_size(i.indexrelid) > 10 * 1024 * 1024
ORDER BY pg_relation_size(i.indexrelid) DESC;

-- Check for long-running transactions
SELECT pid, usename, application_name, 
       age(now(), xact_start) AS xact_age,
       state, query
FROM pg_stat_activity
WHERE state <> 'idle' 
AND age(now(), xact_start) > interval '5 minutes'
ORDER BY xact_start;
```

## 17.4 Managing Database Growth

### 17.4.1 Table Partitioning

Partitioning large tables improves manageability and query performance:

```sql
-- Create a partitioned table for sales data
CREATE TABLE sales.order (
    order_id SERIAL,
    customer_id INTEGER NOT NULL,
    order_date DATE NOT NULL,
    total_amount NUMERIC(10,2) NOT NULL,
    PRIMARY KEY (order_id, order_date)
) PARTITION BY RANGE (order_date);

-- Create partitions by month
CREATE TABLE sales.order_y2023m01 PARTITION OF sales.order
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
    
CREATE TABLE sales.order_y2023m02 PARTITION OF sales.order
    FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');

-- Create a default partition for values outside defined ranges
CREATE TABLE sales.order_default PARTITION OF sales.order DEFAULT;
```

In Domain Driven Design naming convention:

```sql
CREATE TABLE "Sales"."Order" (
    "OrderId" SERIAL,
    "CustomerId" INTEGER NOT NULL,
    "OrderDate" DATE NOT NULL,
    "TotalAmount" NUMERIC(10,2) NOT NULL,
    PRIMARY KEY ("OrderId", "OrderDate")
) PARTITION BY RANGE ("OrderDate");

CREATE TABLE "Sales"."Order_Y2023M01" PARTITION OF "Sales"."Order"
    FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');
```

### 17.4.2 Data Retention Policies

Implementing data retention policies helps manage database size:

```sql
-- Create an audit table for moved data
CREATE TABLE sales.order_archive (LIKE sales.order);

-- Function to move old data to archive and delete from main table
CREATE OR REPLACE FUNCTION sales.archive_old_orders(cutoff_date DATE)
RETURNS INTEGER AS $$
DECLARE
    moved_count INTEGER;
BEGIN
    INSERT INTO sales.order_archive
    SELECT * FROM sales.order
    WHERE order_date < cutoff_date;
    
    GET DIAGNOSTICS moved_count = ROW_COUNT;
    
    DELETE FROM sales.order
    WHERE order_date < cutoff_date;
    
    RETURN moved_count;
END;
$$ LANGUAGE plpgsql;

-- Schedule periodic archiving with pg_cron
SELECT cron.schedule('monthly-order-archive', '0 1 1 * *', 
                     $$SELECT sales.archive_old_orders(current_date - interval '1 year')$$);
```

### 17.4.3 Database Storage Optimization

Strategies for optimizing storage usage:

```sql
-- Find largest tables and indexes
SELECT 
    n.nspname AS schema_name,
    c.relname AS relation_name,
    CASE c.relkind WHEN 'r' THEN 'table' WHEN 'i' THEN 'index' END AS type,
    pg_size_pretty(pg_relation_size(c.oid)) AS size,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size
FROM pg_class c
LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
WHERE c.relkind IN ('r', 'i')
AND n.nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(c.oid) DESC
LIMIT 20;

-- Optimize TOAST storage for large text fields
ALTER TABLE cms.content ALTER COLUMN content_body SET STORAGE EXTERNAL;

-- Use appropriate data types to reduce space
ALTER TABLE inventory.product 
    ALTER COLUMN small_integer_field TYPE SMALLINT,
    ALTER COLUMN large_integer_field TYPE BIGINT,
    ALTER COLUMN fixed_precision_field TYPE NUMERIC(10,2);
```

## 17.5 Handling Database Corruption

### 17.5.1 Detecting Corruption

Identifying database corruption:

```sql
-- Enable data page checksums (during initialization or pg_checksums)
initdb --data-checksums -D /path/to/data

-- Check for corruption with amcheck extension
CREATE EXTENSION amcheck;

-- Verify B-tree index integrity
SELECT bt_index_check(c.oid)
FROM pg_class c, pg_index i
WHERE c.oid = i.indexrelid
AND c.relnamespace::regnamespace::text = 'inventory'
AND c.relkind = 'i'
AND i.indisready AND i.indisvalid;

-- More thorough check (heavier)
SELECT bt_index_parent_check(c.oid, true)
FROM pg_class c, pg_index i
WHERE c.oid = i.indexrelid
AND c.relnamespace::regnamespace::text = 'inventory'
AND c.relkind = 'i'
AND i.indisready AND i.indisvalid;
```

### 17.5.2 Recovering from Corruption

Steps for recovering from database corruption:

1. Identify corrupted objects:
```sql
-- Check for corruption indicators in logs
-- Look for "checksum failure" or "invalid page" errors
```

2. Attempt isolated recovery:
```sql
-- If corruption is in an index, try rebuilding it
REINDEX TABLE inventory.product;

-- If corruption is in a table, try to dump what's possible
pg_dump -U username -d inventory -t inventory.product --data-only --exclude-table-data=inventory.product | grep -v "ERROR:"
```

3. Point-in-time recovery from backups:
```
# Restore from the latest backup before corruption occurred
# Configure recovery to a target time before corruption
```

4. Document lessons learned and improve backup strategy.

## 17.6 Database Upgrades

### 17.6.1 Minor Version Upgrades

For minor version upgrades (e.g., 17.1 to 17.2), the process is:

```bash
# Stop PostgreSQL
sudo systemctl stop postgresql

# Back up the database
pg_dump -U username -d inventory -f inventory_pre_upgrade.dump

# Update PostgreSQL packages
sudo apt update
sudo apt upgrade postgresql

# Start PostgreSQL
sudo systemctl start postgresql

# Verify the version
psql -U username -d inventory -c "SELECT version();"
```

### 17.6.2 Major Version Upgrades

Major version upgrades (e.g., 16 to 17) require more steps:

```bash
# Method 1: pg_dumpall
pg_dumpall -U postgres > full_dump.sql
# Install new version
# Create new data directory
# Initialize new data directory
initdb -D /path/to/new/datadir
# Start new PostgreSQL
# Restore data
psql -U postgres -f full_dump.sql

# Method 2: pg_upgrade
# Install new version alongside old
# Initialize new data directory
pg_upgrade -b /path/to/old/bin -B /path/to/new/bin \
           -d /path/to/old/datadir -D /path/to/new/datadir \
           -c
```

### 17.6.3 Testing Upgrades

Always test upgrades in a staging environment before production:

```bash
# Create a duplicate database
createdb -T inventory inventory_test

# Run database-specific tests
# Check application compatibility
# Verify extensions functionality
# Compare performance metrics

# Generate a list of differences for comparison
pg_dump -U username -d old_db -s > old_schema.sql
pg_dump -U username -d new_db -s > new_schema.sql
diff -u old_schema.sql new_schema.sql > schema_diffs.txt
```

## 17.7 Ongoing Maintenance Best Practices

### 17.7.1 Maintenance Checklist

Regular maintenance tasks to implement:

1. **Daily**:
   - Monitor active sessions and long-running queries
   - Check for database errors in logs
   - Verify backup success

2. **Weekly**:
   - Review slow query logs
   - Check for table bloat
   - Analyze system performance metrics
   - Test recovery procedures

3. **Monthly**:
   - Review access permissions
   - Check for unused indexes
   - Optimize storage usage
   - Review PostgreSQL configuration

4. **Quarterly**:
   - Plan capacity upgrades
   - Review backup and retention policies
   - Evaluate upgrade requirements
   - Test disaster recovery procedures

### 17.7.2 Automating Maintenance

Example of a maintenance script using pg_cron:

```sql
-- Create a maintenance schema
CREATE SCHEMA maintenance;

-- Create logging table
CREATE TABLE maintenance.maintenance_log (
    log_id SERIAL PRIMARY KEY,
    operation TEXT NOT NULL,
    start_time TIMESTAMP WITH TIME ZONE DEFAULT now(),
    end_time TIMESTAMP WITH TIME ZONE,
    rows_affected BIGINT,
    status TEXT,
    notes TEXT
);

-- Create maintenance function
CREATE OR REPLACE FUNCTION maintenance.run_vacuum_analyze()
RETURNS void AS $$
DECLARE
    start_time TIMESTAMP WITH TIME ZONE := clock_timestamp();
    end_time TIMESTAMP WITH TIME ZONE;
    log_id INTEGER;
    tables_vacuumed INTEGER := 0;
BEGIN
    -- Log start
    INSERT INTO maintenance.maintenance_log(operation, start_time)
    VALUES ('vacuum_analyze_all', start_time)
    RETURNING log_id INTO log_id;
    
    -- Run maintenance on user tables
    FOR tables_vacuumed IN 
        EXECUTE 'VACUUM ANALYZE'
    LOOP
        -- This loop only runs once but helps capture errors
    END LOOP;
    
    -- Log completion
    end_time := clock_timestamp();
    UPDATE maintenance.maintenance_log
    SET end_time = end_time,
        rows_affected = tables_vacuumed,
        status = 'SUCCESS',
        notes = format('Vacuum analyze completed in %s', end_time - start_time)
    WHERE log_id = log_id;
    
EXCEPTION WHEN OTHERS THEN
    -- Log failure
    UPDATE maintenance.maintenance_log
    SET end_time = clock_timestamp(),
        status = 'ERROR',
        notes = format('Error: %s', SQLERRM)
    WHERE log_id = log_id;
END;
$$ LANGUAGE plpgsql;

-- Schedule regular maintenance
SELECT cron.schedule('nightly-vacuum-analyze', '0 3 * * *', 
                     'SELECT maintenance.run_vacuum_analyze()');
```

In Domain Driven Design naming convention:

```sql
CREATE SCHEMA "Maintenance";

CREATE TABLE "Maintenance"."MaintenanceLog" (
    "LogId" SERIAL PRIMARY KEY,
    "Operation" TEXT NOT NULL,
    "StartTime" TIMESTAMP WITH TIME ZONE DEFAULT now(),
    "EndTime" TIMESTAMP WITH TIME ZONE,
    "RowsAffected" BIGINT,
    "Status" TEXT,
    "Notes" TEXT
);

CREATE OR REPLACE FUNCTION "Maintenance"."RunVacuumAnalyze"()
RETURNS void AS $$
-- Function body remains the same
$$ LANGUAGE plpgsql;
```

## Conclusion

Proper database maintenance is essential for ensuring PostgreSQL database reliability, performance, and data integrity. By implementing regular vacuuming, backups, monitoring, and following upgrade best practices, you can keep your database systems running efficiently and minimize downtime. Automating routine maintenance tasks reduces administrative overhead and helps prevent problems before they affect your applications.