# Level 3: Database Administration and Management

## Prerequisites
- Level 2 completion (advanced query writing, CTEs, window functions)
- Understanding of database internals and architecture
- Experience with production database environments
- Knowledge of operating system administration (Linux/Windows)
- Familiarity with backup and recovery concepts
- Understanding of security principles

## Problem Statement
As a senior backend engineer and database administrator, you need to:
- **Manage database security** with proper authentication and authorization
- **Implement backup and recovery** strategies for business continuity
- **Monitor and tune performance** for optimal database operations
- **Handle database maintenance** including updates, patches, and migrations
- **Design high availability** and disaster recovery solutions
- **Manage user access** and permissions effectively
- **Implement compliance** with data protection regulations
- **Troubleshoot complex issues** in production environments

These skills are essential for maintaining enterprise-grade database systems that support critical business operations.

---

## Key Concepts

### 1. Database Security Management

#### User Management and Authentication
```sql
-- PostgreSQL User Management
-- Create roles with specific privileges
CREATE ROLE app_read_only;
GRANT CONNECT ON DATABASE myapp TO app_read_only;
GRANT USAGE ON SCHEMA public TO app_read_only;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_read_only;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO app_read_only;

CREATE ROLE app_read_write;
GRANT app_read_only TO app_read_write;
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_read_write;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT INSERT, UPDATE, DELETE ON TABLES TO app_read_write;

-- Create application users
CREATE USER app_user WITH PASSWORD 'secure_password_123!';
GRANT app_read_write TO app_user;

CREATE USER reporting_user WITH PASSWORD 'reporting_pass_456!';
GRANT app_read_only TO reporting_user;

-- Create admin user with limited privileges
CREATE USER db_admin WITH PASSWORD 'admin_pass_789!' CREATEDB;
GRANT ALL PRIVILEGES ON DATABASE myapp TO db_admin;

-- MySQL User Management
CREATE USER 'app_user'@'%' IDENTIFIED BY 'secure_password_123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_user'@'%';

CREATE USER 'reporting_user'@'%' IDENTIFIED BY 'reporting_pass_456!';
GRANT SELECT ON myapp.* TO 'reporting_user'@'%';

-- Restrict access by IP
CREATE USER 'admin_user'@'192.168.1.%' IDENTIFIED BY 'admin_pass_789!';
GRANT ALL PRIVILEGES ON myapp.* TO 'admin_user'@'192.168.1.%';

FLUSH PRIVILEGES;
```

#### Row-Level Security (PostgreSQL)
```sql
-- Enable row-level security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Create policies for different user types
CREATE POLICY customer_orders_policy ON orders
    FOR ALL TO app_user
    USING (customer_id = current_setting('app.current_customer_id')::INTEGER);

CREATE POLICY admin_full_access ON orders
    FOR ALL TO db_admin
    USING (true);

CREATE POLICY reporting_read_only ON orders
    FOR SELECT TO reporting_user
    USING (order_date >= CURRENT_DATE - INTERVAL '2 years');

-- Set application context
-- In application code:
-- SET app.current_customer_id = '123';

-- Test row-level security
SET ROLE app_user;
SET app.current_customer_id = '123';
SELECT * FROM orders; -- Only shows orders for customer 123

RESET ROLE;
```

#### Data Encryption and Security
```sql
-- PostgreSQL encryption functions
-- Encrypt sensitive data
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Create table with encrypted columns
CREATE TABLE customer_sensitive (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    encrypted_ssn BYTEA,
    encrypted_credit_card BYTEA,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert encrypted data
INSERT INTO customer_sensitive (customer_id, encrypted_ssn, encrypted_credit_card)
VALUES (
    123,
    pgp_sym_encrypt('123-45-6789', 'encryption_key_here'),
    pgp_sym_encrypt('4111-1111-1111-1111', 'encryption_key_here')
);

-- Query encrypted data
SELECT 
    customer_id,
    pgp_sym_decrypt(encrypted_ssn, 'encryption_key_here') as ssn,
    pgp_sym_decrypt(encrypted_credit_card, 'encryption_key_here') as credit_card
FROM customer_sensitive
WHERE customer_id = 123;

-- Create function for secure data access
CREATE OR REPLACE FUNCTION get_customer_sensitive_data(
    p_customer_id INTEGER,
    p_encryption_key TEXT
) RETURNS TABLE (
    customer_id INTEGER,
    ssn TEXT,
    credit_card_last_four TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        cs.customer_id,
        pgp_sym_decrypt(cs.encrypted_ssn, p_encryption_key) as ssn,
        RIGHT(pgp_sym_decrypt(cs.encrypted_credit_card, p_encryption_key), 4) as credit_card_last_four
    FROM customer_sensitive cs
    WHERE cs.customer_id = p_customer_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Revoke direct table access
REVOKE ALL ON customer_sensitive FROM PUBLIC;
GRANT EXECUTE ON FUNCTION get_customer_sensitive_data TO app_user;
```

### 2. Backup and Recovery Strategies

