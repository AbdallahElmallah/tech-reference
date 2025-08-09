# Desktop GIS Level 1: Foundation - Basic Map Exploration

## Prerequisites
- Basic computer skills (file management, software installation)
- Understanding of basic geography concepts
- No prior GIS experience required

## Problem This Level Solves
This level introduces you to Desktop GIS applications, teaching you how to navigate the interface, load spatial data, perform basic queries, and understand coordinate systems - the fundamental skills needed for any GIS work.

## Key Concepts

### 1. Adding and Exploring Maps and Layers

#### Understanding GIS Data Structure
- **Layers**: Individual datasets that can be overlaid
- **Vector Data**: Points, lines, polygons representing discrete features
- **Raster Data**: Grid-based data like satellite imagery or elevation models
- **Attribute Data**: Non-spatial information linked to geographic features

#### Common Data Formats
- **Shapefile (.shp)**: Most common vector format
- **GeoTIFF (.tif)**: Georeferenced raster images
- **KML/KMZ**: Google Earth format
- **GeoJSON**: Web-friendly vector format
- **CSV with coordinates**: Simple point data

#### Layer Management
- **Layer Panel**: Organize and control layer visibility
- **Layer Order**: Control drawing order (top layers draw over bottom)
- **Layer Groups**: Organize related layers together
- **Symbology**: Visual representation of data

### 2. Basic Query and Select Data

#### Selection Methods
- **Click Selection**: Select individual features by clicking
- **Rectangle Selection**: Draw rectangle to select multiple features
- **Polygon Selection**: Draw custom shape for selection
- **Attribute Selection**: Select based on data values

#### Query Types
- **Spatial Queries**: "What's within 1km of this point?"
- **Attribute Queries**: "Show all cities with population > 100,000"
- **Combined Queries**: Mix spatial and attribute criteria

#### Query Operators
- **Equals (=)**: Exact match
- **Greater than (>)**: Numeric comparison
- **Contains**: Text contains substring
- **Between**: Value within range
- **Is Null**: Missing data

### 3. Coordinate Types and GOTO Point

#### Coordinate Systems Basics
- **Geographic Coordinates**: Latitude/Longitude (degrees)
- **Projected Coordinates**: X/Y in meters or feet
- **Common Systems**:
  - WGS84 (EPSG:4326): Global standard
  - Web Mercator (EPSG:3857): Web mapping standard
  - UTM: Regional high-accuracy system

#### Coordinate Formats
- **Decimal Degrees**: 40.7128, -74.0060
- **Degrees Minutes Seconds**: 40°42'46"N, 74°00'22"W
- **Projected**: 583960, 4507523 (UTM)

#### Navigation Tools
- **GOTO Point**: Navigate to specific coordinates
- **Zoom to Layer**: Fit layer extent in view
- **Zoom to Selection**: Focus on selected features
- **Bookmarks**: Save frequently used locations

## Hands-On Examples

### Example 1: Getting Started with QGIS

#### Installing and Setting Up QGIS
1. **Download QGIS**:
   - Visit https://qgis.org/en/site/forusers/download.html
   - Choose Long Term Release (LTR) for stability
   - Install with default settings

2. **First Launch Setup**:
   - Open QGIS Desktop
   - Set default coordinate system to WGS84 (EPSG:4326)
   - Configure data source manager

3. **Interface Overview**:
   ```
   Menu Bar: File, Edit, View, Layer, Settings, etc.
   Toolbars: Navigation, Attributes, Digitizing
   Map Canvas: Main viewing area
   Layers Panel: Layer management (left side)
   Browser Panel: Data source browser
   Status Bar: Coordinates, scale, projection info
   ```

#### Loading Your First Dataset
1. **Add Vector Layer**:
   - Click "Add Vector Layer" button or Layer → Add Layer → Add Vector Layer
   - Browse to sample data (QGIS comes with Natural Earth data)
   - Select "ne_110m_admin_0_countries.shp"
   - Click "Add" then "Close"

