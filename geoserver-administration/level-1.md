# GeoServer Administration Level 1: Foundation

## Prerequisites
- Basic understanding of GIS concepts
- Familiarity with web browsers
- Basic knowledge of file systems
- Understanding of spatial data formats (Shapefile, GeoTIFF)

## Problem This Level Solves
This level teaches you how to set up and perform basic administration of GeoServer, enabling you to publish spatial data as web services for use in web applications and GIS software.

## Key Concepts

### 1. Exploring GIS Services

#### What are GIS Web Services?
GIS web services are standardized ways to share spatial data over the internet:

- **WMS (Web Map Service)**: Serves map images
- **WFS (Web Feature Service)**: Serves vector data features
- **WCS (Web Coverage Service)**: Serves raster/coverage data
- **WMTS (Web Map Tile Service)**: Serves pre-rendered map tiles

#### Service Standards
- **OGC (Open Geospatial Consortium)**: International standards organization
- **REST API**: Modern web service architecture
- **SOAP**: Legacy web service protocol

#### Common Use Cases
- Web mapping applications
- Mobile GIS apps
- Desktop GIS software integration
- Data sharing between organizations

### 2. GeoServer Installation

#### System Requirements
- **Java**: OpenJDK 8 or 11 (recommended)
- **Memory**: Minimum 2GB RAM, 4GB+ recommended
- **Storage**: 1GB+ free space
- **Operating System**: Windows, Linux, or macOS

#### Installation Methods

**Option 1: Platform Independent Binary**
1. Download GeoServer from official website
2. Extract to desired directory
3. Set JAVA_HOME environment variable
4. Run startup script

**Option 2: Windows Installer**
1. Download Windows installer
2. Run installer with administrator privileges
3. Follow installation wizard
4. Service automatically configured

**Option 3: Web Archive (WAR)**
1. Download GeoServer WAR file
2. Deploy to existing Tomcat server
3. Configure data directory
4. Restart Tomcat

#### Basic Configuration
```bash
# Set GEOSERVER_DATA_DIR environment variable
export GEOSERVER_DATA_DIR=/path/to/data/directory

# Start GeoServer (Linux/Mac)
./startup.sh

# Start GeoServer (Windows)
startup.bat
```

#### First Access
1. Open web browser
2. Navigate to `http://localhost:8080/geoserver`
3. Login with default credentials:
   - Username: `admin`
   - Password: `geoserver`
4. **Important**: Change default password immediately

### 3. Understanding GeoServer Logs

#### Log File Locations
- **Default Location**: `GEOSERVER_DATA_DIR/logs/`
- **Main Log**: `geoserver.log`
- **Access Log**: `access.log` (if enabled)

#### Log Levels
- **ERROR**: Critical errors that need immediate attention
- **WARN**: Warning messages about potential issues
- **INFO**: General information about operations
- **DEBUG**: Detailed debugging information
- **TRACE**: Very detailed execution traces

#### Reading Log Files
```bash
# View recent log entries (Linux/Mac)
tail -f geoserver.log

# Search for errors
grep "ERROR" geoserver.log

# View logs by date
grep "2024-01-15" geoserver.log
```

#### Common Log Messages
```
# Successful startup
INFO [geoserver.GeoServer] - GeoServer startup complete

# Service request
INFO [ows.OWS] - Request: GetMap

# Data store connection
WARN [data.DataStore] - Connection pool exhausted

# Authentication failure
ERROR [security] - Authentication failed for user: testuser
```

#### Log Configuration
- **Location**: `GEOSERVER_DATA_DIR/logs/DEFAULT_LOGGING.properties`
- **Web Interface**: Global Settings → Logging
- **Log Level**: Adjust based on needs (INFO for production)
- **Log File Size**: Configure rotation settings

### 4. Register and Publish GIS Data

#### Data Store Types
- **Shapefile**: Vector data in file format
- **PostGIS**: PostgreSQL spatial database
- **GeoTIFF**: Raster data format
- **Directory of Shapefiles**: Multiple shapefiles
- **WFS**: External web feature service