#### PostgreSQL Backup Strategies
```bash
#!/bin/bash
# comprehensive-backup.sh - Complete PostgreSQL backup solution

# Configuration
DB_HOST="localhost"
DB_PORT="5432"
DB_NAME="myapp"
DB_USER="postgres"
BACKUP_DIR="/backups/postgresql"
S3_BUCKET="my-db-backups"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Full database backup
perform_full_backup() {
    local backup_file="$BACKUP_DIR/full_backup_${DB_NAME}_${DATE}.sql.gz"
    
    log "Starting full backup of database: $DB_NAME"
    
    pg_dump -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        --verbose --no-password --format=custom --compress=9 \
        --file="${backup_file%.gz}" \
        --exclude-table-data='audit_logs' \
        --exclude-table-data='session_data'
    
    if [ $? -eq 0 ]; then
        gzip "${backup_file%.gz}"
        log "Full backup completed: $backup_file"
        
        # Upload to S3
        aws s3 cp "$backup_file" "s3://$S3_BUCKET/full/"
        
        # Verify backup integrity
        verify_backup "$backup_file"
    else
        log "Full backup failed"
        exit 1
    fi
}

# Incremental backup using WAL archiving
setup_wal_archiving() {
    log "Setting up WAL archiving"
    
    # PostgreSQL configuration (add to postgresql.conf)
    cat << EOF >> /etc/postgresql/15/main/postgresql.conf
# WAL archiving configuration
wal_level = replica
archive_mode = on
archive_command = 'test ! -f /backups/wal_archive/%f && cp %p /backups/wal_archive/%f'
max_wal_senders = 3
wal_keep_size = 1GB
EOF
    
    # Create WAL archive directory
    mkdir -p /backups/wal_archive
    chown postgres:postgres /backups/wal_archive
    
    log "WAL archiving configured. Restart PostgreSQL to apply changes."
}

# Point-in-time recovery backup
perform_pitr_backup() {
    local backup_file="$BACKUP_DIR/pitr_backup_${DB_NAME}_${DATE}.tar.gz"
    
    log "Starting point-in-time recovery backup"
    
    # Start backup
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "SELECT pg_start_backup('PITR Backup $DATE', false, false);"
    
    # Backup data directory
    tar -czf "$backup_file" -C /var/lib/postgresql/15/main .
    
    # Stop backup
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "SELECT pg_stop_backup(false, true);"
    
    log "PITR backup completed: $backup_file"
    
    # Upload to S3
    aws s3 cp "$backup_file" "s3://$S3_BUCKET/pitr/"
}

# Verify backup integrity
verify_backup() {
    local backup_file="$1"
    local test_db="test_restore_$(date +%s)"
    
    log "Verifying backup integrity: $backup_file"
    
    # Create test database
    createdb -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" "$test_db"
    
    # Restore backup to test database
    if [[ "$backup_file" == *.gz ]]; then
        gunzip -c "$backup_file" | pg_restore -h "$DB_HOST" -p "$DB_PORT" \
            -U "$DB_USER" -d "$test_db" --verbose
    else
        pg_restore -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" \
            -d "$test_db" --verbose "$backup_file"
    fi
    
    if [ $? -eq 0 ]; then
        log "Backup verification successful"
        
        # Run basic integrity checks
        psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$test_db" \
            -c "SELECT COUNT(*) FROM information_schema.tables;"
    else
        log "Backup verification failed"
    fi
    
    # Cleanup test database
    dropdb -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" "$test_db"
}

# Cleanup old backups
cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days"
    
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
    find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
    
    # Cleanup S3 backups
    aws s3 ls "s3://$S3_BUCKET/" --recursive | \
        awk '$1 < "'$(date -d "$RETENTION_DAYS days ago" +%Y-%m-%d)'" {print $4}' | \
        xargs -I {} aws s3 rm "s3://$S3_BUCKET/{}"
}

# Main execution
case "$1" in
    "full")
        perform_full_backup
        ;;
    "pitr")
        perform_pitr_backup
        ;;
    "setup-wal")
        setup_wal_archiving
        ;;
    "cleanup")
        cleanup_old_backups
        ;;
    *)
        echo "Usage: $0 {full|pitr|setup-wal|cleanup}"
        exit 1
        ;;
esac

log "Backup operation completed"
```

#### MySQL Backup Strategies
```bash
#!/bin/bash
# mysql-backup.sh - Comprehensive MySQL backup solution

# Configuration
DB_HOST="localhost"
DB_PORT="3306"
DB_NAME="myapp"
DB_USER="backup_user"
DB_PASS="backup_password"
BACKUP_DIR="/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

# Full backup with mysqldump
perform_full_backup() {
    local backup_file="$BACKUP_DIR/full_backup_${DB_NAME}_${DATE}.sql.gz"
    
    log "Starting full MySQL backup"
    
    mysqldump --host="$DB_HOST" --port="$DB_PORT" \
        --user="$DB_USER" --password="$DB_PASS" \
        --single-transaction --routines --triggers \
        --add-drop-database --databases "$DB_NAME" | \
        gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        log "Full backup completed: $backup_file"
    else
        log "Full backup failed"
        exit 1
    fi
}

# Binary log backup for point-in-time recovery
setup_binlog_backup() {
    log "Setting up binary log backup"
    
    # MySQL configuration (add to my.cnf)
    cat << EOF >> /etc/mysql/my.cnf
[mysqld]
log-bin=mysql-bin
server-id=1
binlog-format=ROW
expire_logs_days=7
max_binlog_size=100M
EOF
    
    log "Binary logging configured. Restart MySQL to apply changes."
}

# Incremental backup using binary logs
perform_incremental_backup() {
    local backup_file="$BACKUP_DIR/incremental_${DB_NAME}_${DATE}.sql"
    
    log "Starting incremental backup"
    
    # Get current binary log position
    mysql --host="$DB_HOST" --port="$DB_PORT" \
        --user="$DB_USER" --password="$DB_PASS" \
        -e "SHOW MASTER STATUS\G" > "$BACKUP_DIR/binlog_position_${DATE}.txt"
    
    # Flush logs to ensure all changes are written
    mysql --host="$DB_HOST" --port="$DB_PORT" \
        --user="$DB_USER" --password="$DB_PASS" \
        -e "FLUSH LOGS;"
    
    # Copy binary logs
    cp /var/lib/mysql/mysql-bin.* "$BACKUP_DIR/binlogs/"
    
    log "Incremental backup completed"
}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Execute based on parameter
case "$1" in
    "full")
        perform_full_backup
        ;;
    "incremental")
        perform_incremental_backup
        ;;
    "setup-binlog")
        setup_binlog_backup
        ;;
    *)
        echo "Usage: $0 {full|incremental|setup-binlog}"
        exit 1
        ;;
esac
```

#### Recovery Procedures
```bash
#!/bin/bash
# recovery-procedures.sh - Database recovery procedures

# PostgreSQL Point-in-Time Recovery
perform_postgresql_pitr() {
    local backup_file="$1"
    local target_time="$2"
    local recovery_dir="/var/lib/postgresql/15/recovery"
    
    log "Starting PostgreSQL point-in-time recovery"
    log "Target time: $target_time"
    
    # Stop PostgreSQL
    systemctl stop postgresql
    
    # Backup current data directory
    mv /var/lib/postgresql/15/main /var/lib/postgresql/15/main.backup.$(date +%s)
    
    # Create new data directory
    mkdir -p "$recovery_dir"
    chown postgres:postgres "$recovery_dir"
    
    # Restore base backup
    tar -xzf "$backup_file" -C "$recovery_dir"
    
    # Create recovery configuration
    cat << EOF > "$recovery_dir/postgresql.auto.conf"
restore_command = 'cp /backups/wal_archive/%f %p'
recovery_target_time = '$target_time'
recovery_target_action = 'promote'
EOF
    
    # Create recovery signal file
    touch "$recovery_dir/recovery.signal"
    
    # Update data directory path
    sed -i "s|/var/lib/postgresql/15/main|$recovery_dir|g" /etc/postgresql/15/main/postgresql.conf
    
    # Start PostgreSQL
    systemctl start postgresql
    
    log "PostgreSQL PITR recovery initiated"
    log "Monitor logs: tail -f /var/log/postgresql/postgresql-15-main.log"
}

# MySQL Point-in-Time Recovery
perform_mysql_pitr() {
    local backup_file="$1"
    local target_time="$2"
    local temp_db="recovery_temp_$(date +%s)"
    
    log "Starting MySQL point-in-time recovery"
    log "Target time: $target_time"
    
    # Create temporary database
    mysql -u root -p -e "CREATE DATABASE $temp_db;"
    
    # Restore full backup
    gunzip -c "$backup_file" | mysql -u root -p "$temp_db"
    
    # Apply binary logs up to target time
    mysqlbinlog --stop-datetime="$target_time" /backups/mysql/binlogs/mysql-bin.* | \
        mysql -u root -p "$temp_db"
    
    log "Recovery completed to temporary database: $temp_db"
    log "Verify data and rename database when ready"
}

# Automated recovery testing
test_recovery_procedures() {
    local test_backup="$1"
    
    log "Testing recovery procedures"
    
    # Create test environment
    docker run -d --name postgres-recovery-test \
        -e POSTGRES_PASSWORD=test123 \
        -v "$(dirname $test_backup):/backups" \
        postgres:15
    
    # Wait for container to start
    sleep 10
    
    # Test restore
    docker exec postgres-recovery-test \
        pg_restore -U postgres -d postgres "/backups/$(basename $test_backup)"
    
    if [ $? -eq 0 ]; then
        log "Recovery test successful"
    else
        log "Recovery test failed"
    fi
    
    # Cleanup
    docker stop postgres-recovery-test
    docker rm postgres-recovery-test
}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Execute based on parameters
case "$1" in
    "postgresql-pitr")
        perform_postgresql_pitr "$2" "$3"
        ;;
    "mysql-pitr")
        perform_mysql_pitr "$2" "$3"
        ;;
    "test")
        test_recovery_procedures "$2"
        ;;
    *)
        echo "Usage: $0 {postgresql-pitr|mysql-pitr|test} [backup_file] [target_time]"
        exit 1
        ;;
esac
```

