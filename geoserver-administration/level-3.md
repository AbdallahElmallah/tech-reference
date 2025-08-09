# GeoServer Administration Level 3: Advanced Security and Performance

## Prerequisites
- Completed Level 2: Intermediate Operations
- Understanding of web security concepts (authentication, authorization)
- Basic knowledge of network protocols and firewalls
- Familiarity with performance monitoring tools
- Understanding of database optimization concepts
- Basic knowledge of Java application tuning

## Problem This Level Solves
This level teaches you how to secure your GeoServer deployment, troubleshoot common issues, and optimize performance for production environments, ensuring your services are robust, secure, and scalable.

## Key Concepts

### 1. Basic Security Implementation

#### Authentication Methods
- **Username/Password**: Default authentication mechanism
- **LDAP Integration**: Enterprise directory services
- **Database Authentication**: Custom user stores
- **Header Authentication**: Proxy-based authentication
- **OAuth2/OpenID Connect**: Modern authentication protocols

#### Authorization and Access Control
- **Role-Based Access Control (RBAC)**: Assign permissions to roles
- **Layer-Level Security**: Control access to specific layers
- **Service-Level Security**: Restrict WMS, WFS, WCS access
- **Data-Level Security**: Filter data based on user attributes
- **Administrative Security**: Protect configuration access

#### Security Best Practices
- **Change Default Credentials**: Never use admin/geoserver in production
- **Use HTTPS**: Encrypt all communications
- **Implement Firewall Rules**: Restrict network access
- **Regular Security Updates**: Keep GeoServer updated
- **Audit Logging**: Monitor access and changes

### 2. Basic Troubleshooting

#### Common Issues and Solutions
- **Service Unavailable**: Check logs, memory, and configuration
- **Slow Performance**: Analyze queries, caching, and resources
- **Authentication Failures**: Verify user credentials and configuration
- **Data Display Issues**: Check projections, styling, and data integrity
- **Memory Errors**: Tune JVM settings and optimize queries

#### Diagnostic Tools and Techniques
- **Log Analysis**: Systematic log file examination
- **Performance Monitoring**: Resource usage tracking
- **Network Diagnostics**: Connection and latency testing
- **Database Profiling**: Query performance analysis
- **Load Testing**: Stress testing under realistic conditions

#### Troubleshooting Methodology
1. **Identify Symptoms**: What exactly is not working?
2. **Gather Information**: Logs, configuration, environment
3. **Isolate Variables**: Test components individually
4. **Form Hypothesis**: Educated guess about root cause
5. **Test Solution**: Implement and verify fix
6. **Document Resolution**: Record for future reference

### 3. Basic Performance Tuning

#### JVM Optimization
- **Heap Size Configuration**: Allocate appropriate memory
- **Garbage Collection Tuning**: Optimize GC algorithms
- **JVM Parameters**: Configure for GeoServer workload
- **Memory Monitoring**: Track usage patterns
- **Performance Profiling**: Identify bottlenecks

#### Database Performance
- **Connection Pooling**: Optimize database connections
- **Query Optimization**: Improve spatial query performance
- **Indexing Strategy**: Create appropriate spatial indexes
- **Statistics Updates**: Maintain current database statistics
- **Connection Limits**: Configure appropriate limits

#### Caching Optimization
- **Cache Hit Ratio**: Maximize cache effectiveness
- **Cache Size Management**: Balance memory and disk usage
- **Cache Invalidation**: Ensure data freshness
- **Preseeding Strategy**: Generate tiles proactively
- **CDN Integration**: Distribute cached content

## Hands-On Examples

### Example 1: Implementing Basic Security

#### Setting Up LDAP Authentication

**Configure LDAP Connection**
1. **Access Security Settings**:
   - Navigate to "Security" → "Authentication"
   - Click "Add new" authentication provider
   - Select "LDAP" from the list

2. **LDAP Configuration**:
   ```
   Name: company_ldap
   Server URL: ldap://ldap.company.com:389
   User DN Pattern: uid={0},ou=people,dc=company,dc=com
   User Filter: (objectClass=person)
   Group Search Base: ou=groups,dc=company,dc=com
   Group Filter: member={0}
   ```

3. **Test LDAP Connection**:
   ```bash
   # Test LDAP connectivity
   ldapsearch -x -H ldap://ldap.company.com:389 \
             -D "uid=testuser,ou=people,dc=company,dc=com" \
             -W -b "dc=company,dc=com" "(uid=testuser)"
   ```