#### Publishing Vector Data (Shapefile)

**Step 1: Create Workspace**
1. Navigate to "Workspaces" in admin interface
2. Click "Add new workspace"
3. Enter workspace details:
   - Name: `tutorial`
   - Namespace URI: `http://tutorial.geoserver.org`
4. Save workspace

**Step 2: Add Data Store**
1. Go to "Stores" → "Add new Store"
2. Select "Shapefile" under Vector Data Sources
3. Configure store:
   - Workspace: `tutorial`
   - Data Source Name: `countries`
   - Shapefile location: Browse to .shp file
4. Save data store

**Step 3: Publish Layer**
1. Click "Publish" next to detected layer
2. Configure layer settings:
   - Name: Keep default or customize
   - Title: Human-readable title
   - Abstract: Layer description
3. Set bounding boxes:
   - Click "Compute from data"
   - Click "Compute from native bounds"
4. Save layer

#### Publishing Raster Data (GeoTIFF)

**Step 1: Add Coverage Store**
1. Go to "Stores" → "Add new Store"
2. Select "GeoTIFF" under Raster Data Sources
3. Configure store:
   - Workspace: `tutorial`
   - Data Source Name: `elevation`
   - URL: Browse to .tif file
4. Save coverage store

**Step 2: Publish Coverage**
1. Click "Publish" next to detected coverage
2. Configure coverage settings:
   - Name and title
   - Coordinate reference system
   - Coverage parameters
3. Save coverage

#### Testing Published Services

**Layer Preview**
1. Navigate to "Layer Preview"
2. Find your published layer
3. Click "OpenLayers" to view in browser
4. Test zoom, pan, and identify tools

**Service Capabilities**
```
# WMS Capabilities
http://localhost:8080/geoserver/tutorial/wms?request=GetCapabilities

# WFS Capabilities
http://localhost:8080/geoserver/tutorial/wfs?request=GetCapabilities

# Sample WMS GetMap request
http://localhost:8080/geoserver/tutorial/wms?
  service=WMS&version=1.1.0&request=GetMap&
  layers=tutorial:countries&
  bbox=-180,-90,180,90&
  width=800&height=400&
  srs=EPSG:4326&
  format=image/png
```

## Hands-On Examples

### Example 1: Installing GeoServer on Windows

**Download and Setup**
1. Visit `http://geoserver.org/download/`
2. Download "Platform Independent Binary"
3. Extract to `C:\geoserver`
4. Open Command Prompt as Administrator
5. Set Java path:
   ```cmd
   set JAVA_HOME=C:\Program Files\Java\jdk-11.0.x
   ```
6. Navigate to GeoServer directory:
   ```cmd
   cd C:\geoserver\bin
   ```
7. Start GeoServer:
   ```cmd
   startup.bat
   ```
8. Open browser to `http://localhost:8080/geoserver`

**Initial Configuration**
1. Login with admin/geoserver
2. Go to "Security" → "Users, Groups, Roles"
3. Click "Users" tab
4. Edit "admin" user
5. Change password to secure value
6. Save changes

### Example 2: Publishing Sample Data

**Download Sample Data**
1. Download Natural Earth data:
   - Countries: `ne_110m_admin_0_countries.shp`
   - Cities: `ne_110m_populated_places.shp`
2. Create folder: `C:\geoserver_data\sample_data`
3. Extract shapefiles to this folder

**Create Workspace**
1. In GeoServer admin: "Workspaces" → "Add new workspace"
2. Name: `naturalearth`
3. Namespace URI: `http://naturalearth.geoserver.org`
4. Save

**Add Countries Layer**
1. "Stores" → "Add new Store" → "Shapefile"
2. Workspace: `naturalearth`
3. Data Source Name: `countries`
4. Shapefile location: Browse to countries.shp
5. Save
6. Click "Publish" for detected layer
7. Configure:
   - Title: "World Countries"
   - Abstract: "Natural Earth country boundaries"
8. Compute bounding boxes
9. Save

