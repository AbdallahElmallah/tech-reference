# PostgreSQL/PostGIS Level 2: Database Elements and Spatial SQL

## Prerequisites
- Completed PostgreSQL/PostGIS Level 1
- Solid understanding of SQL fundamentals
- Experience with basic PostGIS installation and connections
- Understanding of spatial data types and coordinate systems

## What Problem Does This Solve?
This level addresses intermediate challenges in spatial database management:
- Creating and managing complex geodatabase structures
- Performing advanced spatial queries and operations
- Implementing efficient data import/export workflows
- Building spatial views and managing spatial metadata

## Key Concepts

### 1. Create Geodatabase Elements (Tables, Views, and Spatial Views)
- **Spatial Tables**: Designing tables with geometry columns and constraints
- **Regular Views**: Creating views for data abstraction and security
- **Spatial Views**: Views that include spatial operations and transformations
- **Materialized Views**: Pre-computed spatial results for performance

### 2. Using SQL (DML) with a Geospatial Database
- **Spatial Data Manipulation**: INSERT, UPDATE, DELETE with spatial data
- **Spatial Joins**: Joining tables based on spatial relationships
- **Spatial Aggregations**: GROUP BY with spatial functions
- **Spatial Subqueries**: Complex nested spatial queries

### 3. Advanced Spatial Operations
- **Geometric Operations**: Buffer, intersection, union, difference
- **Spatial Analysis**: Distance calculations, area measurements
- **Coordinate Transformations**: Converting between different SRIDs
- **Topology Operations**: Spatial relationships and validations

### 4. Data Import/Export Workflows
- **Bulk Data Loading**: Efficient methods for large datasets
- **Format Conversions**: Working with different spatial data formats
- **Data Validation**: Ensuring data quality during import
- **Export Strategies**: Optimizing data export for different use cases

## Hands-on Examples

### Example 1: Creating Spatial Tables with Constraints
```sql
-- Create a comprehensive spatial table for land parcels
CREATE TABLE land_parcels (
    parcel_id SERIAL PRIMARY KEY,
    parcel_number VARCHAR(20) UNIQUE NOT NULL,
    owner_name VARCHAR(100) NOT NULL,
    land_use VARCHAR(50),
    area_sqm NUMERIC(12,2),
    assessed_value NUMERIC(15,2),
    geometry GEOMETRY(POLYGON, 4326) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Add spatial constraints
ALTER TABLE land_parcels 
ADD CONSTRAINT enforce_valid_geometry 
CHECK (ST_IsValid(geometry));

ALTER TABLE land_parcels 
ADD CONSTRAINT enforce_area_consistency 
CHECK (ABS(area_sqm - ST_Area(ST_Transform(geometry, 3857))) < 1.0);

-- Create spatial index
CREATE INDEX idx_land_parcels_geom 
ON land_parcels USING GIST(geometry);

-- Create additional indexes
CREATE INDEX idx_land_parcels_parcel_number 
ON land_parcels(parcel_number);

CREATE INDEX idx_land_parcels_land_use 
ON land_parcels(land_use);
```

### Example 2: Creating Spatial Views
```sql
-- Create a view for high-value commercial properties
CREATE VIEW commercial_high_value AS
SELECT 
    parcel_id,
    parcel_number,
    owner_name,
    assessed_value,
    area_sqm,
    assessed_value / area_sqm as value_per_sqm,
    ST_Centroid(geometry) as centroid,
    geometry
FROM land_parcels
WHERE land_use = 'Commercial' 
  AND assessed_value > 1000000;

-- Create a spatial view with buffer zones
CREATE VIEW parcel_buffer_zones AS
SELECT 
    parcel_id,
    parcel_number,
    'Original' as zone_type,
    0 as buffer_distance,
    geometry
FROM land_parcels
UNION ALL
SELECT 
    parcel_id,
    parcel_number,
    'Buffer_50m' as zone_type,
    50 as buffer_distance,
    ST_Buffer(ST_Transform(geometry, 3857), 50) as geometry
FROM land_parcels
UNION ALL
SELECT 
    parcel_id,
    parcel_number,
    'Buffer_100m' as zone_type,
    100 as buffer_distance,
    ST_Buffer(ST_Transform(geometry, 3857), 100) as geometry
FROM land_parcels;

-- Create materialized view for expensive calculations
CREATE MATERIALIZED VIEW parcel_statistics AS
SELECT 
    land_use,
    COUNT(*) as parcel_count,
    SUM(area_sqm) as total_area,
    AVG(area_sqm) as avg_area,
    SUM(assessed_value) as total_value,
    AVG(assessed_value) as avg_value,
    ST_Union(geometry) as combined_geometry
FROM land_parcels
GROUP BY land_use;

-- Create index on materialized view
CREATE INDEX idx_parcel_stats_geom 
ON parcel_statistics USING GIST(combined_geometry);
```

