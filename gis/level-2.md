# GIS Level 2: Creating Maps and Digital Mapping

## Prerequisites
- Completed [Level 1: Foundation](level-1.md)
- Understanding of coordinate systems and spatial data types
- Basic computer skills
- Familiarity with file management

## Problem It Solves
Level 2 focuses on **creating maps and layers**, **digitizing real world to digital maps**, and **performing map authoring and styling**. You'll learn practical skills for making maps, converting physical features to digital format, and presenting spatial information effectively.

## Key Concepts

### 1. Creating Maps and Layers

#### Map Components
- **Base Layer**: Background reference (satellite imagery, topographic map)
- **Data Layers**: Thematic information (roads, buildings, boundaries)
- **Annotation Layers**: Labels, text, and symbols
- **Legend**: Explains symbols and colors used
- **Scale Bar**: Shows distance relationships
- **North Arrow**: Indicates map orientation
- **Title and Metadata**: Map purpose and data sources

#### Layer Management
- **Layer Order**: Controls what appears on top
- **Visibility**: Turn layers on/off
- **Transparency**: Blend layers together
- **Scale Dependencies**: Show/hide layers at different zoom levels

#### Map Types
- **Reference Maps**: Show locations and features
- **Thematic Maps**: Display specific data patterns
- **Topographic Maps**: Show elevation and terrain
- **Cadastral Maps**: Property boundaries and ownership

### 2. Digitizing Real World to Digital Maps

#### Digitization Process
1. **Preparation**: Gather reference materials (aerial photos, maps, GPS data)
2. **Setup**: Configure coordinate system and scale
3. **Digitizing**: Trace features using points, lines, and polygons
4. **Attribution**: Add descriptive information to features
5. **Quality Control**: Check accuracy and completeness

#### Digitization Methods
- **Heads-up Digitizing**: Tracing from aerial photos or satellite imagery
- **Table Digitizing**: Using paper maps on digitizing tablet
- **GPS Collection**: Direct field data collection
- **Scanning and Vectorization**: Converting paper maps to digital

#### Feature Types to Digitize
- **Points**: Buildings, wells, landmarks, sample locations
- **Lines**: Roads, rivers, pipelines, boundaries
- **Polygons**: Land parcels, administrative areas, land use zones

### 3. Map Authoring and Styling

#### Symbology Principles
- **Point Symbols**: Size, shape, color for different categories
- **Line Symbols**: Width, style, color for roads, boundaries
- **Fill Symbols**: Colors, patterns for areas and regions
- **Text Labels**: Font, size, placement for readability

#### Color Theory for Maps
- **Qualitative**: Different hues for categories (red, blue, green)
- **Sequential**: Light to dark for ordered data (population density)
- **Diverging**: Two colors from central value (temperature change)
- **Accessibility**: Consider color-blind friendly palettes

#### Classification Methods
- **Single Symbol**: All features look the same
- **Categorized**: Different symbols for different categories
- **Graduated**: Symbol size/color varies with data values
- **Rule-based**: Complex conditions for styling

#### Label Placement
- **Point Labels**: Above, below, or beside points
- **Line Labels**: Along the line, curved or straight
- **Polygon Labels**: Centered or following shape
- **Conflict Resolution**: Avoid overlapping labels

### 4. Common File Formats

#### Vector Formats
- **Shapefile (.shp)**: Most common, multiple files required
- **GeoJSON (.geojson)**: Web-friendly, human-readable
- **KML (.kml)**: Google Earth format
- **GeoPackage (.gpkg)**: Modern, single-file format

#### Raster Formats
- **GeoTIFF (.tif)**: Georeferenced images
- **JPEG (.jpg)**: Compressed images, no georeferencing
- **PNG (.png)**: Lossless compression
- **ECW (.ecw)**: Compressed aerial imagery

### 5. Quality Control in Digitizing