4. **Configure Authentication Chain**:
   - Go to "Security" → "Authentication"
   - Add LDAP provider to authentication chain
   - Set order: LDAP first, then username/password
   - Save configuration

#### Setting Up Role-Based Access Control

**Create Roles**
1. **Define Roles**:
   - Navigate to "Security" → "Users, Groups, Roles"
   - Click "Roles" tab → "Add new role"
   - Create roles:
     ```
     ROLE_VIEWER: Read-only access to public layers
     ROLE_EDITOR: Edit access to specific workspaces
     ROLE_ADMIN: Full administrative access
     ```

2. **Assign Role Hierarchy**:
   ```
   ROLE_ADMIN > ROLE_EDITOR > ROLE_VIEWER
   ```

**Configure Layer Security**
1. **Access Data Security**:
   - Navigate to "Security" → "Data"
   - Click "Add new rule"

2. **Create Security Rules**:
   ```
   # Public data - anyone can read
   *.*.r = *
   
   # Sensitive data - only editors can read
   sensitive.*.r = ROLE_EDITOR
   
   # Administrative data - only admins
   admin.*.r = ROLE_ADMIN
   admin.*.w = ROLE_ADMIN
   
   # Workspace-specific access
   planning.*.r = ROLE_PLANNER
   planning.*.w = ROLE_PLANNER
   ```

3. **Test Security Rules**:
   - Create test users with different roles
   - Verify access restrictions work correctly
   - Test both web interface and service access

### Example 2: Advanced Troubleshooting

#### Diagnosing Memory Issues

**Monitor Memory Usage**
```bash
# Check Java process memory
jps -l | grep geoserver
jstat -gc <pid> 5s

# Monitor heap usage
jmap -histo <pid> | head -20

# Generate heap dump for analysis
jmap -dump:format=b,file=geoserver_heap.hprof <pid>
```

**Analyze GeoServer Logs**
```bash
# Look for memory-related errors
grep -i "outofmemory\|heap\|gc" geoserver.log

# Check for long-running requests
grep "took [0-9]\{4,\} ms" geoserver.log

# Monitor request patterns
awk '/GetMap|GetFeature/ {print $1, $2, $7}' access.log | sort | uniq -c
```

**Common Memory Issues and Solutions**

1. **OutOfMemoryError: Java heap space**
   ```bash
   # Increase heap size
   export JAVA_OPTS="-Xms2g -Xmx8g"
   
   # Add to startup script
   JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
   JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
   ```

2. **OutOfMemoryError: PermGen space** (Java 7)
   ```bash
   # Increase PermGen size
   export JAVA_OPTS="-XX:PermSize=256m -XX:MaxPermSize=512m"
   ```

3. **OutOfMemoryError: Metaspace** (Java 8+)
   ```bash
   # Configure Metaspace
   export JAVA_OPTS="-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"
   ```

#### Database Connection Issues

**Diagnose Connection Problems**
```sql
-- Check active connections (PostgreSQL)
SELECT count(*) as active_connections,
       state,
       application_name
FROM pg_stat_activity
WHERE application_name LIKE '%geoserver%'
GROUP BY state, application_name;

-- Check connection pool status
SELECT * FROM pg_stat_database WHERE datname = 'gis_data';

-- Monitor long-running queries
SELECT pid, now() - pg_stat_activity.query_start AS duration,
       query, state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
```

**Optimize Connection Pool**
```xml
<!-- datastore.xml configuration -->
<entry key="max connections">20</entry>
<entry key="min connections">5</entry>
<entry key="connection timeout">20</entry>
<entry key="validate connections">true</entry>
<entry key="test while idle">true</entry>
<entry key="time between eviction runs">300</entry>
```

### Example 3: Performance Optimization

#### JVM Tuning for Production

**Optimal JVM Configuration**
```bash
#!/bin/bash
# geoserver_production.sh

# Memory settings
JAVA_OPTS="-Xms4g -Xmx8g"

# Garbage Collection (G1GC for large heaps)
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
JAVA_OPTS="$JAVA_OPTS -XX:G1HeapRegionSize=16m"

# GC Logging
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGC"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCTimeStamps"
JAVA_OPTS="$JAVA_OPTS -Xloggc:$GEOSERVER_DATA_DIR/logs/gc.log"

# Performance monitoring
JAVA_OPTS="$JAVA_OPTS -XX:+PrintStringDeduplicationStatistics"
JAVA_OPTS="$JAVA_OPTS -XX:+UseStringDeduplication"

# Network performance
JAVA_OPTS="$JAVA_OPTS -Djava.net.preferIPv4Stack=true"
JAVA_OPTS="$JAVA_OPTS -Dsun.net.useExclusiveBind=false"

# GeoServer specific
JAVA_OPTS="$JAVA_OPTS -DGEOSERVER_DATA_DIR=$GEOSERVER_DATA_DIR"
JAVA_OPTS="$JAVA_OPTS -Dorg.geotools.shapefile.datetime=true"

export JAVA_OPTS
```

