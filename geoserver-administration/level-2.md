# GeoServer Administration Level 2: Intermediate Operations

## Prerequisites
- Completed Level 1: Foundation
- Basic understanding of CSS and styling concepts
- Familiarity with XML and JSON formats
- Understanding of web caching concepts
- Basic knowledge of backup and restore procedures

## Problem This Level Solves
This level teaches you how to customize the appearance and performance of your GeoServer services through styling, caching, and proper backup procedures, making your services production-ready and visually appealing.

## Key Concepts

### 1. Map Services Styling

#### Styled Layer Descriptor (SLD)
SLD is an XML-based language for styling geographic data:

- **Purpose**: Define how geographic features should be rendered
- **Standard**: OGC standard for map styling
- **Format**: XML with specific schema
- **Scope**: Works with WMS services

#### CSS Styling (GeoServer Extension)
Simpler alternative to SLD:

- **Syntax**: CSS-like syntax for styling
- **Conversion**: Automatically converts to SLD
- **Ease of use**: More intuitive than raw SLD
- **Limitations**: Not all SLD features supported

#### YSLD (YAML SLD)
YAML-based styling language:

- **Format**: Human-readable YAML syntax
- **Conversion**: Translates to SLD internally
- **Benefits**: Cleaner syntax than XML
- **Support**: GeoServer community extension

### 2. Layer Properties Customization

#### Layer Metadata
- **Title**: Human-readable layer name
- **Abstract**: Detailed layer description
- **Keywords**: Search and discovery terms
- **Attribution**: Data source credits
- **Metadata URLs**: Links to detailed metadata

#### Coordinate Reference Systems
- **Native CRS**: Original data projection
- **Declared CRS**: Override for incorrect data
- **Reprojection**: On-the-fly coordinate transformation
- **Supported CRS**: Additional projections for output

#### Feature Type Details
- **Attributes**: Field definitions and types
- **Geometry**: Spatial column configuration
- **Filters**: Default feature filtering
- **Dimensions**: Time and elevation support

### 3. Map Caching

#### GeoWebCache Integration
Built-in tile caching system:

- **Purpose**: Pre-generate and cache map tiles
- **Benefits**: Faster map loading, reduced server load
- **Storage**: File system or database storage
- **Standards**: WMTS, TMS, Google Maps compatible

#### Cache Configuration
- **Grid Sets**: Tile pyramid definitions
- **Formats**: PNG, JPEG, PNG8 tile formats
- **Zoom Levels**: Minimum and maximum zoom
- **Seeding**: Pre-generation of tiles

#### Cache Management
- **Seeding**: Generate tiles in advance
- **Truncation**: Remove outdated tiles
- **Statistics**: Monitor cache usage
- **Disk Quota**: Manage storage limits

### 4. Backup and Restore

#### Data Directory Structure
```
GEOSERVER_DATA_DIR/
├── global.xml              # Global configuration
├── logging.xml             # Logging configuration
├── workspaces/             # Workspace definitions
├── styles/                 # Style definitions
├── layergroups/           # Layer group configurations
├── security/              # Security settings
├── gwc/                   # GeoWebCache configuration
└── logs/                  # Log files
```

#### Backup Strategies
- **Full Backup**: Complete data directory
- **Configuration Only**: Settings without data
- **Incremental**: Changed files only
- **Database Backup**: External data sources

#### Restore Procedures
- **Clean Install**: Fresh GeoServer installation
- **Selective Restore**: Specific components only
- **Version Compatibility**: Ensure compatible versions
- **Testing**: Verify restored functionality

## Hands-On Examples

### Example 1: Creating Custom Styles

