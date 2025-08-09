# PostgreSQL/PostGIS Level 3: Database Programming and Administration

## Prerequisites
- Completed PostgreSQL/PostGIS Level 1 and Level 2
- Strong understanding of SQL and spatial operations
- Experience with database design and optimization
- Basic understanding of system administration concepts

## What Problem Does This Solve?
This level addresses advanced database programming and basic administration challenges:
- Creating sophisticated database objects (sequences, triggers, stored procedures, functions)
- Implementing automated spatial data processing workflows
- Performing database installation, configuration, and maintenance
- Managing backup and restore operations for spatial databases

## Key Concepts

### 1. Create Advanced Geodatabase Elements
- **Sequences**: Custom sequence generators for spatial data
- **Triggers**: Automated spatial data validation and processing
- **Stored Procedures**: Complex spatial analysis workflows
- **Functions**: Reusable spatial operations and calculations

### 2. Perform Basic Installation
- **PostgreSQL Installation**: Setting up PostgreSQL server
- **PostGIS Extension**: Installing and configuring PostGIS
- **Configuration Management**: Optimizing settings for spatial workloads
- **User Management**: Creating roles and permissions for spatial databases

### 3. Perform Backup / Restore
- **Logical Backups**: Using pg_dump for database backups
- **Physical Backups**: File system level backup strategies
- **Point-in-Time Recovery**: Setting up WAL archiving
- **Spatial Data Considerations**: Handling large spatial datasets in backups

### 4. Database Maintenance and Optimization
- **Vacuum and Analyze**: Maintaining spatial indexes and statistics
- **Reindexing**: Rebuilding spatial indexes for performance
- **Monitoring**: Tracking spatial query performance
- **Maintenance Automation**: Scheduling routine maintenance tasks

## Hands-on Examples

### Example 1: Advanced Database Objects - Sequences and Functions
```sql
-- Create custom sequence for parcel numbering
CREATE SEQUENCE parcel_number_seq
    START WITH 100000
    INCREMENT BY 1
    MINVALUE 100000
    MAXVALUE 999999
    CACHE 10;

-- Function to generate formatted parcel numbers
CREATE OR REPLACE FUNCTION generate_parcel_number(district_code VARCHAR(3))
RETURNS VARCHAR(20) AS $$
DECLARE
    next_num INTEGER;
    formatted_number VARCHAR(20);
BEGIN
    next_num := nextval('parcel_number_seq');
    formatted_number := district_code || '-' || LPAD(next_num::TEXT, 6, '0');
    RETURN formatted_number;
END;
$$ LANGUAGE plpgsql;

-- Advanced spatial function for land use analysis
CREATE OR REPLACE FUNCTION calculate_land_use_density(
    analysis_geometry GEOMETRY,
    target_land_use VARCHAR(50)
)
RETURNS TABLE(
    total_area_sqm NUMERIC,
    land_use_area_sqm NUMERIC,
    density_percentage NUMERIC,
    parcel_count INTEGER
) AS $$
BEGIN
    RETURN QUERY
    WITH analysis_area AS (
        SELECT ST_Area(ST_Transform(analysis_geometry, 3857)) as total_area
    ),
    intersecting_parcels AS (
        SELECT 
            lp.parcel_id,
            lp.land_use,
            ST_Intersection(
                ST_Transform(lp.geometry, 3857),
                ST_Transform(analysis_geometry, 3857)
            ) as intersection_geom
        FROM land_parcels lp
        WHERE ST_Intersects(lp.geometry, analysis_geometry)
    ),
    land_use_stats AS (
        SELECT 
            SUM(CASE 
                WHEN ip.land_use = target_land_use 
                THEN ST_Area(ip.intersection_geom) 
                ELSE 0 
            END) as target_area,
            COUNT(CASE 
                WHEN ip.land_use = target_land_use 
                THEN 1 
            END) as target_count
        FROM intersecting_parcels ip
    )
    SELECT 
        aa.total_area,
        COALESCE(lus.target_area, 0),
        CASE 
            WHEN aa.total_area > 0 
            THEN ROUND((COALESCE(lus.target_area, 0) / aa.total_area) * 100, 2)
            ELSE 0 
        END,
        COALESCE(lus.target_count, 0)::INTEGER
    FROM analysis_area aa
    CROSS JOIN land_use_stats lus;
END;
$$ LANGUAGE plpgsql;
```