#### Database Performance Tuning

**PostgreSQL/PostGIS Optimization**
```sql
-- Create spatial indexes
CREATE INDEX idx_countries_geom ON countries USING GIST (geom);
CREATE INDEX idx_cities_geom ON cities USING GIST (geom);

-- Analyze tables for query planner
ANALYZE countries;
ANALYZE cities;

-- Vacuum tables regularly
VACUUM ANALYZE countries;
VACUUM ANALYZE cities;

-- Check index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;
```

**PostgreSQL Configuration** (`postgresql.conf`):
```ini
# Memory settings
shared_buffers = 2GB
effective_cache_size = 6GB
work_mem = 256MB
maintenance_work_mem = 1GB

# Connection settings
max_connections = 200

# Query planner
random_page_cost = 1.1
effective_io_concurrency = 200

# WAL settings
wal_buffers = 64MB
checkpoint_completion_target = 0.9

# Logging
log_min_duration_statement = 1000
log_checkpoints = on
log_connections = on
log_disconnections = on
```

#### Application-Level Optimization

**GeoServer Configuration Tuning**
```xml
<!-- global.xml optimizations -->
<global>
  <settings>
    <!-- Increase service limits -->
    <maxFeatures>50000</maxFeatures>
    <maxCoordinates>1000000</maxCoordinates>
    
    <!-- Enable feature type caching -->
    <featureTypeCacheSize>10000</featureTypeCacheSize>
    
    <!-- Optimize rendering -->
    <numDecimals>6</numDecimals>
    <charset>UTF-8</charset>
    
    <!-- Resource limits -->
    <resourceErrorHandling>SKIP_MISCONFIGURED_LAYERS</resourceErrorHandling>
  </settings>
  
  <!-- JAI settings for image processing -->
  <jai>
    <allowInterpolation>true</allowInterpolation>
    <recycling>true</recycling>
    <tilePriority>5</tilePriority>
    <tileThreads>7</tileThreads>
    <memoryCapacity>0.75</memoryCapacity>
    <memoryThreshold>0.75</memoryThreshold>
  </jai>
</global>
```

**WMS Performance Settings**
```xml
<!-- wms.xml configuration -->
<wms>
  <maxBuffer>50</maxBuffer>
  <maxRequestMemory>65536</maxRequestMemory>
  <maxRenderingTime>120</maxRenderingTime>
  <maxRenderingErrors>1000</maxRenderingErrors>
  
  <!-- Enable advanced projection handling -->
  <advancedProjectionHandling>true</advancedProjectionHandling>
  
  <!-- Optimize for large datasets -->
  <maxFeatures>1000000</maxFeatures>
</wms>
```

### Example 4: Monitoring and Alerting Setup

#### Performance Monitoring Script
```bash
#!/bin/bash
# geoserver_monitor.sh

LOG_FILE="/var/log/geoserver_monitor.log"
THRESHOLD_CPU=80
THRESHOLD_MEMORY=85
THRESHOLD_RESPONSE=5000

# Get GeoServer process ID
GS_PID=$(pgrep -f geoserver)

if [ -z "$GS_PID" ]; then
    echo "$(date): ERROR - GeoServer process not found" >> $LOG_FILE
    exit 1
fi

# Check CPU usage
CPU_USAGE=$(ps -p $GS_PID -o %cpu --no-headers | awk '{print int($1)}')
if [ $CPU_USAGE -gt $THRESHOLD_CPU ]; then
    echo "$(date): WARNING - High CPU usage: ${CPU_USAGE}%" >> $LOG_FILE
fi

# Check memory usage
MEM_USAGE=$(ps -p $GS_PID -o %mem --no-headers | awk '{print int($1)}')
if [ $MEM_USAGE -gt $THRESHOLD_MEMORY ]; then
    echo "$(date): WARNING - High memory usage: ${MEM_USAGE}%" >> $LOG_FILE
fi

# Check service response time
RESPONSE_TIME=$(curl -w "%{time_total}" -s -o /dev/null \
    "http://localhost:8080/geoserver/wms?request=GetCapabilities" | \
    awk '{print int($1*1000)}')

if [ $RESPONSE_TIME -gt $THRESHOLD_RESPONSE ]; then
    echo "$(date): WARNING - Slow response time: ${RESPONSE_TIME}ms" >> $LOG_FILE
fi

# Check disk space
DISK_USAGE=$(df $GEOSERVER_DATA_DIR | awk 'NR==2 {print int($5)}')
if [ $DISK_USAGE -gt 90 ]; then
    echo "$(date): WARNING - Low disk space: ${DISK_USAGE}% used" >> $LOG_FILE
fi

echo "$(date): INFO - Monitor check completed" >> $LOG_FILE
```