#### Basic SLD Style for Countries
```xml
<?xml version="1.0" encoding="UTF-8"?>
<StyledLayerDescriptor version="1.0.0"
    xmlns="http://www.opengis.net/sld"
    xmlns:ogc="http://www.opengis.net/ogc">
  <NamedLayer>
    <Name>countries_style</Name>
    <UserStyle>
      <Title>Country Boundaries</Title>
      <FeatureTypeStyle>
        <Rule>
          <Name>Countries</Name>
          <Title>Country Polygons</Title>
          <PolygonSymbolizer>
            <Fill>
              <CssParameter name="fill">#E6F3FF</CssParameter>
              <CssParameter name="fill-opacity">0.7</CssParameter>
            </Fill>
            <Stroke>
              <CssParameter name="stroke">#2E75B6</CssParameter>
              <CssParameter name="stroke-width">1</CssParameter>
            </Stroke>
          </PolygonSymbolizer>
        </Rule>
      </FeatureTypeStyle>
    </UserStyle>
  </NamedLayer>
</StyledLayerDescriptor>
```

#### CSS Style (Equivalent)
```css
/* Country styling */
* {
  fill: #E6F3FF;
  fill-opacity: 0.7;
  stroke: #2E75B6;
  stroke-width: 1px;
}
```

#### Creating and Applying Style
1. **Create Style**:
   - Navigate to "Styles" in GeoServer admin
   - Click "Add a new style"
   - Name: `countries_blue`
   - Format: Choose SLD or CSS
   - Paste style content
   - Validate and save

2. **Apply to Layer**:
   - Go to "Layers" → Select your layer
   - "Publishing" tab → "Default Style"
   - Select `countries_blue`
   - Save layer configuration

3. **Test Style**:
   - Use "Layer Preview" to verify appearance
   - Check different zoom levels
   - Validate in external GIS software

### Example 2: Advanced Styling with Rules

#### Population-Based Country Styling
```xml
<FeatureTypeStyle>
  <!-- Small countries -->
  <Rule>
    <Name>SmallCountries</Name>
    <Title>Population &lt; 10M</Title>
    <ogc:Filter>
      <ogc:PropertyIsLessThan>
        <ogc:PropertyName>POP_EST</ogc:PropertyName>
        <ogc:Literal>10000000</ogc:Literal>
      </ogc:PropertyIsLessThan>
    </ogc:Filter>
    <PolygonSymbolizer>
      <Fill>
        <CssParameter name="fill">#FFF2CC</CssParameter>
      </Fill>
      <Stroke>
        <CssParameter name="stroke">#D6B656</CssParameter>
        <CssParameter name="stroke-width">0.5</CssParameter>
      </Stroke>
    </PolygonSymbolizer>
  </Rule>
  
  <!-- Medium countries -->
  <Rule>
    <Name>MediumCountries</Name>
    <Title>Population 10M - 100M</Title>
    <ogc:Filter>
      <ogc:And>
        <ogc:PropertyIsGreaterThanOrEqualTo>
          <ogc:PropertyName>POP_EST</ogc:PropertyName>
          <ogc:Literal>10000000</ogc:Literal>
        </ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyIsLessThan>
          <ogc:PropertyName>POP_EST</ogc:PropertyName>
          <ogc:Literal>100000000</ogc:Literal>
        </ogc:PropertyIsLessThan>
      </ogc:And>
    </ogc:Filter>
    <PolygonSymbolizer>
      <Fill>
        <CssParameter name="fill">#FFE6CC</CssParameter>
      </Fill>
      <Stroke>
        <CssParameter name="stroke">#E67E22</CssParameter>
        <CssParameter name="stroke-width">1</CssParameter>
      </Stroke>
    </PolygonSymbolizer>
  </Rule>
  
  <!-- Large countries -->
  <Rule>
    <Name>LargeCountries</Name>
    <Title>Population &gt; 100M</Title>
    <ogc:Filter>
      <ogc:PropertyIsGreaterThanOrEqualTo>
        <ogc:PropertyName>POP_EST</ogc:PropertyName>
        <ogc:Literal>100000000</ogc:Literal>
      </ogc:PropertyIsGreaterThanOrEqualTo>
    </ogc:Filter>
    <PolygonSymbolizer>
      <Fill>
        <CssParameter name="fill">#FFCCCC</CssParameter>
      </Fill>
      <Stroke>
        <CssParameter name="stroke">#C0392B</CssParameter>
        <CssParameter name="stroke-width">2</CssParameter>
      </Stroke>
    </PolygonSymbolizer>
  </Rule>
</FeatureTypeStyle>
```