### 3. Performance Monitoring and Tuning

#### PostgreSQL Performance Monitoring
```sql
-- Performance monitoring queries

-- 1. Slow query analysis
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows,
    100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 20;

-- 2. Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes 
ORDER BY idx_scan DESC;

-- 3. Table statistics and bloat analysis
SELECT 
    schemaname,
    tablename,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    n_live_tup,
    n_dead_tup,
    ROUND(n_dead_tup * 100.0 / GREATEST(n_live_tup + n_dead_tup, 1), 2) as dead_tuple_percent,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables 
ORDER BY n_dead_tup DESC;

-- 4. Connection and activity monitoring
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query_start,
    state_change,
    EXTRACT(EPOCH FROM (now() - query_start)) as query_duration_seconds,
    LEFT(query, 100) as query_preview
FROM pg_stat_activity 
WHERE state != 'idle'
ORDER BY query_start;

-- 5. Lock monitoring
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.GRANTED;

-- 6. Database size and growth monitoring
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) as database_size,
    pg_database_size(datname) as size_bytes
FROM pg_database 
ORDER BY pg_database_size(datname) DESC;
```

#### Performance Tuning Configuration
```sql
-- PostgreSQL performance tuning parameters
-- Add to postgresql.conf

-- Memory settings
shared_buffers = '256MB'                    -- 25% of RAM for dedicated server
effective_cache_size = '1GB'                -- 50-75% of RAM
work_mem = '4MB'                            -- Per operation memory
maintenance_work_mem = '64MB'               -- For maintenance operations

-- Checkpoint settings
checkpoint_completion_target = 0.9          -- Spread checkpoints over time
wal_buffers = '16MB'                        -- WAL buffer size
max_wal_size = '1GB'                        -- Maximum WAL size
min_wal_size = '80MB'                       -- Minimum WAL size

-- Query planner settings
random_page_cost = 1.1                      -- For SSD storage
effective_io_concurrency = 200               -- For SSD storage

-- Connection settings
max_connections = 100                        -- Adjust based on application needs

-- Logging settings for performance monitoring
log_min_duration_statement = 1000            -- Log queries taking > 1 second
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on

-- Statistics collection
track_activities = on
track_counts = on
track_io_timing = on
track_functions = all
```

#### MySQL Performance Monitoring
```sql
-- MySQL performance monitoring queries

-- 1. Slow query analysis (requires slow query log)
SELECT 
    sql_text,
    exec_count,
    total_latency,
    avg_latency,
    rows_examined_avg,
    rows_sent_avg
FROM sys.statements_with_full_table_scans
ORDER BY total_latency DESC
LIMIT 20;

-- 2. Index usage statistics
SELECT 
    object_schema,
    object_name,
    index_name,
    count_read,
    count_write,
    count_read + count_write as total_usage
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema NOT IN ('mysql', 'performance_schema', 'information_schema')
ORDER BY total_usage DESC;

-- 3. Table statistics
SELECT 
    table_schema,
    table_name,
    table_rows,
    ROUND(data_length / 1024 / 1024, 2) as data_size_mb,
    ROUND(index_length / 1024 / 1024, 2) as index_size_mb,
    ROUND((data_length + index_length) / 1024 / 1024, 2) as total_size_mb
FROM information_schema.tables
WHERE table_schema NOT IN ('mysql', 'performance_schema', 'information_schema')
ORDER BY (data_length + index_length) DESC;

-- 4. Connection monitoring
SELECT 
    id,
    user,
    host,
    db,
    command,
    time,
    state,
    LEFT(info, 100) as query_preview
FROM information_schema.processlist
WHERE command != 'Sleep'
ORDER BY time DESC;

-- 5. InnoDB status monitoring
SELECT 
    variable_name,
    variable_value
FROM performance_schema.global_status
WHERE variable_name IN (
    'Innodb_buffer_pool_read_requests',
    'Innodb_buffer_pool_reads',
    'Innodb_buffer_pool_pages_dirty',
    'Innodb_buffer_pool_pages_free',
    'Innodb_buffer_pool_pages_total',
    'Innodb_rows_read',
    'Innodb_rows_inserted',
    'Innodb_rows_updated',
    'Innodb_rows_deleted'
);
```

### 4. Database Maintenance