#### Log Analysis and Alerting
```bash
#!/bin/bash
# log_analyzer.sh

LOG_DIR="$GEOSERVER_DATA_DIR/logs"
ALERT_EMAIL="admin@company.com"
ERROR_THRESHOLD=10

# Count errors in last hour
ERROR_COUNT=$(grep "$(date -d '1 hour ago' '+%Y-%m-%d %H')" \
    $LOG_DIR/geoserver.log | grep -c "ERROR")

if [ $ERROR_COUNT -gt $ERROR_THRESHOLD ]; then
    # Send alert email
    echo "GeoServer error threshold exceeded: $ERROR_COUNT errors in last hour" | \
        mail -s "GeoServer Alert" $ALERT_EMAIL
fi

# Check for specific error patterns
OOM_ERRORS=$(grep "OutOfMemoryError" $LOG_DIR/geoserver.log | \
    grep "$(date '+%Y-%m-%d')" | wc -l)

if [ $OOM_ERRORS -gt 0 ]; then
    echo "OutOfMemoryError detected: $OOM_ERRORS occurrences today" | \
        mail -s "GeoServer Memory Alert" $ALERT_EMAIL
fi

# Monitor slow queries
SLOW_QUERIES=$(grep "took [0-9]\{4,\} ms" $LOG_DIR/geoserver.log | \
    grep "$(date '+%Y-%m-%d')" | wc -l)

if [ $SLOW_QUERIES -gt 5 ]; then
    echo "Slow queries detected: $SLOW_QUERIES queries > 1 second today" | \
        mail -s "GeoServer Performance Alert" $ALERT_EMAIL
fi
```

## Best Practices

### ✅ Security Best Practices
- **Defense in Depth**: Multiple security layers (network, application, data)
- **Principle of Least Privilege**: Grant minimum necessary permissions
- **Regular Security Audits**: Review access logs and permissions
- **Secure Configuration**: Disable unnecessary services and features
- **Update Management**: Keep GeoServer and dependencies current
- **Backup Security**: Encrypt and secure backup files
- **Network Segmentation**: Isolate GeoServer from public networks
- **SSL/TLS Encryption**: Encrypt all communications

### ✅ Troubleshooting Best Practices
- **Systematic Approach**: Follow consistent troubleshooting methodology
- **Documentation**: Record issues, solutions, and lessons learned
- **Monitoring**: Implement proactive monitoring and alerting
- **Log Management**: Centralize and analyze log files regularly
- **Testing Environment**: Reproduce issues in non-production environment
- **Version Control**: Track configuration changes
- **Escalation Procedures**: Define when and how to escalate issues
- **Knowledge Sharing**: Maintain team knowledge base

### ✅ Performance Best Practices
- **Baseline Establishment**: Measure performance before optimization
- **Incremental Changes**: Make one change at a time
- **Load Testing**: Test under realistic conditions
- **Resource Monitoring**: Track CPU, memory, disk, and network
- **Database Optimization**: Maintain indexes and statistics
- **Caching Strategy**: Implement appropriate caching levels
- **Capacity Planning**: Plan for growth and peak usage
- **Regular Maintenance**: Schedule routine optimization tasks

### ❌ Common Mistakes
- **Ignoring Security**: Using default credentials and configurations
- **Reactive Troubleshooting**: Waiting for problems instead of monitoring
- **Over-optimization**: Making unnecessary changes that add complexity
- **Poor Documentation**: Not recording configuration changes
- **Inadequate Testing**: Not testing changes in realistic environments
- **Resource Starvation**: Under-provisioning hardware resources
- **Neglecting Maintenance**: Skipping regular updates and optimization
- **Single Point of Failure**: Not implementing redundancy and backup

## Advanced Practice Projects

### Project 1: Enterprise Security Implementation
1. Design and implement comprehensive security architecture
2. Integrate with enterprise LDAP/Active Directory
3. Configure fine-grained access controls
4. Implement audit logging and monitoring
5. Create security incident response procedures