2. **Explore the Data**:
   - Right-click layer → "Zoom to Layer"
   - Use mouse wheel to zoom in/out
   - Click and drag to pan
   - Right-click layer → "Open Attribute Table"

3. **Add Raster Data**:
   - Layer → Add Layer → Add Raster Layer
   - Browse for a GeoTIFF file or use online services
   - Add OpenStreetMap basemap: Browser Panel → XYZ Tiles → OpenStreetMap

### Example 2: Basic Data Exploration

#### Examining Layer Properties
1. **Access Layer Properties**:
   - Right-click layer → "Properties"
   - Explore tabs: General, Source, Symbology, Labels

2. **Understanding Attribute Data**:
   ```
   Common Attributes in Country Layer:
   - NAME: Country name
   - POP_EST: Population estimate
   - GDP_MD_EST: GDP estimate
   - CONTINENT: Continental grouping
   ```

3. **Basic Statistics**:
   - Right-click layer → "Properties" → "Source" tab
   - View feature count, extent, coordinate system
   - Check data types and field information

#### Navigation and Measurement
1. **Using Navigation Tools**:
   ```
   Pan Tool: Click and drag to move map
   Zoom In: Click to zoom to point
   Zoom Out: Shift+click to zoom out
   Zoom to Full Extent: View all data
   ```

2. **Measurement Tools**:
   - View → Toolbars → Advanced Digitizing
   - Use "Measure Line" tool to measure distances
   - Use "Measure Area" tool for area calculations
   - Results show in both map units and degrees

### Example 3: Basic Queries and Selection

#### Simple Attribute Queries
1. **Select by Expression**:
   - Click "Select Features by Expression" button
   - Build expression: `"POP_EST" > 100000000`
   - Click "Select Features"
   - Selected countries highlight in yellow

2. **Common Query Examples**:
   ```sql
   -- Countries in Asia
   "CONTINENT" = 'Asia'
   
   -- Large countries by area
   $area > 1000000
   
   -- Countries starting with 'A'
   "NAME" LIKE 'A%'
   
   -- Countries with missing GDP data
   "GDP_MD_EST" IS NULL
   ```

3. **Working with Selections**:
   - View → "Show Selected Features"
   - Right-click layer → "Export" → "Save Selected Features As"
   - Edit → "Copy Features" and "Paste Features"

#### Spatial Selection
1. **Select by Location**:
   - Vector → Research Tools → "Select by Location"
   - Choose selection method (intersect, within, contains)
   - Select features based on spatial relationship

2. **Manual Selection Tools**:
   ```
   Select Features: Click individual features
   Select by Rectangle: Drag rectangle around features
   Select by Polygon: Draw custom selection area
   Select by Freehand: Draw freehand selection
   ```

### Example 4: Coordinate Systems and Navigation

#### Understanding Your Data's Coordinates
1. **Check Layer CRS**:
   - Right-click layer → "Properties" → "Source" tab
   - Note the Coordinate Reference System (CRS)
   - Common systems: EPSG:4326 (WGS84), EPSG:3857 (Web Mercator)

2. **Project CRS vs Layer CRS**:
   - Bottom right corner shows project CRS
   - Click to change project CRS
   - Layers can have different CRS - QGIS reprojects on-the-fly

#### Using GOTO Point Feature
1. **Navigate to Coordinates**:
   - View → "Go to Coordinate"
   - Enter coordinates in current project CRS
   - Examples:
     ```
     Decimal Degrees: -74.0060, 40.7128 (NYC)
     UTM: 583960, 4507523
     ```

2. **Coordinate Display**:
   - Status bar shows cursor coordinates
   - Right-click status bar to change format
   - Options: Decimal degrees, DMS, projected coordinates

3. **Creating Bookmarks**:
   - Navigate to desired location
   - View → "New Spatial Bookmark"
   - Name and save for quick access
   - Access via View → "Show Spatial Bookmarks"

### Example 5: Working with Different Data Sources