### Example 2: Triggers for Spatial Data Validation
```sql
-- Trigger function for automatic geometry validation and area calculation
CREATE OR REPLACE FUNCTION validate_and_update_parcel()
RETURNS TRIGGER AS $$
BEGIN
    -- Validate geometry
    IF NOT ST_IsValid(NEW.geometry) THEN
        RAISE EXCEPTION 'Invalid geometry provided for parcel %', NEW.parcel_number;
    END IF;
    
    -- Ensure geometry is not empty
    IF ST_IsEmpty(NEW.geometry) THEN
        RAISE EXCEPTION 'Empty geometry not allowed for parcel %', NEW.parcel_number;
    END IF;
    
    -- Auto-calculate area if not provided
    IF NEW.area_sqm IS NULL OR NEW.area_sqm = 0 THEN
        NEW.area_sqm := ST_Area(ST_Transform(NEW.geometry, 3857));
    END IF;
    
    -- Validate area consistency (within 5% tolerance)
    IF ABS(NEW.area_sqm - ST_Area(ST_Transform(NEW.geometry, 3857))) > (NEW.area_sqm * 0.05) THEN
        RAISE WARNING 'Area inconsistency detected for parcel %. Calculated: %, Provided: %', 
            NEW.parcel_number, 
            ST_Area(ST_Transform(NEW.geometry, 3857)), 
            NEW.area_sqm;
    END IF;
    
    -- Auto-generate parcel number if not provided
    IF NEW.parcel_number IS NULL THEN
        NEW.parcel_number := generate_parcel_number('GEN');
    END IF;
    
    -- Set updated timestamp
    NEW.updated_at := NOW();
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER trigger_validate_parcel
    BEFORE INSERT OR UPDATE ON land_parcels
    FOR EACH ROW
    EXECUTE FUNCTION validate_and_update_parcel();

-- Audit trigger for tracking changes
CREATE TABLE parcel_audit (
    audit_id SERIAL PRIMARY KEY,
    parcel_id INTEGER,
    operation VARCHAR(10),
    old_data JSONB,
    new_data JSONB,
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_parcel_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO parcel_audit (parcel_id, operation, old_data, changed_by)
        VALUES (OLD.parcel_id, 'DELETE', row_to_json(OLD), current_user);
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO parcel_audit (parcel_id, operation, old_data, new_data, changed_by)
        VALUES (NEW.parcel_id, 'UPDATE', row_to_json(OLD), row_to_json(NEW), current_user);
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO parcel_audit (parcel_id, operation, new_data, changed_by)
        VALUES (NEW.parcel_id, 'INSERT', row_to_json(NEW), current_user);
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_audit_parcels
    AFTER INSERT OR UPDATE OR DELETE ON land_parcels
    FOR EACH ROW
    EXECUTE FUNCTION audit_parcel_changes();
```