### Example 3: Setting Up Tile Caching

#### Enable Caching for Layer
1. **Access Layer Configuration**:
   - Navigate to "Layers" → Select layer
   - Go to "Tile Caching" tab
   - Check "Create a cached layer for this layer"

2. **Configure Cache Settings**:
   ```
   Tile Image Formats: image/png, image/jpeg
   Grid Sets: EPSG:4326, EPSG:3857 (Web Mercator)
   Zoom Start: 0
   Zoom Stop: 18
   ```

3. **Set Cache Parameters**:
   - **Gutter**: 0 pixels (for simple data)
   - **Meta-tiling**: 4x4 (balance between efficiency and memory)
   - **Expire Cache**: 0 (never expires)
   - **Expire Clients**: 7200 seconds (2 hours)

#### Seed Cache
1. **Access GeoWebCache Interface**:
   - Navigate to `http://localhost:8080/geoserver/gwc`
   - Login with GeoServer credentials

2. **Configure Seeding**:
   - Select layer to seed
   - Choose grid set (EPSG:3857 for web maps)
   - Set zoom range (e.g., 0-10 for initial seeding)
   - Select image format (PNG for quality, JPEG for size)

3. **Start Seeding Process**:
   ```
   Number of tasks to use: 2-4 (based on server capacity)
   Type of operation: Seed
   Zoom start: 0
   Zoom stop: 10
   Format: image/png
   ```

4. **Monitor Progress**:
   - Check seeding status in GWC interface
   - Monitor server resources during seeding
   - Verify tiles are being created in cache directory

### Example 4: Backup and Restore Procedure

#### Creating Full Backup
```bash
# Stop GeoServer service
sudo systemctl stop geoserver

# Create backup directory
mkdir -p /backup/geoserver/$(date +%Y%m%d)

# Backup data directory
cp -r $GEOSERVER_DATA_DIR /backup/geoserver/$(date +%Y%m%d)/data_dir

# Backup installation directory (optional)
cp -r /opt/geoserver /backup/geoserver/$(date +%Y%m%d)/installation

# Create archive
tar -czf /backup/geoserver_backup_$(date +%Y%m%d).tar.gz \
    /backup/geoserver/$(date +%Y%m%d)

# Start GeoServer service
sudo systemctl start geoserver
```

#### Automated Backup Script
```bash
#!/bin/bash
# geoserver_backup.sh

BACKUP_DIR="/backup/geoserver"
DATE=$(date +%Y%m%d_%H%M%S)
GEOSERVER_DATA_DIR="/opt/geoserver/data_dir"
RETENTION_DAYS=30

# Create backup directory
mkdir -p "$BACKUP_DIR/$DATE"

# Backup configuration only (faster)
cp -r "$GEOSERVER_DATA_DIR/workspaces" "$BACKUP_DIR/$DATE/"
cp -r "$GEOSERVER_DATA_DIR/styles" "$BACKUP_DIR/$DATE/"
cp -r "$GEOSERVER_DATA_DIR/security" "$BACKUP_DIR/$DATE/"
cp "$GEOSERVER_DATA_DIR/global.xml" "$BACKUP_DIR/$DATE/"
cp "$GEOSERVER_DATA_DIR/logging.xml" "$BACKUP_DIR/$DATE/"

# Create archive
tar -czf "$BACKUP_DIR/geoserver_config_$DATE.tar.gz" \
    -C "$BACKUP_DIR" "$DATE"

# Remove temporary directory
rm -rf "$BACKUP_DIR/$DATE"

# Clean old backups
find "$BACKUP_DIR" -name "geoserver_config_*.tar.gz" \
    -mtime +$RETENTION_DAYS -delete

echo "Backup completed: geoserver_config_$DATE.tar.gz"
```