#### Loading Online Data
1. **Web Map Services (WMS)**:
   - Layer → Add Layer → Add WMS/WMTS Layer
   - Add connection to government data portals
   - Example: USGS National Map services

2. **Online Basemaps**:
   ```
   Browser Panel → XYZ Tiles:
   - OpenStreetMap
   - Google Satellite (if configured)
   - Bing Maps (if configured)
   ```

3. **GPS Data**:
   - Layer → Add Layer → Add Delimited Text Layer
   - Load CSV files with latitude/longitude columns
   - Specify coordinate fields and CRS

#### Data Import Workflow
1. **Preparation Checklist**:
   - [ ] Verify data format compatibility
   - [ ] Check coordinate system information
   - [ ] Ensure file paths are accessible
   - [ ] Backup original data

2. **Import Steps**:
   ```
   1. Add Layer → Choose appropriate layer type
   2. Browse to data file
   3. Configure import settings
   4. Verify coordinate system
   5. Add to map and verify display
   ```

3. **Troubleshooting Common Issues**:
   - **Data not displaying**: Check coordinate system
   - **Wrong location**: Verify CRS settings
   - **Missing attributes**: Check field mapping
   - **Slow performance**: Consider data simplification

## Best Practices

### ✅ Data Management
- **Organize Project Files**: Keep data, projects, and outputs in logical folders
- **Use Relative Paths**: Store data near project files for portability
- **Backup Regularly**: Save project files frequently
- **Document Data Sources**: Keep track of where data came from
- **Consistent Naming**: Use clear, descriptive layer and file names

### ✅ Interface Efficiency
- **Customize Toolbars**: Show only tools you use regularly
- **Learn Keyboard Shortcuts**: Speed up common operations
- **Use Layer Groups**: Organize related layers together
- **Save Workspace**: Preserve your interface layout
- **Create Templates**: Start new projects with standard settings

### ✅ Coordinate System Management
- **Understand Your Data**: Know the CRS of each dataset
- **Choose Appropriate CRS**: Use local projections for accurate measurements
- **Document CRS Choices**: Record why you chose specific coordinate systems
- **Verify Alignment**: Check that layers align properly
- **Plan for Output**: Consider final map projection needs

### ❌ Common Mistakes
- **Ignoring Coordinate Systems**: Not checking if data aligns properly
- **Overloading Maps**: Adding too many layers at once
- **Poor File Organization**: Scattered data files and projects
- **Not Saving Work**: Losing progress due to infrequent saves
- **Skipping Data Exploration**: Not examining attribute tables
- **Wrong Selection Tools**: Using inappropriate selection methods
- **Ignoring Scale**: Working at inappropriate zoom levels
- **Missing Metadata**: Not documenting data sources and processing

## Simple Practice Projects

### Project 1: Local Area Exploration
**Objective**: Create a map of your local area using multiple data sources

**Tasks**:
1. Download administrative boundaries for your region
2. Add OpenStreetMap basemap
3. Load points of interest (schools, hospitals, etc.)
4. Practice navigation and zoom tools
5. Create bookmarks for important locations
6. Export a simple map image

**Skills Practiced**: Data loading, navigation, layer management

### Project 2: Population Analysis
**Objective**: Explore world population data

**Tasks**:
1. Load world countries dataset
2. Examine population attributes
3. Select countries with population > 50 million
4. Create a new layer from selection
5. Calculate basic statistics
6. Navigate to specific countries using coordinates

**Skills Practiced**: Attribute queries, selection, basic analysis

### Project 3: Multi-Format Data Integration
**Objective**: Work with different data formats

**Tasks**:
1. Load shapefile data (administrative boundaries)
2. Add CSV point data (cities with coordinates)
3. Include raster basemap
4. Verify all layers align properly
5. Check and document coordinate systems
6. Practice different selection methods

**Skills Practiced**: Format handling, CRS management, data integration

### Project 4: Basic Spatial Queries
**Objective**: Perform location-based analysis