### Example 3: Stored Procedures for Complex Workflows
```sql
-- Stored procedure for comprehensive spatial analysis
CREATE OR REPLACE PROCEDURE analyze_development_impact(
    IN development_geometry GEOMETRY,
    IN analysis_radius NUMERIC DEFAULT 1000,
    OUT impact_summary JSONB
)
LANGUAGE plpgsql AS $$
DECLARE
    analysis_buffer GEOMETRY;
    affected_parcels INTEGER;
    affected_area NUMERIC;
    infrastructure_impact JSONB;
    environmental_impact JSONB;
BEGIN
    -- Create analysis buffer
    analysis_buffer := ST_Buffer(ST_Transform(development_geometry, 3857), analysis_radius);
    
    -- Analyze affected parcels
    SELECT 
        COUNT(*),
        SUM(ST_Area(ST_Intersection(
            ST_Transform(lp.geometry, 3857),
            analysis_buffer
        )))
    INTO affected_parcels, affected_area
    FROM land_parcels lp
    WHERE ST_Intersects(ST_Transform(lp.geometry, 3857), analysis_buffer);
    
    -- Analyze infrastructure impact
    WITH road_analysis AS (
        SELECT 
            COUNT(*) as affected_roads,
            SUM(ST_Length(ST_Intersection(
                ST_Transform(r.geometry, 3857),
                analysis_buffer
            ))) as affected_road_length
        FROM roads r
        WHERE ST_Intersects(ST_Transform(r.geometry, 3857), analysis_buffer)
    ),
    utility_analysis AS (
        SELECT 
            COUNT(*) as affected_utilities
        FROM utilities u
        WHERE ST_Intersects(ST_Transform(u.geometry, 3857), analysis_buffer)
    )
    SELECT jsonb_build_object(
        'affected_roads', ra.affected_roads,
        'affected_road_length_m', ROUND(ra.affected_road_length, 2),
        'affected_utilities', ua.affected_utilities
    )
    INTO infrastructure_impact
    FROM road_analysis ra
    CROSS JOIN utility_analysis ua;
    
    -- Analyze environmental impact
    WITH env_analysis AS (
        SELECT 
            SUM(CASE WHEN eb.protection_level = 'High' 
                THEN ST_Area(ST_Intersection(
                    ST_Transform(eb.geometry, 3857),
                    analysis_buffer
                )) ELSE 0 END) as high_protection_area,
            SUM(CASE WHEN eb.protection_level = 'Medium' 
                THEN ST_Area(ST_Intersection(
                    ST_Transform(eb.geometry, 3857),
                    analysis_buffer
                )) ELSE 0 END) as medium_protection_area
        FROM environmental_boundaries eb
        WHERE ST_Intersects(ST_Transform(eb.geometry, 3857), analysis_buffer)
    )
    SELECT jsonb_build_object(
        'high_protection_area_sqm', ROUND(COALESCE(ea.high_protection_area, 0), 2),
        'medium_protection_area_sqm', ROUND(COALESCE(ea.medium_protection_area, 0), 2)
    )
    INTO environmental_impact
    FROM env_analysis ea;
    
    -- Compile final summary
    impact_summary := jsonb_build_object(
        'analysis_radius_m', analysis_radius,
        'affected_parcels', affected_parcels,
        'affected_area_sqm', ROUND(COALESCE(affected_area, 0), 2),
        'infrastructure_impact', infrastructure_impact,
        'environmental_impact', environmental_impact,
        'analysis_timestamp', NOW()
    );
    
    -- Log the analysis
    INSERT INTO analysis_log (analysis_type, parameters, results)
    VALUES ('development_impact', 
            jsonb_build_object('radius', analysis_radius, 'geometry', ST_AsText(development_geometry)),
            impact_summary);
END;
$$;
```

### Example 4: Installation and Configuration Scripts
```bash
#!/bin/bash
# PostgreSQL and PostGIS installation script for Ubuntu/Debian

# Update system packages
sudo apt update

# Install PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Install PostGIS
sudo apt install -y postgis postgresql-14-postgis-3

# Start and enable PostgreSQL service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create PostGIS database
sudo -u postgres createdb spatial_db
sudo -u postgres psql -d spatial_db -c "CREATE EXTENSION postgis;"
sudo -u postgres psql -d spatial_db -c "CREATE EXTENSION postgis_topology;"

# Create application user
sudo -u postgres createuser --interactive gis_user
sudo -u postgres psql -c "ALTER USER gis_user WITH PASSWORD 'secure_password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE spatial_db TO gis_user;"

echo "PostgreSQL and PostGIS installation completed!"
```