**Test the Service**
1. Go to "Layer Preview"
2. Find "naturalearth:countries"
3. Click "OpenLayers"
4. Verify map displays correctly
5. Test zoom and pan functionality

### Example 3: Basic Log Monitoring

**Enable Detailed Logging**
1. Go to "Global" → "Logging"
2. Set logging level to "INFO"
3. Enable "Log to StdOut"
4. Save configuration

**Monitor Logs**
1. Open Command Prompt
2. Navigate to logs directory:
   ```cmd
   cd C:\geoserver\data_dir\logs
   ```
3. View current log:
   ```cmd
   type geoserver.log
   ```
4. Monitor real-time (PowerShell):
   ```powershell
   Get-Content geoserver.log -Wait -Tail 10
   ```

**Common Issues to Watch**
- Memory warnings
- Database connection errors
- Authentication failures
- Slow query performance
- Service request errors

## Best Practices

### ✅ Installation and Setup
- **Use dedicated user account**: Don't run as administrator/root
- **Set appropriate memory**: Configure JVM heap size based on usage
- **Secure installation**: Change default passwords immediately
- **Regular backups**: Backup data directory and configuration
- **Monitor disk space**: Ensure adequate space for logs and data
- **Use stable Java version**: Stick to LTS Java releases

### ✅ Data Management
- **Organize workspaces**: Group related data logically
- **Use descriptive names**: Clear layer and store names
- **Set proper metadata**: Title, abstract, and keywords
- **Validate data**: Check coordinate systems and geometry
- **Optimize file formats**: Use appropriate formats for data type
- **Document data sources**: Keep track of data origins

### ✅ Service Configuration
- **Test services**: Verify all published layers work correctly
- **Set appropriate limits**: Configure service limits for performance
- **Use standard projections**: Stick to common coordinate systems
- **Enable caching**: Configure tile caching for better performance
- **Monitor service usage**: Track which services are used most
- **Regular maintenance**: Keep services updated and optimized

### ❌ Common Mistakes
- **Using default passwords**: Major security risk
- **Ignoring log files**: Missing important error messages
- **Poor workspace organization**: Makes management difficult
- **Not testing services**: Publishing broken or slow services
- **Inadequate documentation**: No metadata or descriptions
- **Mixing coordinate systems**: Causes display and analysis errors
- **Overloading server**: Publishing too much data without optimization

## Simple Practice Projects

### Project 1: Basic GeoServer Setup
1. Install GeoServer on your system
2. Change default admin password
3. Create a workspace called "practice"
4. Configure logging to INFO level
5. Document the installation process

### Project 2: Publish Local Data
1. Find or download sample GIS data (shapefiles)
2. Create appropriate workspace and data store
3. Publish at least 2 different layers
4. Test services using Layer Preview
5. Document service URLs and capabilities

### Project 3: Log Analysis
1. Generate some service requests
2. Examine log files for these requests
3. Identify different types of log messages
4. Create a simple log monitoring procedure
5. Document common log patterns

### Project 4: Service Testing
1. Use published services in QGIS or other GIS software
2. Test WMS and WFS capabilities
3. Verify data displays correctly
4. Test different output formats
5. Document service performance

## Related Levels
- **Next**: [Level 2 - Styling and Caching](level-2.md)
- **Related**: [GIS Level 1](../gis/level-1.md)
- **Related**: [PostgreSQL/PostGIS Level 1](../postgresql-postgis/level-1.md)

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: What is GeoServer and why would I use it?</strong></summary>

**Answer**: GeoServer is an open-source server for sharing geospatial data:

**What it does**:
- Publishes spatial data as web services (WMS, WFS, WCS)
- Provides web-based administration interface
- Supports multiple data formats and databases
- Implements OGC standards for interoperability

**Why use it**:
- **Free and open source**: No licensing costs
- **Standards compliant**: Works with any OGC-compatible client
- **Flexible**: Supports many data sources and formats
- **Scalable**: Can handle enterprise-level deployments
- **Active community**: Good support and documentation

**Common use cases**:
- Web mapping applications
- Spatial data infrastructure
- Government data portals
- Environmental monitoring systems
- Urban planning applications
</details>