#### Restore Procedure
```bash
# Stop GeoServer
sudo systemctl stop geoserver

# Backup current configuration (safety)
mv $GEOSERVER_DATA_DIR $GEOSERVER_DATA_DIR.backup.$(date +%Y%m%d)

# Extract backup
tar -xzf /backup/geoserver_backup_20240115.tar.gz -C /tmp

# Restore data directory
cp -r /tmp/backup/geoserver/20240115/data_dir $GEOSERVER_DATA_DIR

# Set proper ownership
chown -R geoserver:geoserver $GEOSERVER_DATA_DIR

# Start GeoServer
sudo systemctl start geoserver

# Verify restoration
curl -s http://localhost:8080/geoserver/rest/about/version.json
```

## Best Practices

### ✅ Styling Best Practices
- **Use semantic naming**: Style names should describe purpose, not appearance
- **Organize styles**: Group related styles in logical categories
- **Test at multiple scales**: Verify styles work at different zoom levels
- **Optimize for performance**: Avoid complex filters in high-traffic layers
- **Document styles**: Include comments explaining styling logic
- **Version control**: Keep style definitions in version control
- **Use appropriate formats**: SLD for complex rules, CSS for simple styling

### ✅ Caching Best Practices
- **Cache popular layers**: Focus on frequently accessed data
- **Choose appropriate formats**: PNG for quality, JPEG for file size
- **Set reasonable zoom ranges**: Don't cache unnecessary detail levels
- **Monitor disk usage**: Implement disk quotas and cleanup policies
- **Seed strategically**: Pre-generate tiles for expected usage areas
- **Update cache regularly**: Refresh when underlying data changes
- **Use CDN**: Distribute cached tiles through content delivery networks

### ✅ Backup and Restore Best Practices
- **Regular automated backups**: Schedule daily configuration backups
- **Test restore procedures**: Regularly verify backup integrity
- **Document procedures**: Maintain clear restoration instructions
- **Separate data and config**: Backup strategies may differ
- **Version compatibility**: Ensure backups work with GeoServer versions
- **Secure backup storage**: Protect backups from unauthorized access
- **Monitor backup success**: Alert on backup failures

### ❌ Common Mistakes
- **Overly complex styles**: Performance impact from complex SLD rules
- **Inconsistent styling**: Different styles for similar data types
- **Excessive caching**: Caching data that changes frequently
- **Inadequate testing**: Not testing styles across zoom levels
- **Poor backup strategy**: Infrequent or untested backups
- **Ignoring performance**: Not monitoring cache hit rates
- **Missing documentation**: Undocumented styling and configuration choices

## Practice Projects

### Project 1: Advanced Layer Styling
1. Create a choropleth map style based on population density
2. Implement scale-dependent styling (different styles at different zoom levels)
3. Add labels that appear only at appropriate scales
4. Create a legend-friendly style with clear categories
5. Test style performance with large datasets

### Project 2: Comprehensive Caching Setup
1. Enable caching for multiple layers with different characteristics
2. Configure appropriate grid sets for your region
3. Set up seeding for critical zoom levels
4. Implement cache monitoring and alerting
5. Create cache maintenance procedures

### Project 3: Backup and Recovery System
1. Design backup strategy for your GeoServer deployment
2. Implement automated backup scripts
3. Test full restoration procedure
4. Document recovery time objectives
5. Create disaster recovery plan

### Project 4: Layer Group Management
1. Create logical layer groups for related data
2. Configure group-level styling and metadata
3. Set up caching for layer groups
4. Implement group-based security
5. Test group functionality in client applications

## Related Levels
- **Previous**: [Level 1 - Foundation](level-1.md)
- **Next**: [Level 3 - Security and Performance](level-3.md)
- **Related**: [GIS Level 2](../gis/level-2.md)
- **Related**: [PostgreSQL/PostGIS Level 2](../postgresql-postgis/level-2.md)

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: What's the difference between SLD, CSS, and YSLD styling?</strong></summary>

**Answer**: Three different approaches to styling GeoServer layers:

**SLD (Styled Layer Descriptor)**:
- **Format**: XML-based OGC standard
- **Pros**: Full feature support, industry standard, portable
- **Cons**: Verbose XML syntax, steep learning curve
- **Best for**: Complex styling rules, production environments