```sql
-- PostgreSQL configuration optimization for spatial workloads
-- Add to postgresql.conf

-- Memory settings
shared_buffers = '256MB'                    -- 25% of RAM for small systems
effective_cache_size = '1GB'                -- 75% of RAM
work_mem = '16MB'                           -- Per operation memory
maintenance_work_mem = '256MB'              -- For maintenance operations

-- Checkpoint settings
checkpoint_completion_target = 0.9
wal_buffers = '16MB'

-- Query planner settings
random_page_cost = 1.1                      -- For SSD storage
effective_io_concurrency = 200              -- For SSD storage

-- Logging settings
log_statement = 'mod'                       -- Log modifications
log_min_duration_statement = 1000           -- Log slow queries (1 second)
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

-- PostGIS specific settings
max_locks_per_transaction = 256             -- For complex spatial operations
```

### Example 5: Backup and Restore Procedures
```bash
#!/bin/bash
# Comprehensive backup script for PostGIS databases

DB_NAME="spatial_db"
BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.sql"
COMPRESSED_FILE="${BACKUP_FILE}.gz"

# Create backup directory if it doesn't exist
mkdir -p $BACKUP_DIR

# Perform logical backup
echo "Starting backup of $DB_NAME..."
pg_dump -h localhost -U postgres -d $DB_NAME \
    --verbose \
    --format=custom \
    --compress=9 \
    --file="${BACKUP_FILE}.backup"

# Also create plain SQL backup for portability
pg_dump -h localhost -U postgres -d $DB_NAME \
    --verbose \
    --format=plain \
    --file="$BACKUP_FILE"

# Compress plain SQL backup
gzip "$BACKUP_FILE"

# Verify backup integrity
echo "Verifying backup integrity..."
pg_restore --list "${BACKUP_FILE}.backup" > /dev/null

if [ $? -eq 0 ]; then
    echo "Backup completed successfully: ${BACKUP_FILE}.backup"
    echo "Compressed SQL backup: $COMPRESSED_FILE"
else
    echo "Backup verification failed!"
    exit 1
fi

# Clean up old backups (keep last 7 days)
find $BACKUP_DIR -name "${DB_NAME}_*.sql.gz" -mtime +7 -delete
find $BACKUP_DIR -name "${DB_NAME}_*.backup" -mtime +7 -delete

echo "Backup process completed."
```

```bash
#!/bin/bash
# Restore script for PostGIS databases

DB_NAME="spatial_db"
BACKUP_FILE="$1"
NEW_DB_NAME="${DB_NAME}_restored"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

if [ ! -f "$BACKUP_FILE" ]; then
    echo "Backup file not found: $BACKUP_FILE"
    exit 1
fi

# Create new database for restore
echo "Creating database $NEW_DB_NAME..."
createdb -h localhost -U postgres $NEW_DB_NAME

# Enable PostGIS extension
psql -h localhost -U postgres -d $NEW_DB_NAME -c "CREATE EXTENSION IF NOT EXISTS postgis;"
psql -h localhost -U postgres -d $NEW_DB_NAME -c "CREATE EXTENSION IF NOT EXISTS postgis_topology;"

# Restore from backup
echo "Restoring from $BACKUP_FILE..."
if [[ $BACKUP_FILE == *.backup ]]; then
    # Custom format restore
    pg_restore -h localhost -U postgres -d $NEW_DB_NAME \
        --verbose \
        --clean \
        --if-exists \
        "$BACKUP_FILE"
else
    # Plain SQL restore
    if [[ $BACKUP_FILE == *.gz ]]; then
        gunzip -c "$BACKUP_FILE" | psql -h localhost -U postgres -d $NEW_DB_NAME
    else
        psql -h localhost -U postgres -d $NEW_DB_NAME < "$BACKUP_FILE"
    fi
fi

if [ $? -eq 0 ]; then
    echo "Restore completed successfully to database: $NEW_DB_NAME"
else
    echo "Restore failed!"
    exit 1
fi
```

## Best Practices

### Database Programming
1. **Error Handling**: Implement comprehensive error handling in functions
2. **Performance**: Use appropriate data types and avoid unnecessary operations
3. **Security**: Validate inputs and use proper permissions
4. **Documentation**: Document function parameters and return values