#### Automated Maintenance Scripts
```bash
#!/bin/bash
# database-maintenance.sh - Comprehensive database maintenance

# Configuration
DB_TYPE="postgresql"  # or "mysql"
DB_HOST="localhost"
DB_PORT="5432"
DB_NAME="myapp"
DB_USER="postgres"
LOG_FILE="/var/log/db-maintenance.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# PostgreSQL maintenance
postgresql_maintenance() {
    log "Starting PostgreSQL maintenance for database: $DB_NAME"
    
    # Update table statistics
    log "Updating table statistics..."
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "ANALYZE;" >> "$LOG_FILE" 2>&1
    
    # Vacuum tables
    log "Performing VACUUM..."
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "VACUUM (ANALYZE, VERBOSE);" >> "$LOG_FILE" 2>&1
    
    # Reindex heavily used indexes
    log "Reindexing database..."
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "REINDEX DATABASE $DB_NAME;" >> "$LOG_FILE" 2>&1
    
    # Check for bloated tables
    log "Checking for table bloat..."
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" << 'EOF' >> "$LOG_FILE" 2>&1
SELECT 
    schemaname,
    tablename,
    n_dead_tup,
    n_live_tup,
    ROUND(n_dead_tup * 100.0 / GREATEST(n_live_tup + n_dead_tup, 1), 2) as dead_tuple_percent
FROM pg_stat_user_tables 
WHERE n_dead_tup > 1000
ORDER BY dead_tuple_percent DESC;
EOF
    
    # Update extensions
    log "Updating extensions..."
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
        -c "SELECT name, installed_version, default_version FROM pg_available_extensions WHERE installed_version IS NOT NULL AND installed_version != default_version;" >> "$LOG_FILE" 2>&1
}

# MySQL maintenance
mysql_maintenance() {
    log "Starting MySQL maintenance for database: $DB_NAME"
    
    # Optimize tables
    log "Optimizing tables..."
    mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
        -e "SELECT CONCAT('OPTIMIZE TABLE ', table_schema, '.', table_name, ';') FROM information_schema.tables WHERE table_schema = '$DB_NAME';" \
        | grep -v CONCAT | mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" >> "$LOG_FILE" 2>&1
    
    # Analyze tables
    log "Analyzing tables..."
    mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
        -e "SELECT CONCAT('ANALYZE TABLE ', table_schema, '.', table_name, ';') FROM information_schema.tables WHERE table_schema = '$DB_NAME';" \
        | grep -v CONCAT | mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" >> "$LOG_FILE" 2>&1
    
    # Check and repair tables
    log "Checking tables for errors..."
    mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
        -e "SELECT CONCAT('CHECK TABLE ', table_schema, '.', table_name, ';') FROM information_schema.tables WHERE table_schema = '$DB_NAME' AND engine = 'InnoDB';" \
        | grep -v CONCAT | mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" >> "$LOG_FILE" 2>&1
    
    # Update table statistics
    log "Updating table statistics..."
    mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
        -e "SELECT CONCAT('ANALYZE TABLE ', table_schema, '.', table_name, ';') FROM information_schema.tables WHERE table_schema = '$DB_NAME';" \
        | grep -v CONCAT | mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" >> "$LOG_FILE" 2>&1
}

# Check disk space
check_disk_space() {
    log "Checking disk space..."
    df -h | grep -E "/$|/var|/tmp" >> "$LOG_FILE"
    
    # Alert if disk usage > 80%
    DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$DISK_USAGE" -gt 80 ]; then
        log "WARNING: Disk usage is ${DISK_USAGE}%"
        # Send alert (implement your alerting mechanism)
        # send_alert "High disk usage: ${DISK_USAGE}%"
    fi
}

# Archive old logs
archive_logs() {
    log "Archiving old log files..."
    
    # Archive logs older than 30 days
    find /var/log -name "*.log" -mtime +30 -exec gzip {} \;
    
    # Remove archived logs older than 90 days
    find /var/log -name "*.log.gz" -mtime +90 -delete
    
    log "Log archiving completed"
}

# Generate maintenance report
generate_report() {
    local report_file="/tmp/db_maintenance_report_$(date +%Y%m%d).txt"
    
    log "Generating maintenance report: $report_file"
    
    cat << EOF > "$report_file"
Database Maintenance Report
Generated: $(date)
Database: $DB_NAME
Host: $DB_HOST

=== Maintenance Summary ===
EOF
    
    # Add database-specific information
    if [ "$DB_TYPE" = "postgresql" ]; then
        echo "\n=== PostgreSQL Statistics ===" >> "$report_file"
        psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
            -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database WHERE datname = '$DB_NAME';" >> "$report_file"
    elif [ "$DB_TYPE" = "mysql" ]; then
        echo "\n=== MySQL Statistics ===" >> "$report_file"
        mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
            -e "SELECT table_schema, ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'DB Size in MB' FROM information_schema.tables WHERE table_schema = '$DB_NAME';" >> "$report_file"
    fi
    
    # Add system information
    echo "\n=== System Information ===" >> "$report_file"
    df -h >> "$report_file"
    
    log "Maintenance report generated: $report_file"
}

# Main execution
log "Starting database maintenance routine"

check_disk_space

if [ "$DB_TYPE" = "postgresql" ]; then
    postgresql_maintenance
elif [ "$DB_TYPE" = "mysql" ]; then
    mysql_maintenance
else
    log "ERROR: Unsupported database type: $DB_TYPE"
    exit 1
fi

archive_logs
generate_report

log "Database maintenance completed successfully"
```

### 5. High Availability and Disaster Recovery

#### PostgreSQL Streaming Replication Setup
```bash
#!/bin/bash
# setup-postgresql-replication.sh - PostgreSQL streaming replication

# Configuration
PRIMARY_HOST="10.0.1.10"
REPLICA_HOST="10.0.1.11"
REPL_USER="replicator"
REPL_PASS="replication_password_123"
PG_VERSION="15"
DATA_DIR="/var/lib/postgresql/$PG_VERSION/main"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Setup primary server
setup_primary() {
    log "Setting up primary PostgreSQL server"
    
    # Create replication user
    sudo -u postgres psql << EOF
CREATE USER $REPL_USER REPLICATION LOGIN ENCRYPTED PASSWORD '$REPL_PASS';
EOF
    
    # Configure postgresql.conf
    cat << EOF >> /etc/postgresql/$PG_VERSION/main/postgresql.conf
# Replication settings
wal_level = replica
max_wal_senders = 3
max_replication_slots = 3
wal_keep_size = 1GB
archive_mode = on
archive_command = 'test ! -f /var/lib/postgresql/wal_archive/%f && cp %p /var/lib/postgresql/wal_archive/%f'
EOF
    
    # Configure pg_hba.conf
    echo "host replication $REPL_USER $REPLICA_HOST/32 md5" >> /etc/postgresql/$PG_VERSION/main/pg_hba.conf
    
    # Create WAL archive directory
    mkdir -p /var/lib/postgresql/wal_archive
    chown postgres:postgres /var/lib/postgresql/wal_archive
    
    # Restart PostgreSQL
    systemctl restart postgresql
    
    log "Primary server setup completed"
}

# Setup replica server
setup_replica() {
    log "Setting up replica PostgreSQL server"
    
    # Stop PostgreSQL
    systemctl stop postgresql
    
    # Backup existing data directory
    mv "$DATA_DIR" "${DATA_DIR}.backup.$(date +%s)"
    
    # Create base backup from primary
    sudo -u postgres pg_basebackup -h "$PRIMARY_HOST" -D "$DATA_DIR" \
        -U "$REPL_USER" -W -v -P -R
    
    # Configure standby settings
    cat << EOF >> "$DATA_DIR/postgresql.auto.conf"
# Standby settings
hot_standby = on
max_standby_streaming_delay = 30s
wal_receiver_status_interval = 10s
hot_standby_feedback = on
EOF
    
    # Start PostgreSQL
    systemctl start postgresql
    
    log "Replica server setup completed"
}

# Monitor replication status
monitor_replication() {
    log "Monitoring replication status"
    
    # On primary server
    echo "=== Primary Server Status ==="
    sudo -u postgres psql -c "SELECT client_addr, state, sent_lsn, write_lsn, flush_lsn, replay_lsn, sync_state FROM pg_stat_replication;"
    
    # On replica server
    echo "\n=== Replica Server Status ==="
    sudo -u postgres psql -c "SELECT pg_is_in_recovery(), pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp();"
    
    # Check replication lag
    echo "\n=== Replication Lag ==="
    sudo -u postgres psql -c "SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp())) AS lag_seconds;"
}

# Failover procedure
perform_failover() {
    log "Performing failover to replica server"
    
    # Promote replica to primary
    sudo -u postgres pg_ctl promote -D "$DATA_DIR"
    
    # Update application connection strings
    log "Update application configuration to point to new primary: $REPLICA_HOST"
    
    # Reconfigure old primary as new replica (manual step)
    log "Manual step required: Reconfigure old primary as replica"
}

# Execute based on parameter
case "$1" in
    "setup-primary")
        setup_primary
        ;;
    "setup-replica")
        setup_replica
        ;;
    "monitor")
        monitor_replication
        ;;
    "failover")
        perform_failover
        ;;
    *)
        echo "Usage: $0 {setup-primary|setup-replica|monitor|failover}"
        exit 1
        ;;
esac
```