### Project 2: Performance Optimization Suite
1. Establish performance baselines and monitoring
2. Implement comprehensive caching strategy
3. Optimize database connections and queries
4. Tune JVM and application settings
5. Create automated performance testing

### Project 3: High Availability Setup
1. Design clustered GeoServer deployment
2. Implement load balancing and failover
3. Configure shared data storage
4. Create disaster recovery procedures
5. Test failover and recovery scenarios

### Project 4: Monitoring and Alerting System
1. Implement comprehensive monitoring solution
2. Create automated alerting for critical issues
3. Develop performance dashboards
4. Establish SLA monitoring and reporting
5. Create operational runbooks

## Related Levels
- **Previous**: [Level 2 - Styling and Caching](level-2.md)
- **Related**: [GIS Level 3](../gis/level-3.md)
- **Related**: [PostgreSQL/PostGIS Level 3](../postgresql-postgis/level-3.md)
- **Related**: [System Administration](../system-administration/overview.md)

## Q&A Section

### Basic Questions

<details>
<summary><strong>Q1: How do I secure GeoServer for production use?</strong></summary>

**Answer**: Implement multiple security layers:

**Essential Security Steps**:
1. **Change Default Credentials**:
   ```
   - Never use admin/geoserver in production
   - Use strong passwords (12+ characters)
   - Consider multi-factor authentication
   ```

2. **Enable HTTPS**:
   ```bash
   # Configure SSL certificate
   keytool -genkey -alias geoserver -keyalg RSA \
           -keystore geoserver.jks -keysize 2048
   
   # Update server configuration
   # Add SSL connector to server.xml (Tomcat)
   ```

3. **Configure Firewall**:
   ```bash
   # Allow only necessary ports
   ufw allow 443/tcp  # HTTPS
   ufw allow 22/tcp   # SSH (from specific IPs)
   ufw deny 8080/tcp  # Block direct Tomcat access
   ```

4. **Implement Access Controls**:
   - Create role-based permissions
   - Restrict administrative access
   - Use layer-level security rules
   - Enable audit logging

5. **Network Security**:
   - Use reverse proxy (nginx/Apache)
   - Implement rate limiting
   - Hide server version information
   - Use VPN for administrative access

**Security Checklist**:
- [ ] Default passwords changed
- [ ] HTTPS enabled
- [ ] Firewall configured
- [ ] Access controls implemented
- [ ] Audit logging enabled
- [ ] Regular security updates scheduled
</details>

<details>
<summary><strong>Q2: What are the most common GeoServer performance issues?</strong></summary>

**Answer**: Identify and address common bottlenecks:

**Memory Issues**:
- **Symptoms**: OutOfMemoryError, slow response times
- **Causes**: Insufficient heap size, memory leaks, large datasets
- **Solutions**: Increase heap size, optimize queries, implement caching

**Database Performance**:
- **Symptoms**: Slow layer loading, timeout errors
- **Causes**: Missing indexes, poor queries, connection limits
- **Solutions**: Create spatial indexes, optimize connection pools

**Rendering Performance**:
- **Symptoms**: Slow map generation, high CPU usage
- **Causes**: Complex styling, large geometries, no caching
- **Solutions**: Simplify styles, implement tile caching, optimize data

**Network Bottlenecks**:
- **Symptoms**: Slow client response, high bandwidth usage
- **Causes**: Large responses, no compression, poor caching
- **Solutions**: Enable compression, implement CDN, optimize formats

**Common Performance Fixes**:
```bash
# Increase JVM heap
export JAVA_OPTS="-Xms2g -Xmx8g"

# Enable tile caching
# Configure in GeoWebCache interface

# Optimize database connections
# Set appropriate pool sizes in datastore configuration

# Enable response compression
# Configure in web.xml or reverse proxy
```
</details>

<details>
<summary><strong>Q3: How do I troubleshoot "Service Unavailable" errors?</strong></summary>

**Answer**: Follow systematic diagnostic approach:

**Step 1: Check Service Status**
```bash
# Check if GeoServer is running
ps aux | grep geoserver
sudo systemctl status geoserver

# Check port availability
netstat -tlnp | grep :8080
```

**Step 2: Examine Logs**
```bash
# Check startup logs
tail -f $GEOSERVER_DATA_DIR/logs/geoserver.log

# Look for specific errors
grep -i "error\|exception\|failed" geoserver.log

# Check system logs
sudo journalctl -u geoserver -f
```