### Example 3: Advanced Spatial SQL Operations
```sql
-- Complex spatial join: Find parcels within 100m of water bodies
WITH water_buffers AS (
    SELECT 
        water_id,
        name as water_name,
        ST_Buffer(ST_Transform(geometry, 3857), 100) as buffer_geom
    FROM water_bodies
)
SELECT 
    lp.parcel_id,
    lp.parcel_number,
    lp.owner_name,
    wb.water_name,
    ST_Distance(
        ST_Transform(lp.geometry, 3857),
        ST_Transform(wb.geometry, 3857)
    ) as distance_to_water
FROM land_parcels lp
JOIN water_bodies wb ON ST_DWithin(
    ST_Transform(lp.geometry, 3857),
    ST_Transform(wb.geometry, 3857),
    100
)
ORDER BY lp.parcel_id, distance_to_water;

-- Spatial aggregation: Calculate land use statistics by district
SELECT 
    d.district_name,
    lp.land_use,
    COUNT(*) as parcel_count,
    SUM(lp.area_sqm) as total_area_sqm,
    ROUND(SUM(lp.area_sqm) / 10000, 2) as total_area_hectares,
    AVG(lp.assessed_value) as avg_assessed_value,
    ST_Area(ST_Union(lp.geometry)) as union_area
FROM districts d
JOIN land_parcels lp ON ST_Within(lp.geometry, d.geometry)
GROUP BY d.district_name, lp.land_use
ORDER BY d.district_name, total_area_sqm DESC;

-- Complex spatial analysis: Find optimal locations
WITH candidate_locations AS (
    SELECT 
        ST_Centroid(geometry) as location,
        parcel_id
    FROM land_parcels
    WHERE land_use = 'Vacant' 
      AND area_sqm > 5000
),
school_distances AS (
    SELECT 
        cl.parcel_id,
        cl.location,
        MIN(ST_Distance(
            ST_Transform(cl.location, 3857),
            ST_Transform(s.geometry, 3857)
        )) as nearest_school_distance
    FROM candidate_locations cl
    CROSS JOIN schools s
    GROUP BY cl.parcel_id, cl.location
),
road_distances AS (
    SELECT 
        cl.parcel_id,
        MIN(ST_Distance(
            ST_Transform(cl.location, 3857),
            ST_Transform(r.geometry, 3857)
        )) as nearest_road_distance
    FROM candidate_locations cl
    CROSS JOIN major_roads r
    GROUP BY cl.parcel_id
)
SELECT 
    lp.parcel_id,
    lp.parcel_number,
    lp.area_sqm,
    sd.nearest_school_distance,
    rd.nearest_road_distance,
    -- Scoring function (lower is better)
    (sd.nearest_school_distance * 0.3 + rd.nearest_road_distance * 0.7) as location_score
FROM land_parcels lp
JOIN school_distances sd ON lp.parcel_id = sd.parcel_id
JOIN road_distances rd ON lp.parcel_id = rd.parcel_id
ORDER BY location_score
LIMIT 10;
```

### Example 4: Data Import/Export Workflows
```sql
-- Create staging table for data import
CREATE TABLE parcels_staging (
    id SERIAL PRIMARY KEY,
    raw_data JSONB,
    geometry_text TEXT,
    import_batch VARCHAR(50),
    import_timestamp TIMESTAMP DEFAULT NOW(),
    processed BOOLEAN DEFAULT FALSE,
    error_message TEXT
);

-- Function to process staged data
CREATE OR REPLACE FUNCTION process_parcel_staging()
RETURNS INTEGER AS $$
DECLARE
    processed_count INTEGER := 0;
    staging_record RECORD;
BEGIN
    FOR staging_record IN 
        SELECT * FROM parcels_staging 
        WHERE NOT processed AND error_message IS NULL
    LOOP
        BEGIN
            -- Validate and insert data
            INSERT INTO land_parcels (
                parcel_number,
                owner_name,
                land_use,
                area_sqm,
                assessed_value,
                geometry
            )
            SELECT 
                staging_record.raw_data->>'parcel_number',
                staging_record.raw_data->>'owner_name',
                staging_record.raw_data->>'land_use',
                (staging_record.raw_data->>'area_sqm')::NUMERIC,
                (staging_record.raw_data->>'assessed_value')::NUMERIC,
                ST_GeomFromText(staging_record.geometry_text, 4326)
            WHERE ST_IsValid(ST_GeomFromText(staging_record.geometry_text, 4326));
            
            -- Mark as processed
            UPDATE parcels_staging 
            SET processed = TRUE 
            WHERE id = staging_record.id;
            
            processed_count := processed_count + 1;
            
        EXCEPTION WHEN OTHERS THEN
            -- Log error
            UPDATE parcels_staging 
            SET error_message = SQLERRM 
            WHERE id = staging_record.id;
        END;
    END LOOP;
    
    RETURN processed_count;
END;
$$ LANGUAGE plpgsql;

-- Export data with spatial transformations
CREATE OR REPLACE VIEW export_parcels_wgs84 AS
SELECT 
    parcel_number,
    owner_name,
    land_use,
    area_sqm,
    assessed_value,
    ST_AsGeoJSON(ST_Transform(geometry, 4326)) as geometry_geojson,
    ST_AsText(ST_Transform(geometry, 4326)) as geometry_wkt
FROM land_parcels;
```