#### MySQL Master-Slave Replication
```bash
#!/bin/bash
# setup-mysql-replication.sh - MySQL master-slave replication

# Configuration
MASTER_HOST="10.0.1.10"
SLAVE_HOST="10.0.1.11"
REPL_USER="replicator"
REPL_PASS="replication_password_123"
DB_ROOT_PASS="root_password"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Setup master server
setup_master() {
    log "Setting up MySQL master server"
    
    # Configure my.cnf
    cat << EOF >> /etc/mysql/my.cnf
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
max_binlog_size = 100M
expire_logs_days = 7
EOF
    
    # Restart MySQL
    systemctl restart mysql
    
    # Create replication user
    mysql -u root -p"$DB_ROOT_PASS" << EOF
CREATE USER '$REPL_USER'@'%' IDENTIFIED BY '$REPL_PASS';
GRANT REPLICATION SLAVE ON *.* TO '$REPL_USER'@'%';
FLUSH PRIVILEGES;
EOF
    
    # Get master status
    mysql -u root -p"$DB_ROOT_PASS" -e "SHOW MASTER STATUS\G" > /tmp/master_status.txt
    
    log "Master server setup completed"
    log "Master status saved to /tmp/master_status.txt"
}

# Setup slave server
setup_slave() {
    local master_log_file="$1"
    local master_log_pos="$2"
    
    if [ -z "$master_log_file" ] || [ -z "$master_log_pos" ]; then
        log "ERROR: Master log file and position required"
        log "Usage: $0 setup-slave <master_log_file> <master_log_pos>"
        exit 1
    fi
    
    log "Setting up MySQL slave server"
    
    # Configure my.cnf
    cat << EOF >> /etc/mysql/my.cnf
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
log_slave_updates = 1
read_only = 1
EOF
    
    # Restart MySQL
    systemctl restart mysql
    
    # Configure slave
    mysql -u root -p"$DB_ROOT_PASS" << EOF
CHANGE MASTER TO
    MASTER_HOST='$MASTER_HOST',
    MASTER_USER='$REPL_USER',
    MASTER_PASSWORD='$REPL_PASS',
    MASTER_LOG_FILE='$master_log_file',
    MASTER_LOG_POS=$master_log_pos;

START SLAVE;
EOF
    
    log "Slave server setup completed"
}

# Monitor replication status
monitor_replication() {
    log "Monitoring MySQL replication status"
    
    # On master server
    echo "=== Master Server Status ==="
    mysql -u root -p"$DB_ROOT_PASS" -e "SHOW MASTER STATUS\G"
    mysql -u root -p"$DB_ROOT_PASS" -e "SHOW SLAVE HOSTS\G"
    
    # On slave server
    echo "\n=== Slave Server Status ==="
    mysql -u root -p"$DB_ROOT_PASS" -e "SHOW SLAVE STATUS\G"
}

# Execute based on parameter
case "$1" in
    "setup-master")
        setup_master
        ;;
    "setup-slave")
        setup_slave "$2" "$3"
        ;;
    "monitor")
        monitor_replication
        ;;
    *)
        echo "Usage: $0 {setup-master|setup-slave|monitor}"
        exit 1
        ;;
esac
```

---

## Compliance and Auditing

### GDPR Compliance Implementation
```sql
-- GDPR compliance features

-- 1. Data retention policies
CREATE TABLE data_retention_policies (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    retention_period_months INTEGER NOT NULL,
    deletion_criteria TEXT,
    last_cleanup TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert retention policies
INSERT INTO data_retention_policies (table_name, retention_period_months, deletion_criteria) VALUES
('user_sessions', 1, 'created_at < CURRENT_DATE - INTERVAL ''1 month'''),
('audit_logs', 84, 'created_at < CURRENT_DATE - INTERVAL ''7 years'''),
('user_activity_logs', 24, 'created_at < CURRENT_DATE - INTERVAL ''2 years''');

-- 2. Data anonymization function
CREATE OR REPLACE FUNCTION anonymize_user_data(user_id_param INTEGER)
RETURNS VOID AS $$
BEGIN
    -- Anonymize personal data
    UPDATE users 
    SET 
        email = 'anonymized_' || user_id_param || '@deleted.com',
        first_name = 'Anonymized',
        last_name = 'User',
        phone = NULL,
        address = NULL,
        date_of_birth = NULL,
        anonymized_at = CURRENT_TIMESTAMP
    WHERE id = user_id_param;
    
    -- Log the anonymization
    INSERT INTO gdpr_actions (user_id, action_type, performed_at, performed_by)
    VALUES (user_id_param, 'ANONYMIZE', CURRENT_TIMESTAMP, 'system');
    
    -- Remove from marketing lists
    DELETE FROM marketing_subscriptions WHERE user_id = user_id_param;
    
    RAISE NOTICE 'User % data anonymized successfully', user_id_param;
END;
$$ LANGUAGE plpgsql;

-- 3. Data export function for data portability
CREATE OR REPLACE FUNCTION export_user_data(user_id_param INTEGER)
RETURNS JSON AS $$
DECLARE
    user_data JSON;
BEGIN
    SELECT json_build_object(
        'personal_info', (
            SELECT json_build_object(
                'id', id,
                'email', email,
                'first_name', first_name,
                'last_name', last_name,
                'phone', phone,
                'address', address,
                'registration_date', registration_date
            )
            FROM users WHERE id = user_id_param
        ),
        'orders', (
            SELECT json_agg(
                json_build_object(
                    'order_id', id,
                    'order_date', order_date,
                    'total_amount', total_amount,
                    'status', status
                )
            )
            FROM orders WHERE customer_id = user_id_param
        ),
        'preferences', (
            SELECT json_agg(
                json_build_object(
                    'preference_type', preference_type,
                    'preference_value', preference_value,
                    'updated_at', updated_at
                )
            )
            FROM user_preferences WHERE user_id = user_id_param
        )
    ) INTO user_data;
    
    -- Log the export
    INSERT INTO gdpr_actions (user_id, action_type, performed_at, performed_by)
    VALUES (user_id_param, 'EXPORT', CURRENT_TIMESTAMP, 'system');
    
    RETURN user_data;
END;
$$ LANGUAGE plpgsql;

-- 4. Automated data cleanup procedure
CREATE OR REPLACE FUNCTION cleanup_expired_data()
RETURNS VOID AS $$
DECLARE
    policy RECORD;
    cleanup_sql TEXT;
    deleted_count INTEGER;
BEGIN
    FOR policy IN SELECT * FROM data_retention_policies LOOP
        -- Build dynamic SQL for cleanup
        cleanup_sql := format('DELETE FROM %I WHERE %s', 
                             policy.table_name, 
                             policy.deletion_criteria);
        
        -- Execute cleanup
        EXECUTE cleanup_sql;
        GET DIAGNOSTICS deleted_count = ROW_COUNT;
        
        -- Update last cleanup timestamp
        UPDATE data_retention_policies 
        SET last_cleanup = CURRENT_TIMESTAMP 
        WHERE id = policy.id;
        
        -- Log cleanup activity
        INSERT INTO cleanup_log (table_name, deleted_count, cleanup_date)
        VALUES (policy.table_name, deleted_count, CURRENT_TIMESTAMP);
        
        RAISE NOTICE 'Cleaned up % records from table %', deleted_count, policy.table_name;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

### Audit Trail Implementation
```sql
-- Comprehensive audit trail system