<details>
<summary><strong>Q2: How do I know if GeoServer is running correctly?</strong></summary>

**Answer**: Check multiple indicators:

**Web Interface Access**:
- Navigate to `http://localhost:8080/geoserver`
- Should see GeoServer welcome page
- Admin interface should be accessible

**Log File Indicators**:
```
# Successful startup message
INFO [geoserver.GeoServer] - GeoServer startup complete

# No critical errors
# Should not see repeated ERROR messages
```

**Service Capabilities**:
- WMS capabilities: `http://localhost:8080/geoserver/wms?request=GetCapabilities`
- Should return XML document without errors

**Layer Preview**:
- Access "Layer Preview" in admin interface
- Sample layers should display correctly
- No error messages in browser console

**System Resources**:
- Check memory usage (should be reasonable)
- Verify disk space availability
- Monitor CPU usage during requests
</details>

<details>
<summary><strong>Q3: What should I do if I can't access the GeoServer web interface?</strong></summary>

**Answer**: Troubleshoot systematically:

**Check Service Status**:
```bash
# Check if GeoServer process is running
# Windows
tasklist | findstr java

# Linux/Mac
ps aux | grep geoserver
```

**Verify Port Availability**:
```bash
# Check if port 8080 is in use
# Windows
netstat -an | findstr :8080

# Linux/Mac
lsof -i :8080
```

**Common Solutions**:
1. **Port conflict**: Change port in configuration
2. **Firewall blocking**: Configure firewall rules
3. **Java issues**: Verify JAVA_HOME is set correctly
4. **Permissions**: Ensure user has proper file permissions
5. **Memory issues**: Increase JVM heap size

**Check Logs**:
- Look for startup errors in `geoserver.log`
- Check system logs for Java errors
- Verify data directory permissions
</details>

<details>
<summary><strong>Q4: How do I change the default admin password?</strong></summary>

**Answer**: Change password through web interface:

**Steps**:
1. Login to GeoServer admin interface
2. Navigate to "Security" → "Users, Groups, Roles"
3. Click "Users" tab
4. Click "admin" user to edit
5. Enter new password in "Password" field
6. Confirm password in "Confirm password" field
7. Click "Save"

**Password Requirements**:
- Use strong password (8+ characters)
- Include uppercase, lowercase, numbers
- Avoid common words or patterns
- Don't reuse passwords from other systems

**Alternative Method** (if locked out):
1. Stop GeoServer
2. Edit `GEOSERVER_DATA_DIR/security/usergroup/default/users.xml`
3. Reset admin user entry
4. Restart GeoServer
5. Login and set new password

**Security Note**: Always change default passwords before production use.
</details>

<details>
<summary><strong>Q5: What's the difference between a workspace, store, and layer in GeoServer?</strong></summary>

**Answer**: These are hierarchical organizational concepts:

**Workspace**:
- **Purpose**: Top-level container for organizing related data
- **Contains**: Multiple data stores and layers
- **Example**: "environmental_data", "transportation", "cadastral"
- **Namespace**: Provides unique naming context

**Data Store**:
- **Purpose**: Connection to a specific data source
- **Contains**: Configuration for accessing data
- **Examples**: Shapefile directory, PostGIS database, GeoTIFF file
- **Types**: Vector stores, raster stores, coverage stores

**Layer**:
- **Purpose**: Individual dataset published as a service
- **Contains**: Styling, metadata, service configuration
- **Examples**: "countries", "roads", "elevation"
- **Services**: Available through WMS, WFS, or WCS

**Relationship**:
```
Workspace: environmental_data
├── Store: weather_stations (PostGIS)
│   ├── Layer: temperature_sensors
│   └── Layer: rainfall_gauges
└── Store: satellite_imagery (GeoTIFF)
    └── Layer: ndvi_2024
```

**Best Practice**: Organize logically by theme, project, or data source.
</details>

---

**Next Step**: Ready to advance your GeoServer skills? Continue to [Level 2](level-2.md) to learn about styling, caching, and advanced layer management.