**Step 3: Verify Configuration**
```bash
# Check data directory permissions
ls -la $GEOSERVER_DATA_DIR

# Verify Java installation
java -version
echo $JAVA_HOME

# Check available memory
free -h
df -h
```

**Common Causes and Solutions**:

1. **Out of Memory**:
   ```bash
   # Increase heap size
   export JAVA_OPTS="-Xms1g -Xmx4g"
   ```

2. **Port Conflict**:
   ```bash
   # Change port in server.xml
   <Connector port="8081" protocol="HTTP/1.1"/>
   ```

3. **Permission Issues**:
   ```bash
   # Fix data directory permissions
   sudo chown -R geoserver:geoserver $GEOSERVER_DATA_DIR
   ```

4. **Database Connection**:
   ```bash
   # Test database connectivity
   psql -h localhost -U geoserver -d gis_data
   ```

**Recovery Steps**:
1. Stop GeoServer service
2. Fix identified issue
3. Clear temporary files if needed
4. Restart service
5. Verify functionality
</details>

### Intermediate Questions

<details>
<summary><strong>Q4: How do I implement role-based security for different user groups?</strong></summary>

**Answer**: Design and implement comprehensive RBAC system:

**Step 1: Define Role Hierarchy**
```
ROLE_ADMIN
├── Full system access
├── User management
└── Configuration changes

ROLE_PUBLISHER
├── Publish/modify layers
├── Manage workspaces
└── Configure styles

ROLE_ANALYST
├── Access analytical tools
├── Download data
└── Create custom maps

ROLE_VIEWER
├── View public layers
├── Basic map navigation
└── Print maps
```

**Step 2: Configure Authentication**
```xml
<!-- security/config.xml -->
<authenticationProviders>
  <authenticationProvider>
    <name>ldap</name>
    <className>org.geoserver.security.ldap.LDAPAuthenticationProvider</className>
    <userGroupServiceName>ldap</userGroupServiceName>
  </authenticationProvider>
</authenticationProviders>
```

**Step 3: Create Security Rules**
```
# Data access rules (security/layers.properties)
# Format: workspace.layer.access=role1,role2

# Public data - everyone can read
*.*.r=*

# Sensitive data - restricted access
sensitive.*.r=ROLE_ANALYST,ROLE_ADMIN
sensitive.*.w=ROLE_ADMIN

# Administrative data - admin only
admin.*.r=ROLE_ADMIN
admin.*.w=ROLE_ADMIN

# Department-specific data
planning.*.r=ROLE_PLANNER,ROLE_ADMIN
planning.*.w=ROLE_PLANNER,ROLE_ADMIN

engineering.*.r=ROLE_ENGINEER,ROLE_ADMIN
engineering.*.w=ROLE_ENGINEER,ROLE_ADMIN
```

**Step 4: Service-Level Security**
```
# Service access rules (security/services.properties)
# Format: service.method=role1,role2

# WMS access
wms.GetMap=*
wms.GetFeatureInfo=*
wms.GetCapabilities=*

# WFS access - restricted
wfs.GetFeature=ROLE_ANALYST,ROLE_ADMIN
wfs.Transaction=ROLE_PUBLISHER,ROLE_ADMIN

# Administrative services
rest.**=ROLE_ADMIN
```

**Step 5: Test Security Implementation**
```bash
# Test with different user accounts
curl -u viewer:password \
  "http://localhost:8080/geoserver/wms?request=GetCapabilities"

curl -u analyst:password \
  "http://localhost:8080/geoserver/wfs?request=GetFeature&typeName=sensitive:data"

# Verify access denied for unauthorized users
curl -u viewer:password \
  "http://localhost:8080/geoserver/rest/workspaces"
```
</details>

<details>
<summary><strong>Q5: What JVM settings should I use for a production GeoServer?</strong></summary>

**Answer**: Optimize JVM for GeoServer workload characteristics:

**Memory Configuration**:
```bash
# For servers with 16GB+ RAM
JAVA_OPTS="-Xms4g -Xmx12g"

# For servers with 8GB RAM
JAVA_OPTS="-Xms2g -Xmx6g"

# For servers with 4GB RAM
JAVA_OPTS="-Xms1g -Xmx3g"
```

**Garbage Collection (Java 8+)**:
```bash
# G1GC for large heaps (>4GB)
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
JAVA_OPTS="$JAVA_OPTS -XX:G1HeapRegionSize=16m"
JAVA_OPTS="$JAVA_OPTS -XX:+UseStringDeduplication"

# Parallel GC for smaller heaps (<4GB)
# JAVA_OPTS="$JAVA_OPTS -XX:+UseParallelGC"
# JAVA_OPTS="$JAVA_OPTS -XX:ParallelGCThreads=4"
```