**Tasks**:
1. Load road network and city points
2. Select cities within 10km of major highways
3. Find all roads that intersect with water bodies
4. Identify the largest city in each administrative region
5. Create selections based on multiple criteria
6. Export results for further analysis

**Skills Practiced**: Spatial selection, complex queries, result export

## Related Levels
- **Next**: [Level 2 - Layer Management and Styling](level-2.md)
- **Related**: [GIS Fundamentals](../gis/level-1.md)
- **Related**: [Spatial Databases](../spatial-databases/level-1.md)

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: What's the difference between vector and raster data?</strong></summary>

**Answer**: Understanding data types is fundamental to GIS work:

**Vector Data**:
- **Structure**: Discrete geometric shapes (points, lines, polygons)
- **Examples**: City locations (points), roads (lines), country boundaries (polygons)
- **Advantages**: Precise boundaries, compact file size, scalable
- **File Formats**: Shapefile, GeoJSON, KML
- **Use Cases**: Administrative boundaries, infrastructure, precise mapping

**Raster Data**:
- **Structure**: Grid of cells/pixels with values
- **Examples**: Satellite imagery, elevation models, weather data
- **Advantages**: Good for continuous phenomena, analysis-ready
- **File Formats**: GeoTIFF, JPEG2000, NetCDF
- **Use Cases**: Environmental monitoring, terrain analysis, imagery

**When to Use Each**:
```
Vector: When you need precise boundaries and discrete features
Raster: When working with continuous data or imagery
Combined: Most projects use both types together
```

**Practical Example**:
- Vector: Property boundaries for land management
- Raster: Elevation data to calculate slope for each property
</details>

<details>
<summary><strong>Q2: How do I know which coordinate system to use?</strong></summary>

**Answer**: Coordinate system choice depends on your project needs:

**For Global Projects**:
- **WGS84 (EPSG:4326)**: Standard for GPS data and global datasets
- **Web Mercator (EPSG:3857)**: Web mapping applications

**For Regional/Local Projects**:
- **UTM Zones**: High accuracy for areas within UTM zone boundaries
- **State Plane (US)**: Optimized for individual US states
- **National Grids**: Country-specific systems (e.g., British National Grid)

**Decision Factors**:
```
1. Geographic Extent:
   - Global: WGS84
   - Continental: Regional projection
   - Local: Local coordinate system

2. Accuracy Requirements:
   - High precision: Local projected system
   - General mapping: Geographic coordinates

3. Measurement Needs:
   - Distance/area calculations: Projected system
   - Display only: Geographic acceptable

4. Data Integration:
   - Match existing project data
   - Consider data source CRS
```

**Common Workflow**:
1. Check what CRS your data uses
2. Determine your analysis area
3. Choose appropriate local projection if measuring
4. Reproject data if necessary
5. Document your choice
</details>

<details>
<summary><strong>Q3: Why isn't my data showing up on the map?</strong></summary>

**Answer**: This is a common issue with several possible causes:

**Most Common Causes**:

1. **Wrong Coordinate System**:
   ```
   Problem: Data appears in wrong location or not at all
   Solution: Check layer CRS in Properties → Source
   Fix: Right-click layer → Set CRS → Choose correct system
   ```

2. **Data Outside Current View**:
   ```
   Problem: Data exists but not visible in current extent
   Solution: Right-click layer → Zoom to Layer
   Prevention: Always zoom to new layers after adding
   ```

3. **Incorrect File Path**:
   ```
   Problem: Layer shows with red exclamation mark
   Solution: Right-click → Change Data Source
   Fix: Browse to correct file location
   ```

4. **Unsupported Format**:
   ```
   Problem: File won't load at all
   Solution: Check QGIS supported formats list
   Fix: Convert to supported format or install plugins
   ```

**Troubleshooting Steps**:
1. **Check Layer Panel**: Look for warning symbols
2. **Verify Extent**: Use "Zoom to Layer" function
3. **Examine Properties**: Check source path and CRS
4. **Test with Known Data**: Load sample data to verify QGIS works
5. **Check File Integrity**: Open file in text editor if possible