-- 1. Audit log table
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    operation VARCHAR(10) NOT NULL, -- INSERT, UPDATE, DELETE
    record_id VARCHAR(100),
    old_values JSONB,
    new_values JSONB,
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    session_id VARCHAR(100),
    ip_address INET,
    user_agent TEXT
);

-- Create indexes for performance
CREATE INDEX idx_audit_log_table_operation ON audit_log (table_name, operation);
CREATE INDEX idx_audit_log_changed_at ON audit_log (changed_at);
CREATE INDEX idx_audit_log_changed_by ON audit_log (changed_by);

-- 2. Generic audit trigger function
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
DECLARE
    old_data JSONB;
    new_data JSONB;
    changed_by_user VARCHAR(100);
    session_id_val VARCHAR(100);
    ip_addr INET;
    user_agent_val TEXT;
BEGIN
    -- Get session information
    changed_by_user := COALESCE(current_setting('audit.user_id', true), 'system');
    session_id_val := COALESCE(current_setting('audit.session_id', true), '');
    ip_addr := COALESCE(current_setting('audit.ip_address', true)::INET, NULL);
    user_agent_val := COALESCE(current_setting('audit.user_agent', true), '');
    
    -- Handle different operations
    IF TG_OP = 'DELETE' THEN
        old_data := to_jsonb(OLD);
        new_data := NULL;
        
        INSERT INTO audit_log (
            table_name, operation, record_id, old_values, new_values,
            changed_by, session_id, ip_address, user_agent
        ) VALUES (
            TG_TABLE_NAME, TG_OP, OLD.id::TEXT, old_data, new_data,
            changed_by_user, session_id_val, ip_addr, user_agent_val
        );
        
        RETURN OLD;
        
    ELSIF TG_OP = 'UPDATE' THEN
        old_data := to_jsonb(OLD);
        new_data := to_jsonb(NEW);
        
        -- Only log if data actually changed
        IF old_data != new_data THEN
            INSERT INTO audit_log (
                table_name, operation, record_id, old_values, new_values,
                changed_by, session_id, ip_address, user_agent
            ) VALUES (
                TG_TABLE_NAME, TG_OP, NEW.id::TEXT, old_data, new_data,
                changed_by_user, session_id_val, ip_addr, user_agent_val
            );
        END IF;
        
        RETURN NEW;
        
    ELSIF TG_OP = 'INSERT' THEN
        old_data := NULL;
        new_data := to_jsonb(NEW);
        
        INSERT INTO audit_log (
            table_name, operation, record_id, old_values, new_values,
            changed_by, session_id, ip_address, user_agent
        ) VALUES (
            TG_TABLE_NAME, TG_OP, NEW.id::TEXT, old_data, new_data,
            changed_by_user, session_id_val, ip_addr, user_agent_val
        );
        
        RETURN NEW;
    END IF;
    
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- 3. Apply audit triggers to tables
CREATE TRIGGER users_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER orders_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON orders
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

CREATE TRIGGER products_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON products
    FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