**Performance Tuning**:
```bash
# JIT compilation
JAVA_OPTS="$JAVA_OPTS -XX:+TieredCompilation"
JAVA_OPTS="$JAVA_OPTS -XX:TieredStopAtLevel=1"

# Network performance
JAVA_OPTS="$JAVA_OPTS -Djava.net.preferIPv4Stack=true"
JAVA_OPTS="$JAVA_OPTS -Dsun.net.useExclusiveBind=false"

# File I/O optimization
JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true"
JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF8"
```

**Monitoring and Debugging**:
```bash
# GC logging
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGC"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails"
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCTimeStamps"
JAVA_OPTS="$JAVA_OPTS -Xloggc:$GEOSERVER_DATA_DIR/logs/gc.log"
JAVA_OPTS="$JAVA_OPTS -XX:+UseGCLogFileRotation"
JAVA_OPTS="$JAVA_OPTS -XX:NumberOfGCLogFiles=5"
JAVA_OPTS="$JAVA_OPTS -XX:GCLogFileSize=10M"

# JMX monitoring
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.port=9999"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.authenticate=false"
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote.ssl=false"
```

**GeoServer-Specific Settings**:
```bash
# Data directory
JAVA_OPTS="$JAVA_OPTS -DGEOSERVER_DATA_DIR=$GEOSERVER_DATA_DIR"

# Shapefile optimization
JAVA_OPTS="$JAVA_OPTS -Dorg.geotools.shapefile.datetime=true"

# Image processing
JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true"
JAVA_OPTS="$JAVA_OPTS -Dorg.geotools.coverage.jaiext.enabled=true"

# Security
JAVA_OPTS="$JAVA_OPTS -Djava.security.egd=file:/dev/./urandom"
```

**Complete Production Configuration**:
```bash
#!/bin/bash
# production_java_opts.sh

# Memory (adjust based on available RAM)
JAVA_OPTS="-Xms4g -Xmx12g"

# Garbage Collection
JAVA_OPTS="$JAVA_OPTS -XX:+UseG1GC"
JAVA_OPTS="$JAVA_OPTS -XX:MaxGCPauseMillis=200"
JAVA_OPTS="$JAVA_OPTS -XX:G1HeapRegionSize=16m"
JAVA_OPTS="$JAVA_OPTS -XX:+UseStringDeduplication"

# Performance
JAVA_OPTS="$JAVA_OPTS -XX:+TieredCompilation"
JAVA_OPTS="$JAVA_OPTS -Djava.net.preferIPv4Stack=true"
JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true"

# Monitoring
JAVA_OPTS="$JAVA_OPTS -XX:+PrintGCDetails"
JAVA_OPTS="$JAVA_OPTS -Xloggc:$GEOSERVER_DATA_DIR/logs/gc.log"

# GeoServer specific
JAVA_OPTS="$JAVA_OPTS -DGEOSERVER_DATA_DIR=$GEOSERVER_DATA_DIR"
JAVA_OPTS="$JAVA_OPTS -Dorg.geotools.shapefile.datetime=true"

export JAVA_OPTS
```
</details>

<details>
<summary><strong>Q6: How do I set up monitoring and alerting for production GeoServer?</strong></summary>

**Answer**: Implement comprehensive monitoring strategy:

**System-Level Monitoring**:
```bash
# Install monitoring tools
sudo apt-get install htop iotop nethogs

# Monitor GeoServer process
ps aux | grep geoserver
top -p $(pgrep -f geoserver)

# Monitor memory usage
free -h
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable"

# Monitor disk usage
df -h $GEOSERVER_DATA_DIR
du -sh $GEOSERVER_DATA_DIR/gwc
```

**Application Monitoring**:
```bash
# JVM monitoring with jstat
jstat -gc $(pgrep -f geoserver) 5s

# Heap analysis
jmap -histo $(pgrep -f geoserver) | head -20

# Thread analysis
jstack $(pgrep -f geoserver) > geoserver_threads.txt
```