**Prevention Tips**:
- Always zoom to layer after adding
- Keep data files in project folder
- Use relative paths when possible
- Document coordinate systems
- Test data before important work
</details>

<details>
<summary><strong>Q4: How do I select features that meet multiple criteria?</strong></summary>

**Answer**: Complex selections require combining multiple conditions:

**Using Select by Expression**:
1. **Access Tool**: Click "Select by Expression" button or F3
2. **Build Complex Expressions**:
   ```sql
   -- Multiple AND conditions
   "POPULATION" > 1000000 AND "CONTINENT" = 'Europe'
   
   -- OR conditions
   "COUNTRY" = 'USA' OR "COUNTRY" = 'Canada'
   
   -- Range conditions
   "GDP_PER_CAPITA" BETWEEN 20000 AND 50000
   
   -- Text patterns
   "CITY_NAME" LIKE 'New%' AND "POPULATION" > 100000
   
   -- Null value handling
   "GDP" IS NOT NULL AND "GDP" > 0
   ```

**Combining Spatial and Attribute Criteria**:
```sql
-- Cities in specific region with large population
intersects($geometry, geometry(get_feature('regions', 'name', 'California')))
AND "POPULATION" > 500000

-- Features within distance of point
distance($geometry, make_point(-122.4194, 37.7749)) < 50000
AND "TYPE" = 'Hospital'
```

**Step-by-Step Selection Process**:
1. **Plan Your Criteria**: Write down what you want to select
2. **Check Field Names**: Open attribute table to verify field names
3. **Test Simple Conditions**: Start with one condition, then add more
4. **Use Parentheses**: Group conditions logically
5. **Verify Results**: Check selection makes sense

**Advanced Selection Techniques**:
- **Select by Location**: Vector → Research Tools → Select by Location
- **Iterative Selection**: Build selection in multiple steps
- **Save Selections**: Right-click layer → Export → Save Selected Features
</details>

<details>
<summary><strong>Q5: What file formats should I use for different types of GIS data?</strong></summary>

**Answer**: Choose formats based on data type and intended use:

**Vector Data Formats**:

**Shapefile (.shp)**:
- **Best for**: General vector data, compatibility
- **Pros**: Universal support, stable, well-documented
- **Cons**: 2GB size limit, multiple files, limited field names
- **Use when**: Sharing data, long-term storage, compatibility needed

**GeoPackage (.gpkg)**:
- **Best for**: Modern projects, complex data
- **Pros**: Single file, no size limits, supports multiple layers
- **Cons**: Newer format, less universal support
- **Use when**: Working within QGIS, need multiple layers in one file

**GeoJSON (.geojson)**:
- **Best for**: Web applications, simple data
- **Pros**: Human-readable, web-friendly, lightweight
- **Cons**: Large file sizes for complex data
- **Use when**: Web mapping, data exchange, simple geometries

**Raster Data Formats**:

**GeoTIFF (.tif)**:
- **Best for**: Most raster applications
- **Pros**: Preserves georeferencing, widely supported
- **Cons**: Large file sizes
- **Use when**: Satellite imagery, elevation data, analysis

**Format Selection Guide**:
```
For Sharing: Shapefile (vector), GeoTIFF (raster)
For Analysis: GeoPackage (vector), GeoTIFF (raster)
For Web: GeoJSON (vector), Web tiles (raster)
For Archives: Shapefile (vector), GeoTIFF (raster)
```

**Best Practices**:
- Keep original data in native format
- Use appropriate format for each use case
- Document format choices
- Consider file size vs. quality trade-offs
- Test compatibility with target software
</details>

---

**Congratulations!** You've completed the foundation level of Desktop GIS. You should now be comfortable with basic GIS operations, data loading, simple queries, and coordinate systems. Ready to move on to [Level 2 - Layer Management and Styling](level-2.md)?