**CSS Styling**:
- **Format**: CSS-like syntax (GeoServer extension)
- **Pros**: Familiar syntax, easier to learn, more concise
- **Cons**: Limited feature set, GeoServer-specific
- **Best for**: Simple to moderate styling, rapid development

**YSLD (YAML SLD)**:
- **Format**: YAML-based syntax
- **Pros**: Human-readable, less verbose than SLD
- **Cons**: Community extension, less widespread adoption
- **Best for**: Teams familiar with YAML, configuration management

**Recommendation**: Start with CSS for simple styles, move to SLD for complex requirements.
</details>

<details>
<summary><strong>Q2: How do I know if my tile caching is working effectively?</strong></summary>

**Answer**: Monitor several key indicators:

**Cache Hit Rate**:
- Access GeoWebCache statistics page
- Look for high hit rates (>80% for popular layers)
- Low hit rates indicate poor cache utilization

**Response Times**:
- Compare cached vs. uncached layer performance
- Cached tiles should load significantly faster
- Use browser developer tools to measure

**Server Load**:
- Monitor CPU and memory usage
- Effective caching reduces server load
- Check during peak usage periods

**Cache Statistics**:
```
# Access cache statistics
http://localhost:8080/geoserver/gwc/rest/statistics

# Key metrics to monitor:
- Cache hits vs. misses
- Disk usage
- Seeding progress
- Error rates
```

**Troubleshooting Poor Performance**:
- Check if appropriate zoom levels are cached
- Verify grid sets match client requests
- Ensure adequate disk space
- Monitor seeding completion
</details>

<details>
<summary><strong>Q3: How often should I backup my GeoServer configuration?</strong></summary>

**Answer**: Backup frequency depends on change rate and criticality:

**Configuration Backups**:
- **Daily**: For active development environments
- **Weekly**: For stable production systems
- **Before changes**: Always backup before major modifications
- **After changes**: Backup successful configurations

**Data Backups**:
- **Real-time**: For critical, frequently changing data
- **Daily**: For important datasets
- **Weekly**: For reference data that rarely changes

**Backup Strategy**:
```bash
# Minimal daily backup (configuration only)
- Workspaces, styles, security settings
- Size: Usually < 100MB
- Time: Few minutes

# Full weekly backup (everything)
- Complete data directory
- Size: Depends on cached tiles
- Time: Can be hours for large caches
```

**Retention Policy**:
- Keep daily backups for 30 days
- Keep weekly backups for 6 months
- Keep monthly backups for 2 years
- Archive critical configurations permanently
</details>

### Intermediate Questions

<details>
<summary><strong>Q4: How do I create scale-dependent styling?</strong></summary>

**Answer**: Use scale denominators in SLD rules:

**Scale Denominator Concept**:
- Represents map scale (1:X)
- Larger numbers = smaller scale (zoomed out)
- Smaller numbers = larger scale (zoomed in)

**SLD Scale Rules**:
```xml
<!-- Show only at large scales (zoomed in) -->
<Rule>
  <Name>DetailedView</Name>
  <MaxScaleDenominator>100000</MaxScaleDenominator>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#FF0000</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#000000</CssParameter>
      <CssParameter name="stroke-width">2</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>

<!-- Show only at small scales (zoomed out) -->
<Rule>
  <Name>OverviewView</Name>
  <MinScaleDenominator>100000</MinScaleDenominator>
  <PolygonSymbolizer>
    <Fill>
      <CssParameter name="fill">#0000FF</CssParameter>
    </Fill>
    <Stroke>
      <CssParameter name="stroke">#000000</CssParameter>
      <CssParameter name="stroke-width">1</CssParameter>
    </Stroke>
  </PolygonSymbolizer>
</Rule>
```

**CSS Equivalent**:
```css
/* Detailed view for large scales */
[@scale < 100000] {
  fill: #FF0000;
  stroke: #000000;
  stroke-width: 2px;
}

/* Overview for small scales */
[@scale >= 100000] {
  fill: #0000FF;
  stroke: #000000;
  stroke-width: 1px;
}
```