**Service Health Checks**:
```bash
#!/bin/bash
# health_check.sh

GS_URL="http://localhost:8080/geoserver"
TIMEOUT=30
ALERT_EMAIL="admin@company.com"

# Check service availability
if ! curl -f -s --max-time $TIMEOUT "$GS_URL/web/" > /dev/null; then
    echo "GeoServer web interface unavailable" | \
        mail -s "GeoServer Down" $ALERT_EMAIL
    exit 1
fi

# Check WMS service
if ! curl -f -s --max-time $TIMEOUT \
    "$GS_URL/wms?request=GetCapabilities" > /dev/null; then
    echo "WMS service unavailable" | \
        mail -s "GeoServer WMS Down" $ALERT_EMAIL
    exit 1
fi

# Check response time
RESPONSE_TIME=$(curl -w "%{time_total}" -s -o /dev/null \
    "$GS_URL/wms?request=GetCapabilities")

if (( $(echo "$RESPONSE_TIME > 5.0" | bc -l) )); then
    echo "Slow response time: ${RESPONSE_TIME}s" | \
        mail -s "GeoServer Performance Alert" $ALERT_EMAIL
fi

echo "Health check passed at $(date)"
```

**Log Monitoring**:
```bash
#!/bin/bash
# log_monitor.sh

LOG_FILE="$GEOSERVER_DATA_DIR/logs/geoserver.log"
ERROR_THRESHOLD=5
WARN_THRESHOLD=20
ALERT_EMAIL="admin@company.com"

# Count recent errors
ERRORS=$(grep "$(date '+%Y-%m-%d %H')" "$LOG_FILE" | grep -c "ERROR")
WARNINGS=$(grep "$(date '+%Y-%m-%d %H')" "$LOG_FILE" | grep -c "WARN")

if [ $ERRORS -gt $ERROR_THRESHOLD ]; then
    echo "High error rate: $ERRORS errors in last hour" | \
        mail -s "GeoServer Error Alert" $ALERT_EMAIL
fi

if [ $WARNINGS -gt $WARN_THRESHOLD ]; then
    echo "High warning rate: $WARNINGS warnings in last hour" | \
        mail -s "GeoServer Warning Alert" $ALERT_EMAIL
fi

# Check for specific issues
OOM_COUNT=$(grep "OutOfMemoryError" "$LOG_FILE" | \
    grep "$(date '+%Y-%m-%d')" | wc -l)

if [ $OOM_COUNT -gt 0 ]; then
    echo "OutOfMemoryError detected: $OOM_COUNT occurrences today" | \
        mail -s "GeoServer Memory Alert" $ALERT_EMAIL
fi
```

**Performance Dashboard Script**:
```bash
#!/bin/bash
# performance_dashboard.sh

GS_PID=$(pgrep -f geoserver)
DATA_DIR="$GEOSERVER_DATA_DIR"
OUTPUT_FILE="/var/www/html/geoserver_status.html"

# Generate HTML dashboard
cat > $OUTPUT_FILE << EOF
<!DOCTYPE html>
<html>
<head>
    <title>GeoServer Status Dashboard</title>
    <meta http-equiv="refresh" content="60">
</head>
<body>
    <h1>GeoServer Status - $(date)</h1>
    
    <h2>System Resources</h2>
    <p>CPU Usage: $(ps -p $GS_PID -o %cpu --no-headers)%</p>
    <p>Memory Usage: $(ps -p $GS_PID -o %mem --no-headers)%</p>
    <p>Disk Usage: $(df $DATA_DIR | awk 'NR==2 {print $5}')</p>
    
    <h2>Service Status</h2>
    <p>Process ID: $GS_PID</p>
    <p>Uptime: $(ps -p $GS_PID -o etime --no-headers)</p>
    
    <h2>Recent Errors</h2>
    <pre>
EOF

# Add recent errors
tail -20 $DATA_DIR/logs/geoserver.log | grep "ERROR" >> $OUTPUT_FILE

cat >> $OUTPUT_FILE << EOF
    </pre>
    
    <h2>Cache Statistics</h2>
    <p>Cache Size: $(du -sh $DATA_DIR/gwc 2>/dev/null | cut -f1)</p>
    
</body>
</html>
EOF

echo "Dashboard updated: $(date)"
```

**Automated Monitoring Setup**:
```bash
# Add to crontab (crontab -e)
# Check health every 5 minutes
*/5 * * * * /opt/scripts/health_check.sh

# Monitor logs every hour
0 * * * * /opt/scripts/log_monitor.sh

# Update dashboard every minute
* * * * * /opt/scripts/performance_dashboard.sh

# Daily performance report
0 6 * * * /opt/scripts/daily_report.sh
```
</details>

---

**Congratulations!** You've completed the GeoServer Administration learning path. You now have the skills to deploy, secure, and optimize GeoServer for production environments. Consider exploring advanced topics like clustering, custom extensions, and integration with other enterprise systems.