### Installation and Configuration
1. **Security**: Change default passwords and configure proper authentication
2. **Performance**: Optimize configuration for spatial workloads
3. **Monitoring**: Set up logging and monitoring from the start
4. **Backup Strategy**: Implement automated backup procedures

### Maintenance
1. **Regular Maintenance**: Schedule VACUUM, ANALYZE, and REINDEX operations
2. **Monitoring**: Track performance metrics and query patterns
3. **Capacity Planning**: Monitor disk space and plan for growth
4. **Documentation**: Maintain documentation of procedures and configurations

## Common Mistakes to Avoid

1. **No Error Handling**: Not implementing proper error handling in functions
2. **Poor Performance**: Creating inefficient functions without considering performance
3. **Security Issues**: Not validating inputs or using proper permissions
4. **Backup Neglect**: Not implementing or testing backup procedures
5. **Configuration Defaults**: Using default PostgreSQL settings for spatial workloads
6. **No Monitoring**: Not setting up proper monitoring and alerting
7. **Trigger Complexity**: Creating overly complex triggers that impact performance
8. **No Testing**: Not testing functions and procedures thoroughly

## Advanced Practice Projects

### Project 1: Automated Spatial Data Processing Pipeline
**Objective**: Build a complete automated processing system
**Tasks**:
- Create functions for data validation and transformation
- Implement triggers for automated processing
- Build stored procedures for complex analysis workflows
- Set up automated backup and maintenance procedures
- Create monitoring and alerting systems

### Project 2: Multi-tenant Spatial Database System
**Objective**: Design a system supporting multiple clients
**Tasks**:
- Implement row-level security for spatial data
- Create functions for tenant-specific operations
- Build automated provisioning procedures
- Implement backup strategies for multi-tenant data
- Create performance monitoring per tenant

### Project 3: High-Availability Spatial Database
**Objective**: Set up a production-ready spatial database cluster
**Tasks**:
- Configure PostgreSQL streaming replication
- Implement automated failover procedures
- Set up point-in-time recovery
- Create comprehensive monitoring and alerting
- Build disaster recovery procedures

## Related Levels
- **Previous**: PostgreSQL/PostGIS Level 2 (Database Elements and Spatial SQL)
- **Next**: PostgreSQL/PostGIS Level 4 (Advanced Administration and Performance)
- **Related Topics**: 
  - System Administration
  - Database Performance Tuning
  - Backup and Recovery Strategies

## Q&A Section

### Basic Questions
**Q: When should I use stored procedures vs functions in PostGIS?**
A: Use functions for calculations and data transformations that return values. Use stored procedures for complex workflows that perform multiple operations and may not return values.

**Q: How do I optimize PostgreSQL configuration for spatial workloads?**
A: Increase shared_buffers, work_mem, and maintenance_work_mem. Adjust random_page_cost for SSD storage, and increase max_locks_per_transaction for complex spatial operations.

**Q: What's the best backup strategy for large spatial databases?**
A: Use pg_dump with custom format for smaller databases. For large databases, consider physical backups with WAL archiving for point-in-time recovery.

**Q: How do I handle errors in PostGIS functions?**
A: Use EXCEPTION blocks in PL/pgSQL functions, validate inputs, and provide meaningful error messages. Log errors for debugging and monitoring.

### Intermediate Questions
**Q: How do I implement efficient spatial triggers?**
A: Keep trigger logic simple and fast, use conditional logic to avoid unnecessary operations, consider using statement-level triggers for bulk operations, and monitor trigger performance.

**Q: What are the best practices for spatial function performance?**
A: Use spatial indexes effectively, avoid unnecessary coordinate transformations, use appropriate spatial predicates, and consider geometry simplification for complex operations.

**Q: How do I manage database schema changes in production?**
A: Use migration scripts, test changes in staging environments, implement rollback procedures, and coordinate with application deployments.

**Q: What monitoring should I implement for PostGIS databases?**
A: Monitor query performance, spatial index usage, disk space, connection counts, and backup success. Set up alerts for critical metrics.

---

*This level focuses on advanced database programming, basic administration tasks, and establishing robust operational procedures for PostGIS databases.*