#### Common Errors
- **Overshoots**: Lines extending beyond intersections
- **Undershoots**: Lines not meeting at intersections
- **Gaps**: Missing areas between polygons
- **Overlaps**: Polygons covering same area
- **Duplicate Features**: Same feature digitized twice

#### Quality Checks
- **Visual Inspection**: Compare with source material
- **Topology Validation**: Check geometric relationships
- **Attribute Validation**: Verify data completeness
- **Scale Appropriateness**: Match detail level to map scale

## Hands-On Examples

### Example 1: Creating a Basic Map in QGIS

**Step-by-Step Process:**
1. **Open QGIS** and create a new project
2. **Add Base Layer**: 
   - Go to Browser Panel → XYZ Tiles → OpenStreetMap
   - Drag to map canvas
3. **Add Vector Layer**:
   - Layer → Add Layer → Add Vector Layer
   - Browse to your shapefile (e.g., cities.shp)
4. **Style the Layer**:
   - Right-click layer → Properties → Symbology
   - Choose "Single Symbol" or "Categorized"
   - Adjust colors, sizes, and transparency
5. **Add Labels**:
   - In Layer Properties → Labels
   - Enable labeling and choose field (e.g., city name)
6. **Create Layout**:
   - Project → New Print Layout
   - Add map, legend, scale bar, and north arrow

### Example 2: Digitizing Features

**Digitizing a Building:**
1. **Create New Layer**:
   - Layer → Create Layer → New Shapefile Layer
   - Choose Polygon geometry type
   - Add fields: building_id, building_type, height
2. **Start Editing**:
   - Select layer → Toggle Editing (pencil icon)
   - Use "Add Polygon Feature" tool
3. **Digitize Process**:
   - Click to create vertices around building outline
   - Right-click to finish polygon
   - Enter attribute values in dialog
4. **Quality Control**:
   - Use "Vertex Tool" to adjust points
   - Check for gaps and overlaps
   - Validate topology

### Example 3: Map Styling and Symbology

**Creating a Graduated Symbol Map:**
1. **Prepare Data**: Population data by city
2. **Open Symbology**:
   - Layer Properties → Symbology
   - Choose "Graduated"
3. **Configure Symbols**:
   - Value: population field
   - Method: Natural Breaks (Jenks)
   - Symbol: Circle with varying sizes
   - Color ramp: Light to dark
4. **Refine Appearance**:
   - Adjust class breaks manually if needed
   - Add outline to symbols
   - Set transparency for overlapping features

## Best Practices

### ✅ Data Management Do's
- **Use open standards** when possible (GeoPackage, GeoJSON)
- **Maintain original data** in separate archive
- **Document transformations** and processing steps
- **Implement version control** for data updates
- **Create data dictionaries** explaining attributes
- **Use consistent naming conventions**

### ❌ Common Mistakes
- **Mixing coordinate systems** without proper transformation
- **Ignoring data lineage** and processing history
- **Using inappropriate formats** for the use case
- **Skipping quality assessment** before analysis
- **Not backing up original data** before processing

### Format Selection Guidelines

| Use Case | Recommended Format | Why |
|----------|-------------------|-----|
| Web mapping | GeoJSON | Lightweight, web-friendly |
| Desktop GIS | GeoPackage | Multi-layer, efficient |
| Data exchange | Shapefile | Universal compatibility |
| Google Earth | KML/KMZ | Native format |
| Large datasets | GeoPackage/PostGIS | Database efficiency |
| Raster analysis | GeoTIFF | Standard with georeferencing |

## Mini-Projects

### Project 1: Data Format Laboratory
1. Download sample data in shapefile format
2. Convert to 3 different formats (GeoJSON, KML, GeoPackage)
3. Compare file sizes and loading times
4. Test compatibility with different software
5. Document pros/cons of each format

### Project 2: Buffer Analysis Workflow
1. Create buffers around schools (500m, 1km, 2km)
2. Calculate population within each buffer zone
3. Identify underserved areas (no schools within 2km)
4. Create a map showing service coverage
5. Generate a report with statistics