-- 4. Audit query functions
CREATE OR REPLACE FUNCTION get_audit_trail(
    p_table_name VARCHAR DEFAULT NULL,
    p_record_id VARCHAR DEFAULT NULL,
    p_start_date TIMESTAMP DEFAULT NULL,
    p_end_date TIMESTAMP DEFAULT NULL,
    p_limit INTEGER DEFAULT 100
) RETURNS TABLE (
    id BIGINT,
    table_name VARCHAR,
    operation VARCHAR,
    record_id VARCHAR,
    changed_by VARCHAR,
    changed_at TIMESTAMP,
    changes JSONB
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        al.id,
        al.table_name,
        al.operation,
        al.record_id,
        al.changed_by,
        al.changed_at,
        CASE 
            WHEN al.operation = 'UPDATE' THEN
                jsonb_build_object(
                    'old', al.old_values,
                    'new', al.new_values,
                    'diff', jsonb_diff(al.old_values, al.new_values)
                )
            WHEN al.operation = 'INSERT' THEN
                jsonb_build_object('new', al.new_values)
            WHEN al.operation = 'DELETE' THEN
                jsonb_build_object('deleted', al.old_values)
        END as changes
    FROM audit_log al
    WHERE 
        (p_table_name IS NULL OR al.table_name = p_table_name)
        AND (p_record_id IS NULL OR al.record_id = p_record_id)
        AND (p_start_date IS NULL OR al.changed_at >= p_start_date)
        AND (p_end_date IS NULL OR al.changed_at <= p_end_date)
    ORDER BY al.changed_at DESC
    LIMIT p_limit;
END;
$$ LANGUAGE plpgsql;

-- Helper function to compare JSONB objects
CREATE OR REPLACE FUNCTION jsonb_diff(old_data JSONB, new_data JSONB)
RETURNS JSONB AS $$
DECLARE
    result JSONB := '{}'::JSONB;
    key TEXT;
    old_val JSONB;
    new_val JSONB;
BEGIN
    -- Check for changed and new keys
    FOR key IN SELECT jsonb_object_keys(new_data) LOOP
        old_val := old_data -> key;
        new_val := new_data -> key;
        
        IF old_val IS DISTINCT FROM new_val THEN
            result := result || jsonb_build_object(key, jsonb_build_object(
                'old', old_val,
                'new', new_val
            ));
        END IF;
    END LOOP;
    
    -- Check for deleted keys
    FOR key IN SELECT jsonb_object_keys(old_data) LOOP
        IF NOT new_data ? key THEN
            result := result || jsonb_build_object(key, jsonb_build_object(
                'old', old_data -> key,
                'new', null
            ));
        END IF;
    END LOOP;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql;
```

---

## Hands-on Examples

### Example 1: Complete Database Security Setup
```sql
-- Comprehensive security implementation for e-commerce database

-- 1. Create security roles
CREATE ROLE ecommerce_admin;
CREATE ROLE ecommerce_app;
CREATE ROLE ecommerce_readonly;
CREATE ROLE ecommerce_reporting;

-- 2. Grant appropriate permissions
-- Admin role (full access)
GRANT ALL PRIVILEGES ON DATABASE ecommerce TO ecommerce_admin;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO ecommerce_admin;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO ecommerce_admin;

-- Application role (CRUD operations)
GRANT CONNECT ON DATABASE ecommerce TO ecommerce_app;
GRANT USAGE ON SCHEMA public TO ecommerce_app;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO ecommerce_app;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO ecommerce_app;

-- Read-only role
GRANT CONNECT ON DATABASE ecommerce TO ecommerce_readonly;
GRANT USAGE ON SCHEMA public TO ecommerce_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ecommerce_readonly;

-- Reporting role (with specific restrictions)
GRANT ecommerce_readonly TO ecommerce_reporting;
REVOKE SELECT ON customer_sensitive FROM ecommerce_reporting;

-- 3. Create users and assign roles
CREATE USER app_prod WITH PASSWORD 'prod_app_pass_2024!';
GRANT ecommerce_app TO app_prod;

CREATE USER app_staging WITH PASSWORD 'staging_app_pass_2024!';
GRANT ecommerce_app TO app_staging;

CREATE USER reporting_user WITH PASSWORD 'reporting_pass_2024!';
GRANT ecommerce_reporting TO reporting_user;

CREATE USER dba_user WITH PASSWORD 'dba_pass_2024!' CREATEDB;
GRANT ecommerce_admin TO dba_user;

-- 4. Implement row-level security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE customer_data ENABLE ROW LEVEL SECURITY;

-- Customer can only see their own orders
CREATE POLICY customer_orders_policy ON orders
    FOR ALL TO ecommerce_app
    USING (customer_id = current_setting('app.current_customer_id')::INTEGER);

-- Admin can see all orders
CREATE POLICY admin_orders_policy ON orders
    FOR ALL TO ecommerce_admin
    USING (true);

-- Reporting can only see orders from last 2 years
CREATE POLICY reporting_orders_policy ON orders
    FOR SELECT TO ecommerce_reporting
    USING (order_date >= CURRENT_DATE - INTERVAL '2 years');
```

### Example 2: Automated Backup and Recovery System
```bash
#!/bin/bash
# enterprise-backup-system.sh - Enterprise-grade backup solution

# Configuration
BACKUP_CONFIG="/etc/db-backup/config.conf"
source "$BACKUP_CONFIG"

# Logging setup
LOG_DIR="/var/log/db-backup"
LOG_FILE="$LOG_DIR/backup-$(date +%Y%m%d).log"
mkdir -p "$LOG_DIR"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Notification function
send_notification() {
    local status="$1"
    local message="$2"
    
    # Send to Slack
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"Database Backup $status: $message\"}" \
        "$SLACK_WEBHOOK_URL"
    
    # Send email
    echo "$message" | mail -s "Database Backup $status" "$ADMIN_EMAIL"
}

# Pre-backup checks
perform_pre_backup_checks() {
    log "Performing pre-backup checks"
    
    # Check disk space
    AVAILABLE_SPACE=$(df "$BACKUP_DIR" | awk 'NR==2 {print $4}')
    REQUIRED_SPACE=10485760  # 10GB in KB
    
    if [ "$AVAILABLE_SPACE" -lt "$REQUIRED_SPACE" ]; then
        log "ERROR: Insufficient disk space. Available: ${AVAILABLE_SPACE}KB, Required: ${REQUIRED_SPACE}KB"
        send_notification "FAILED" "Insufficient disk space for backup"
        exit 1
    fi
    
    # Check database connectivity
    if ! pg_isready -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER"; then
        log "ERROR: Cannot connect to database"
        send_notification "FAILED" "Database connection failed"
        exit 1
    fi
    
    # Check backup directory permissions
    if [ ! -w "$BACKUP_DIR" ]; then
        log "ERROR: Backup directory not writable"
        send_notification "FAILED" "Backup directory permission error"
        exit 1
    fi
    
    log "Pre-backup checks completed successfully"
}

# Perform backup with verification
perform_verified_backup() {
    local backup_type="$1"
    local backup_file="$BACKUP_DIR/${backup_type}_backup_${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql.gz"
    
    log "Starting $backup_type backup: $backup_file"
    
    # Create backup
    if [ "$backup_type" = "full" ]; then
        pg_dump -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" \
            --verbose --no-password --format=custom --compress=9 \
            --file="${backup_file%.gz}" \
            --exclude-table-data='audit_logs' \
            --exclude-table-data='session_data' 2>&1 | tee -a "$LOG_FILE"
    else
        # Incremental backup logic here
        log "Incremental backup not implemented in this example"
        return 1
    fi
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        gzip "${backup_file%.gz}"
        log "Backup created successfully: $backup_file"
        
        # Verify backup integrity
        verify_backup_integrity "$backup_file"
        
        # Upload to cloud storage
        upload_to_cloud "$backup_file"
        
        # Update backup catalog
        update_backup_catalog "$backup_file" "$backup_type"
        
        send_notification "SUCCESS" "Backup completed: $(basename $backup_file)"
    else
        log "ERROR: Backup failed"
        send_notification "FAILED" "Backup creation failed"
        exit 1
    fi
}

# Verify backup integrity
verify_backup_integrity() {
    local backup_file="$1"
    local test_db="backup_test_$(date +%s)"
    
    log "Verifying backup integrity: $backup_file"
    
    # Create test database
    createdb -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" "$test_db" 2>&1 | tee -a "$LOG_FILE"
    
    # Restore to test database
    gunzip -c "$backup_file" | pg_restore -h "$DB_HOST" -p "$DB_PORT" \
        -U "$DB_USER" -d "$test_db" --verbose 2>&1 | tee -a "$LOG_FILE"
    
    if [ ${PIPESTATUS[1]} -eq 0 ]; then
        # Run integrity checks
        psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$test_db" \
            -c "SELECT COUNT(*) as table_count FROM information_schema.tables WHERE table_schema = 'public';" \
            2>&1 | tee -a "$LOG_FILE"
        
        log "Backup verification successful"
    else
        log "ERROR: Backup verification failed"
        send_notification "FAILED" "Backup verification failed"
    fi
    
    # Cleanup test database
    dropdb -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" "$test_db" 2>&1 | tee -a "$LOG_FILE"
}

# Upload to cloud storage
upload_to_cloud() {
    local backup_file="$1"
    
    log "Uploading backup to cloud storage"
    
    # Upload to AWS S3
    aws s3 cp "$backup_file" "s3://$S3_BUCKET/backups/$(date +%Y/%m/%d)/" \
        --storage-class STANDARD_IA 2>&1 | tee -a "$LOG_FILE"
    
    if [ $? -eq 0 ]; then
        log "Cloud upload successful"
    else
        log "WARNING: Cloud upload failed"
        send_notification "WARNING" "Cloud upload failed for $(basename $backup_file)"
    fi
}

# Update backup catalog
update_backup_catalog() {
    local backup_file="$1"
    local backup_type="$2"
    local file_size=$(stat -c%s "$backup_file")
    
    psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME" << EOF
INSERT INTO backup_catalog (
    backup_file, backup_type, file_size, backup_date, 
    verification_status, cloud_uploaded
) VALUES (
    '$(basename $backup_file)', '$backup_type', $file_size, 
    CURRENT_TIMESTAMP, 'verified', true
);
EOF
}

# Main execution
log "Starting enterprise backup system"

perform_pre_backup_checks
perform_verified_backup "full"

log "Backup system completed successfully"
```

---

## Best Practices

### Security Best Practices
1. **Principle of Least Privilege**: Grant minimum necessary permissions
2. **Regular Security Audits**: Review user permissions quarterly
3. **Strong Password Policies**: Enforce complex passwords and rotation
4. **Network Security**: Use SSL/TLS for all connections
5. **Data Encryption**: Encrypt sensitive data at rest and in transit
6. **Regular Updates**: Keep database software updated with security patches
7. **Backup Encryption**: Encrypt all backup files
8. **Access Logging**: Log all database access and changes

### Performance Best Practices
1. **Regular Maintenance**: Schedule VACUUM, ANALYZE, and REINDEX operations
2. **Monitor Query Performance**: Use pg_stat_statements and slow query logs
3. **Index Optimization**: Regularly review and optimize indexes
4. **Connection Pooling**: Use connection pooling for better resource management
5. **Resource Monitoring**: Monitor CPU, memory, and disk usage
6. **Query Optimization**: Regularly review and optimize slow queries
7. **Partitioning**: Use table partitioning for large tables
8. **Archive Old Data**: Implement data archiving strategies

### Backup and Recovery Best Practices
1. **3-2-1 Rule**: 3 copies, 2 different media, 1 offsite
2. **Regular Testing**: Test backup restoration procedures monthly
3. **Automated Verification**: Verify backup integrity automatically
4. **Documentation**: Maintain detailed recovery procedures
5. **RTO/RPO Planning**: Define and test recovery time/point objectives
6. **Multiple Backup Types**: Use full, incremental, and differential backups
7. **Secure Storage**: Encrypt and secure backup storage
8. **Retention Policies**: Implement appropriate backup retention policies

---

## Common Mistakes

1. **Insufficient Testing**: Not testing backup and recovery procedures
2. **Weak Security**: Using default passwords and excessive permissions
3. **Poor Monitoring**: Not monitoring database performance and health
4. **Inadequate Documentation**: Missing or outdated procedures
5. **Ignoring Logs**: Not reviewing database and system logs regularly
6. **No Disaster Recovery Plan**: Lacking comprehensive DR procedures
7. **Improper Maintenance**: Skipping regular maintenance tasks
8. **Security Oversights**: Not implementing proper audit trails
9. **Resource Planning**: Inadequate capacity planning
10. **Compliance Gaps**: Not meeting regulatory requirements

---

## Practice Projects

### Project 1: Enterprise Database Security Implementation
**Objective**: Implement comprehensive security for a multi-tenant application

**Requirements**:
- Set up role-based access control with 5 different roles
- Implement row-level security for tenant isolation
- Create audit trail for all data changes
- Implement data encryption for sensitive fields
- Set up automated security monitoring

**Deliverables**:
- Security configuration scripts
- User management procedures
- Audit trail reports
- Security monitoring dashboard

### Project 2: High Availability Database Cluster
**Objective**: Set up a highly available PostgreSQL cluster

**Requirements**:
- Configure master-slave replication
- Implement automatic failover
- Set up load balancing for read queries
- Create monitoring and alerting
- Document failover procedures

**Deliverables**:
- Cluster configuration scripts
- Failover automation
- Monitoring setup
- Operational procedures

### Project 3: Comprehensive Backup and Recovery System
**Objective**: Design and implement enterprise backup strategy

**Requirements**:
- Multiple backup types (full, incremental, differential)
- Automated backup verification
- Cloud storage integration
- Point-in-time recovery capability
- Disaster recovery procedures

**Deliverables**:
- Backup automation scripts
- Recovery procedures
- Testing documentation
- Disaster recovery plan

---

## Related Levels
- **Level 1**: [Query Execution Fundamentals](./level-1.md) - Basic SQL operations
- **Level 2**: [Advanced Query Writing](./level-2.md) - Complex queries and optimization
- **Related Topics**: Docker, System Administration, Cloud Platforms

---

## Q&A Section

**Q: How often should I perform database maintenance tasks?**
A: Daily for critical monitoring, weekly for routine maintenance (VACUUM, ANALYZE), monthly for comprehensive reviews (index optimization, security audits), and quarterly for major updates and capacity planning.

**Q: What's the difference between logical and physical backups?**
A: Logical backups (pg_dump, mysqldump) export data as SQL statements - portable but slower to restore. Physical backups copy actual data files - faster but platform-specific. Use logical for smaller databases and cross-platform compatibility, physical for large databases and faster recovery.

**Q: How do I determine the right backup retention policy?**
A: Consider: business requirements (how far back you need to recover), compliance regulations (legal retention periods), storage costs, and recovery scenarios. Common approach: daily backups for 30 days, weekly for 12 weeks, monthly for 12 months, yearly for regulatory period.

**Q: What should I monitor in a production database?**
A: Key metrics include: connection count, query performance, lock waits, disk space, memory usage, replication lag, backup status, security events, and error rates. Set up alerts for thresholds that indicate potential issues.

**Q: How do I handle database schema changes in production?**
A: Use migration scripts, test in staging environment, plan for rollback, consider downtime requirements, use online schema change tools for large tables, and maintain version control for all schema changes.

**Q: What's the best approach for database security in cloud environments?**
A: Use cloud provider security features (VPC, security groups), enable encryption at rest and in transit, implement IAM integration, use managed database services when possible, regular security assessments, and follow cloud security best practices.

**Q: How do I troubleshoot slow database performance?**
A: Start with: identifying slow queries (pg_stat_statements), checking for lock contention, analyzing query execution plans, reviewing index usage, monitoring system resources, and checking for table bloat. Use systematic approach to isolate the root cause.

**Q: What's the difference between VACUUM and VACUUM FULL in PostgreSQL?**
A: VACUUM reclaims space from deleted rows but doesn't return it to the OS - it's online and safe for production. VACUUM FULL rebuilds the entire table, returns space to OS, but requires exclusive lock and significant downtime - use sparingly and during maintenance windows.