**Common Scale Ranges**:
- World view: > 50,000,000
- Country view: 1,000,000 - 50,000,000
- Regional view: 100,000 - 1,000,000
- City view: 10,000 - 100,000
- Street view: < 10,000
</details>

<details>
<summary><strong>Q5: What should I include in a disaster recovery plan for GeoServer?</strong></summary>

**Answer**: Comprehensive disaster recovery planning:

**Recovery Time Objectives (RTO)**:
- **Critical services**: < 4 hours
- **Important services**: < 24 hours
- **Non-critical services**: < 72 hours

**Recovery Point Objectives (RPO)**:
- **Configuration**: < 24 hours data loss
- **Critical data**: < 1 hour data loss
- **Reference data**: < 1 week data loss

**Disaster Recovery Components**:

1. **Backup Strategy**:
   ```bash
   # Automated daily backups
   - Configuration files
   - Style definitions
   - Security settings
   - Workspace definitions
   
   # Weekly full backups
   - Complete data directory
   - Cache tiles (if critical)
   - External data sources
   ```

2. **Infrastructure Requirements**:
   - Backup hardware specifications
   - Network connectivity requirements
   - Software licensing considerations
   - External dependencies (databases, file systems)

3. **Recovery Procedures**:
   ```bash
   # Step-by-step restoration process
   1. Provision new hardware/VM
   2. Install GeoServer (same version)
   3. Restore data directory
   4. Verify external connections
   5. Test critical services
   6. Update DNS/load balancers
   ```

4. **Testing Schedule**:
   - Monthly: Backup integrity verification
   - Quarterly: Partial recovery testing
   - Annually: Full disaster recovery drill

5. **Communication Plan**:
   - Stakeholder notification procedures
   - Status update mechanisms
   - Service restoration announcements
</details>

<details>
<summary><strong>Q6: How do I optimize tile caching for different types of data?</strong></summary>

**Answer**: Tailor caching strategies to data characteristics:

**Static Reference Data** (boundaries, roads):
```
Cache Strategy: Aggressive
- Zoom levels: 0-18
- Formats: PNG (for quality)
- Seeding: Full extent
- Expiration: Never (or very long)
- Meta-tiling: 4x4 or 8x8
```

**Dynamic Data** (weather, traffic):
```
Cache Strategy: Conservative
- Zoom levels: 0-12 (overview only)
- Formats: JPEG (for speed)
- Seeding: Popular areas only
- Expiration: 1-6 hours
- Meta-tiling: 2x2
```

**Large Scale Detailed Data** (parcels, buildings):
```
Cache Strategy: Selective
- Zoom levels: 12-18 (high detail only)
- Formats: PNG (for accuracy)
- Seeding: Urban areas only
- Expiration: Daily
- Meta-tiling: 4x4
```

**Optimization Techniques**:

1. **Grid Set Selection**:
   - Use EPSG:3857 for web applications
   - Use local projections for regional data
   - Avoid unnecessary grid sets

2. **Format Selection**:
   ```
   PNG: Best for
   - Data with transparency
   - Sharp boundaries
   - Text/labels
   
   JPEG: Best for
   - Aerial imagery
   - Continuous data (elevation)
   - Large file size concerns
   
   PNG8: Best for
   - Simple thematic maps
   - Limited color palettes
   - Balance of quality and size
   ```

3. **Seeding Strategy**:
   ```bash
   # Seed by bounding box (urban areas)
   gwc-seed.sh -type seed -bbox "-74.1,40.6,-73.9,40.8" \
               -gridSet EPSG:3857 -zoomStart 10 -zoomStop 16
   
   # Seed by polygon (irregular areas)
   gwc-seed.sh -type seed -mask /path/to/polygon.wkt \
               -gridSet EPSG:3857 -zoomStart 8 -zoomStop 14
   ```

4. **Performance Monitoring**:
   - Track cache hit rates by layer
   - Monitor disk I/O during seeding
   - Analyze request patterns
   - Adjust based on usage statistics
</details>

---

**Next Step**: Ready to secure and optimize your GeoServer? Continue to [Level 3](level-3.md) to learn about security, troubleshooting, and performance tuning.