### Project 3: Data Quality Assessment
1. Obtain a dataset with known quality issues
2. Develop a quality assessment checklist
3. Run automated quality checks
4. Document all issues found
5. Create a data cleaning workflow
6. Validate improvements

## Geoprocessing Workflows

### Simple Workflow Example
```
1. Data Input
   ↓
2. Coordinate System Check/Transform
   ↓
3. Data Quality Assessment
   ↓
4. Spatial Analysis (Buffer/Overlay)
   ↓
5. Results Validation
   ↓
6. Output Generation
   ↓
7. Documentation
```

### Workflow Documentation Template
```markdown
## Workflow: [Name]
**Purpose**: [What problem does this solve?]
**Input Data**: [List all input datasets]
**Processing Steps**:
1. [Step 1 with parameters]
2. [Step 2 with parameters]
...
**Output**: [Description of results]
**Quality Checks**: [How to validate results]
**Known Limitations**: [What to watch out for]
```

## Related Levels
- **Previous**: [Level 1 - Foundation](level-1.md)
- **Next**: [Level 3 - Complex Analysis](level-3.md)
- **Related**: [Desktop GIS Level 2](../desktop-gis/level-2.md)

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: Which spatial data format should I use for my project?</strong></summary>

**Answer**: Choose based on your specific needs:

**For web applications**: GeoJSON (lightweight, JavaScript-friendly)
**For desktop GIS**: GeoPackage (efficient, multi-layer support)
**For data sharing**: Shapefile (universal compatibility, despite limitations)
**For Google Earth**: KML/KMZ (native support)
**For large datasets**: PostGIS database (performance, multi-user access)

**Consider**: File size, software compatibility, update frequency, and collaboration needs.
</details>

<details>
<summary><strong>Q2: What's the difference between WMS, WFS, and WCS?</strong></summary>

**Answer**: These are OGC web services for different data types:

**WMS (Web Map Service)**:
- Serves pre-rendered map images
- Good for visualization
- Cannot access underlying data
- Fast display, limited interaction

**WFS (Web Feature Service)**:
- Serves vector features with attributes
- Allows querying and editing
- Returns actual data (GML, GeoJSON)
- Interactive analysis possible

**WCS (Web Coverage Service)**:
- Serves raster/coverage data
- Returns actual pixel values
- Good for analysis and processing
- Supports subsetting and format conversion
</details>

<details>
<summary><strong>Q3: How do I handle coordinate system transformations safely?</strong></summary>

**Answer**: Follow these steps:

1. **Identify source CRS**: Check metadata or .prj file
2. **Choose target CRS**: Based on analysis requirements
3. **Use proper transformation**: Don't just change the CRS label
4. **Validate results**: Check known reference points
5. **Document the process**: Record transformation parameters

**Tools**: Use GIS software transformation tools or libraries like PROJ, not manual coordinate conversion.

**Validation**: Compare transformed coordinates with known accurate locations.
</details>

### Intermediate Questions

<details>
<summary><strong>Q4: What are the key considerations for spatial data quality assessment?</strong></summary>

**Answer**: Assess multiple quality dimensions:

**Positional Accuracy**:
- Compare with higher accuracy reference data
- Check against GPS measurements
- Validate coordinate system consistency

**Attribute Accuracy**:
- Cross-reference with authoritative sources
- Check for logical consistency
- Validate against field observations

**Completeness**:
- Assess spatial coverage gaps
- Check for missing attributes
- Evaluate temporal completeness

**Consistency**:
- Topological validation (no gaps/overlaps)
- Attribute standardization
- Format consistency

**Currency**:
- Check data collection/update dates
- Assess relevance for current use
- Plan for regular updates
</details>

<details>
<summary><strong>Q5: How do I optimize spatial data for web applications?</strong></summary>

**Answer**: Multi-step optimization approach:

**Geometry Simplification**:
- Reduce vertex density for appropriate scale
- Use Douglas-Peucker algorithm
- Maintain topological relationships