## Best Practices

### Table Design
1. **Proper Constraints**: Use CHECK constraints for geometry validation
2. **Spatial Indexes**: Always create spatial indexes on geometry columns
3. **Consistent SRIDs**: Use consistent coordinate reference systems
4. **Normalization**: Follow database normalization principles

### Query Optimization
1. **Use Spatial Indexes**: Ensure spatial predicates can use indexes
2. **Appropriate Functions**: Use ST_DWithin instead of ST_Distance for proximity
3. **Geometry Simplification**: Simplify geometries for performance when appropriate
4. **Query Planning**: Use EXPLAIN ANALYZE to optimize queries

### Data Management
1. **Staging Tables**: Use staging tables for data import validation
2. **Batch Processing**: Process large datasets in batches
3. **Error Handling**: Implement comprehensive error handling
4. **Data Validation**: Validate spatial data before committing

### View Management
1. **Materialized Views**: Use for expensive calculations
2. **View Dependencies**: Document view dependencies
3. **Refresh Strategies**: Plan materialized view refresh schedules
4. **Security**: Use views to control data access

## Common Mistakes to Avoid

1. **Missing Spatial Indexes**: Not creating spatial indexes on geometry columns
2. **SRID Inconsistency**: Mixing different coordinate systems in operations
3. **Invalid Geometries**: Not validating geometries before operations
4. **Inefficient Joins**: Using non-spatial joins for spatial relationships
5. **Over-complex Views**: Creating views that are too complex to optimize
6. **No Error Handling**: Not implementing proper error handling in data processing
7. **Ignoring Performance**: Not monitoring query performance
8. **Poor Data Validation**: Insufficient validation during data import

## Intermediate Practice Projects

### Project 1: Urban Planning Database
**Objective**: Create a comprehensive urban planning database
**Tasks**:
- Design tables for parcels, buildings, zoning, infrastructure
- Create spatial views for zoning analysis
- Implement data import workflows for CAD and GIS data
- Build queries for development impact analysis
- Create materialized views for planning statistics

### Project 2: Environmental Monitoring System
**Objective**: Build a spatial database for environmental data
**Tasks**:
- Create tables for monitoring stations, measurements, boundaries
- Design views for temporal and spatial analysis
- Implement data validation for sensor data
- Build spatial queries for pollution analysis
- Create export functions for reporting

### Project 3: Transportation Network Analysis
**Objective**: Develop a transportation analysis database
**Tasks**:
- Design network topology tables
- Create views for route analysis
- Implement spatial joins for accessibility analysis
- Build queries for service area calculations
- Create performance optimization strategies

## Related Levels
- **Previous**: PostgreSQL/PostGIS Level 1 (Exploring Geospatial Database)
- **Next**: PostgreSQL/PostGIS Level 3 (Database Programming and Administration)
- **Related Topics**: 
  - Desktop GIS Level 2 (Layer Management and Styling)
  - GIS Level 2 (Intermediate GIS Operations)

## Q&A Section

### Basic Questions
**Q: What's the difference between a regular view and a materialized view?**
A: Regular views are virtual tables that execute the underlying query each time they're accessed. Materialized views store the query results physically and need to be refreshed when underlying data changes.

**Q: When should I use GEOMETRY vs GEOGRAPHY data types?**
A: Use GEOMETRY for local/regional analysis with projected coordinate systems (faster, more functions). Use GEOGRAPHY for global analysis with geographic coordinates (more accurate for distance calculations).

**Q: How do I optimize spatial queries for large datasets?**
A: Use spatial indexes, appropriate spatial predicates (ST_DWithin vs ST_Distance), geometry simplification, and consider partitioning large tables.

**Q: What's the best way to handle data import errors?**
A: Use staging tables, implement validation functions, log errors with details, and process data in batches with transaction control.

### Intermediate Questions
**Q: How do I design efficient spatial joins?**
A: Use spatial indexes, appropriate spatial predicates, consider geometry simplification, and use ST_Intersects before expensive operations like ST_Intersection.

**Q: What are the best practices for managing materialized views?**
A: Plan refresh schedules based on data update frequency, create indexes on materialized views, monitor refresh performance, and consider incremental refresh strategies.

**Q: How do I handle coordinate system transformations efficiently?**
A: Cache transformed geometries when possible, use appropriate target projections for analysis, and consider creating indexed computed columns for frequently used transformations.

**Q: What's the best approach for validating spatial data quality?**
A: Implement CHECK constraints, use ST_IsValid for geometry validation, check for appropriate coordinate ranges, and validate spatial relationships between related tables.

---

*This level focuses on creating and managing spatial database structures, performing advanced spatial queries, and implementing efficient data workflows in PostGIS.*