**Format Selection**:
- GeoJSON for small datasets
- Vector tiles for large datasets
- Consider TopoJSON for topology preservation

**Attribute Optimization**:
- Remove unnecessary fields
- Shorten field names
- Use appropriate data types

**Spatial Indexing**:
- Create spatial indices
- Use tiling strategies
- Implement level-of-detail (LOD)

**Compression**:
- Enable gzip compression
- Use efficient encoding
- Consider binary formats for large data
</details>

<details>
<summary><strong>Q6: What's the best approach for handling large spatial datasets?</strong></summary>

**Answer**: Implement a tiered strategy:

**Storage**:
- Use spatial databases (PostGIS, SpatiaLite)
- Implement spatial indexing
- Consider data partitioning

**Processing**:
- Process in chunks/tiles
- Use streaming algorithms
- Implement parallel processing

**Access**:
- Create multiple resolution levels
- Use caching strategies
- Implement progressive loading

**Tools**:
- GDAL for raster processing
- PostGIS for vector operations
- Cloud platforms for scalability

**Optimization**:
- Profile performance bottlenecks
- Optimize queries with spatial indices
- Use appropriate coordinate systems
</details>

### Advanced Questions

<details>
<summary><strong>Q7: How do I design an effective spatial data validation workflow?</strong></summary>

**Answer**: Create a comprehensive validation pipeline:

**Automated Checks**:
```python
# Example validation workflow
def validate_spatial_data(dataset):
    checks = {
        'geometry_validity': check_geometry_validity(dataset),
        'coordinate_bounds': check_coordinate_bounds(dataset),
        'attribute_completeness': check_attribute_completeness(dataset),
        'topological_consistency': check_topology(dataset),
        'metadata_presence': check_metadata(dataset)
    }
    return checks
```

**Manual Review**:
- Visual inspection of sample areas
- Cross-reference with authoritative sources
- Field verification of critical features

**Documentation**:
- Record all validation steps
- Document known limitations
- Create quality reports

**Continuous Monitoring**:
- Set up automated quality checks
- Monitor data updates
- Track quality metrics over time
</details>

<details>
<summary><strong>Q8: What are the best practices for spatial data integration from multiple sources?</strong></summary>

**Answer**: Follow a systematic integration approach:

**Pre-Integration Assessment**:
- Catalog all data sources
- Assess quality and compatibility
- Identify integration challenges

**Harmonization**:
- Standardize coordinate systems
- Align attribute schemas
- Resolve scale differences
- Handle temporal mismatches

**Conflict Resolution**:
- Define precedence rules
- Implement quality-based selection
- Document resolution decisions

**Quality Assurance**:
- Validate integrated results
- Check for edge effects
- Assess uncertainty propagation

**Documentation**:
- Record integration methodology
- Document data lineage
- Create integration metadata
</details>

<details>
<summary><strong>Q9: How do I implement effective spatial data versioning and change management?</strong></summary>

**Answer**: Establish a comprehensive versioning system:

**Version Control Strategy**:
- Use semantic versioning (major.minor.patch)
- Track both data and schema changes
- Maintain change logs

**Technical Implementation**:
```sql
-- Example versioning table structure
CREATE TABLE data_versions (
    version_id SERIAL PRIMARY KEY,
    version_number VARCHAR(20),
    change_date TIMESTAMP,
    change_type VARCHAR(50),
    description TEXT,
    changed_by VARCHAR(100)
);
```

**Change Detection**:
- Implement automated change detection
- Track geometry and attribute changes
- Monitor data quality metrics

**Rollback Capability**:
- Maintain previous versions
- Enable selective rollback
- Test rollback procedures

**User Communication**:
- Notify users of changes
- Provide migration guides
- Document breaking changes
</details>

---

**Next Step**: Ready for more complex analysis? Continue to [Level 3](level-3.md) to learn advanced spatial modeling and optimization techniques.