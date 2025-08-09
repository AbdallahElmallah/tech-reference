# Level 4: Advanced Docker - Tuning, Security, Troubleshooting

## Prerequisites
- Level 3 completion (Docker Compose, multi-container orchestration, scripting)
- Deep understanding of Linux systems and networking
- Experience with production environments
- Knowledge of security principles and best practices
- Familiarity with monitoring and observability tools

## Problem Statement
As a senior backend engineer, you need to:
- **Optimize Docker performance** for production workloads
- **Implement comprehensive security** measures and hardening
- **Troubleshoot complex issues** in containerized environments
- **Monitor and tune resource usage** effectively
- **Design resilient architectures** with proper fault tolerance
- **Implement advanced networking** and storage solutions
- **Ensure compliance** with security standards and regulations

These advanced skills are essential for running Docker in enterprise production environments.

---

## Key Concepts

### 1. Performance Tuning and Optimization

#### Container Resource Management
```bash
# CPU limits and reservations
docker run -d \
  --cpus="1.5" \
  --cpu-shares=1024 \
  --cpuset-cpus="0,1" \
  nginx

# Memory limits and swap
docker run -d \
  --memory="512m" \
  --memory-swap="1g" \
  --memory-swappiness=10 \
  --oom-kill-disable \
  nginx

# I/O limits
docker run -d \
  --device-read-bps /dev/sda:1mb \
  --device-write-bps /dev/sda:1mb \
  --device-read-iops /dev/sda:1000 \
  --device-write-iops /dev/sda:1000 \
  nginx

# Process limits
docker run -d \
  --pids-limit=100 \
  --ulimit nofile=65536:65536 \
  --ulimit nproc=4096:4096 \
  nginx
```

#### Docker Daemon Optimization
```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "live-restore": true,
  "userland-proxy": false,
  "experimental": false,
  "metrics-addr": "127.0.0.1:9323",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 65536,
      "Soft": 65536
    },
    "nproc": {
      "Name": "nproc",
      "Hard": 4096,
      "Soft": 4096
    }
  },
  "max-concurrent-downloads": 10,
  "max-concurrent-uploads": 5,
  "default-shm-size": "64M",
  "no-new-privileges": true
}
```

#### Advanced Dockerfile Optimization
```dockerfile
# Multi-stage build with optimization
FROM node:18-alpine AS base

# Install security updates
RUN apk update && apk upgrade && apk add --no-cache dumb-init

# Create app directory with proper permissions
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001 -G nodejs && \
    chown -R nextjs:nodejs /app

# Dependencies stage
FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production --no-audit --no-fund && \
    npm cache clean --force

# Build stage
FROM base AS builder
COPY package*.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build && \
    npm prune --production

# Production stage
FROM base AS runner

# Copy built application
COPY --from=deps --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/package.json ./

# Security hardening
RUN rm -rf /tmp/* /var/cache/apk/* && \
    chmod -R 755 /app && \
    find /app -type f -exec chmod 644 {} \;

USER nextjs

# Health check with timeout
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Use dumb-init for proper signal handling
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
```

#### Performance Monitoring
```bash
# Monitor container performance
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Detailed container inspection
docker inspect container_name | jq '.[] | {
  "Memory": .HostConfig.Memory,
  "CPUs": .HostConfig.NanoCpus,
  "PidsLimit": .HostConfig.PidsLimit,
  "RestartPolicy": .HostConfig.RestartPolicy
}'

# System-wide Docker metrics
curl http://localhost:9323/metrics

# Container process monitoring
docker exec container_name ps aux
docker exec container_name top

# Network performance
docker exec container_name ss -tuln
docker exec container_name netstat -i
```

### 2. Security Hardening

#### Container Security Best Practices
```dockerfile
# Security-hardened Dockerfile
FROM node:18-alpine AS base

# Install security updates and minimal tools
RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init curl && \
    rm -rf /var/cache/apk/*

# Create non-root user with specific UID/GID
RUN addgroup -g 10001 -S appgroup && \
    adduser -u 10001 -S appuser -G appgroup -h /app

WORKDIR /app

# Set strict file permissions
COPY --chown=appuser:appgroup package*.json ./
RUN npm ci --only=production --no-audit && \
    npm cache clean --force

COPY --chown=appuser:appgroup . .

# Remove unnecessary files and set permissions
RUN rm -rf .git .gitignore README.md docs/ tests/ && \
    find /app -type f -exec chmod 644 {} \; && \
    find /app -type d -exec chmod 755 {} \; && \
    chmod +x /app/entrypoint.sh

# Switch to non-root user
USER appuser

# Security labels
LABEL security.scan="enabled" \
      security.non-root="true" \
      security.no-new-privileges="true"

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

ENTRYPOINT ["dumb-init", "--"]
CMD ["./entrypoint.sh"]
```

#### Runtime Security Configuration
```bash
# Run with security options
docker run -d \
  --name secure-app \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=100m \
  --tmpfs /var/run:rw,noexec,nosuid,size=50m \
  --security-opt=no-new-privileges:true \
  --security-opt=apparmor:docker-default \
  --security-opt=seccomp:default \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --user 10001:10001 \
  --pids-limit=100 \
  --memory=512m \
  --cpus=1.0 \
  my-secure-app:latest

# Network security
docker run -d \
  --name isolated-app \
  --network=none \
  --add-host=api.internal:10.0.1.100 \
  my-app:latest

# Volume security
docker run -d \
  --name app-with-volumes \
  -v /host/data:/app/data:ro,Z \
  -v app-logs:/app/logs:rw,Z \
  --tmpfs /app/cache:rw,noexec,nosuid,size=100m \
  my-app:latest
```

#### Docker Compose Security
```yaml
version: '3.8'

services:
  web:
    build: .
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=100m
      - /var/run:rw,noexec,nosuid,size=50m
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
      - seccomp:default.json
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    user: "10001:10001"
    pids_limit: 100
    mem_limit: 512m
    cpus: 1.0
    networks:
      - frontend
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
      - jwt_secret
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

  database:
    image: postgres:15-alpine
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=100m
      - /var/run/postgresql:rw,noexec,nosuid,size=50m
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - SETUID
      - SETGID
      - DAC_OVERRIDE
    user: "999:999"
    networks:
      - backend
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres_data:/var/lib/postgresql/data:Z
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

secrets:
  db_password:
    external: true
  jwt_secret:
    external: true

volumes:
  postgres_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /secure/postgres-data

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24
  backend:
    driver: bridge
    internal: true
    ipam:
      config:
        - subnet: 172.21.0.0/24
```

#### Security Scanning and Compliance
```bash
# Vulnerability scanning with Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $HOME/Library/Caches:/root/.cache/ \
  aquasec/trivy image --severity HIGH,CRITICAL my-app:latest

# Generate security report
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/output \
  aquasec/trivy image --format json --output /output/security-report.json my-app:latest

# Dockerfile security scanning
docker run --rm -i hadolint/hadolint < Dockerfile

# Runtime security monitoring
docker run -d \
  --name falco \
  --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev \
  -v /proc:/host/proc:ro \
  -v /boot:/host/boot:ro \
  -v /lib/modules:/host/lib/modules:ro \
  -v /usr:/host/usr:ro \
  falcosecurity/falco:latest

# Container compliance checking
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  docker/docker-bench-security
```

### 3. Advanced Networking

#### Custom Network Configuration
```bash
# Create custom networks
docker network create \
  --driver bridge \
  --subnet=172.30.0.0/16 \
  --ip-range=172.30.240.0/20 \
  --gateway=172.30.0.1 \
  --opt com.docker.network.bridge.name=custom-bridge \
  --opt com.docker.network.bridge.enable_icc=false \
  --opt com.docker.network.bridge.enable_ip_masquerade=true \
  custom-network

# Overlay network for multi-host
docker network create \
  --driver overlay \
  --subnet=10.0.0.0/24 \
  --gateway=10.0.0.1 \
  --attachable \
  multi-host-network

# MACVLAN network for direct host access
docker network create \
  --driver macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  --opt parent=eth0 \
  macvlan-network
```

#### Network Security and Isolation
```yaml
# docker-compose.yml with network segmentation
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    networks:
      - public
      - frontend-backend
    ports:
      - "80:80"
      - "443:443"

  api:
    build: ./api
    networks:
      - frontend-backend
      - backend-database
    environment:
      - DATABASE_URL=postgresql://user:pass@database:5432/myapp

  database:
    image: postgres:15-alpine
    networks:
      - backend-database
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  redis:
    image: redis:7-alpine
    networks:
      - backend-database
    command: redis-server --requirepass ${REDIS_PASSWORD}

networks:
  public:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24
  frontend-backend:
    driver: bridge
    internal: false
    ipam:
      config:
        - subnet: 172.21.0.0/24
  backend-database:
    driver: bridge
    internal: true
    ipam:
      config:
        - subnet: 172.22.0.0/24
```

#### Load Balancing and Service Discovery
```yaml
# HAProxy load balancer configuration
version: '3.8'

services:
  haproxy:
    image: haproxy:2.6-alpine
    ports:
      - "80:80"
      - "443:443"
      - "8404:8404"  # Stats page
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
      - ./ssl:/etc/ssl/certs:ro
    networks:
      - frontend
    depends_on:
      - web1
      - web2
      - web3

  web1:
    build: .
    networks:
      - frontend
      - backend
    environment:
      - INSTANCE_ID=web1

  web2:
    build: .
    networks:
      - frontend
      - backend
    environment:
      - INSTANCE_ID=web2

  web3:
    build: .
    networks:
      - frontend
      - backend
    environment:
      - INSTANCE_ID=web3

  consul:
    image: consul:1.15
    command: agent -server -bootstrap-expect=1 -ui -client=0.0.0.0
    ports:
      - "8500:8500"
    networks:
      - backend
    volumes:
      - consul_data:/consul/data

volumes:
  consul_data:

networks:
  frontend:
  backend:
```

### 4. Advanced Storage and Data Management

#### Volume Optimization
```bash
# Create optimized volumes
docker volume create \
  --driver local \
  --opt type=ext4 \
  --opt device=/dev/sdb1 \
  --opt o=defaults,noatime \
  fast-storage

# NFS volume for shared storage
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=nfs-server.local,rw \
  --opt device=:/path/to/shared \
  shared-storage

# Encrypted volume
docker volume create \
  --driver local \
  --opt type=ext4 \
  --opt device=/dev/mapper/encrypted-volume \
  encrypted-storage
```

#### Backup and Recovery Strategies
```bash
#!/bin/bash
# backup-volumes.sh - Comprehensive backup script

BACKUP_DIR="/backups/docker"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

backup_volume() {
    local volume_name=$1
    local backup_file="$BACKUP_DIR/${volume_name}_${DATE}.tar.gz"
    
    log "Backing up volume: $volume_name"
    
    docker run --rm \
        -v "$volume_name:/data:ro" \
        -v "$BACKUP_DIR:/backup" \
        alpine:latest \
        tar czf "/backup/$(basename $backup_file)" -C /data .
    
    if [ $? -eq 0 ]; then
        log "Backup completed: $backup_file"
    else
        log "Backup failed for volume: $volume_name"
        return 1
    fi
}

backup_database() {
    local container_name=$1
    local db_name=$2
    local backup_file="$BACKUP_DIR/${db_name}_${DATE}.sql.gz"
    
    log "Backing up database: $db_name"
    
    docker exec "$container_name" pg_dump -U postgres "$db_name" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        log "Database backup completed: $backup_file"
    else
        log "Database backup failed: $db_name"
        return 1
    fi
}

cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days"
    find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete
    find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete
}

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Backup volumes
for volume in $(docker volume ls -q); do
    backup_volume "$volume"
done

# Backup databases
backup_database "postgres-container" "myapp"
backup_database "postgres-container" "analytics"

# Cleanup old backups
cleanup_old_backups

log "Backup process completed"
```

### 5. Monitoring and Observability

#### Comprehensive Monitoring Stack
```yaml
# monitoring-stack.yml
version: '3.8'

services:
  # Metrics collection
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./rules:/etc/prometheus/rules:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--storage.tsdb.retention.size=10GB'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    networks:
      - monitoring

  # Metrics visualization
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./grafana/datasources:/etc/grafana/provisioning/datasources:ro
    networks:
      - monitoring
    depends_on:
      - prometheus

  # Log aggregation
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./loki.yml:/etc/loki/local-config.yaml:ro
      - loki_data:/loki
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - monitoring

  # Log collection
  promtail:
    image: grafana/promtail:latest
    volumes:
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - ./promtail.yml:/etc/promtail/config.yml:ro
    command: -config.file=/etc/promtail/config.yml
    networks:
      - monitoring
    depends_on:
      - loki

  # Application performance monitoring
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14268:14268"
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    networks:
      - monitoring

  # Node exporter for host metrics
  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring

  # cAdvisor for container metrics
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    privileged: true
    devices:
      - /dev/kmsg
    networks:
      - monitoring

  # Alerting
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
      - alertmanager_data:/alertmanager
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
  alertmanager_data:

networks:
  monitoring:
    driver: bridge
```

#### Custom Metrics and Alerts
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "/etc/prometheus/rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'docker-daemon'
    static_configs:
      - targets: ['host.docker.internal:9323']

  - job_name: 'application'
    static_configs:
      - targets: ['web:3000', 'api:8080']
    metrics_path: '/metrics'
    scrape_interval: 5s
```

```yaml
# rules/docker-alerts.yml
groups:
  - name: docker-alerts
    rules:
      - alert: ContainerDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.instance }} is down"
          description: "Container {{ $labels.instance }} has been down for more than 1 minute."

      - alert: HighMemoryUsage
        expr: (container_memory_usage_bytes / container_spec_memory_limit_bytes) * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.name }}"
          description: "Container {{ $labels.name }} is using {{ $value }}% of its memory limit."

      - alert: HighCPUUsage
        expr: rate(container_cpu_usage_seconds_total[5m]) * 100 > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.name }}"
          description: "Container {{ $labels.name }} is using {{ $value }}% CPU."

      - alert: DiskSpaceRunningOut
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk space running out on {{ $labels.instance }}"
          description: "Disk space on {{ $labels.instance }} is below 10% ({{ $value }}% remaining)."
```

---

## Troubleshooting Complex Issues

### 1. Performance Debugging

#### Container Performance Analysis
```bash
#!/bin/bash
# performance-debug.sh - Comprehensive performance debugging

CONTAINER_NAME=$1

if [ -z "$CONTAINER_NAME" ]; then
    echo "Usage: $0 <container_name>"
    exit 1
fi

echo "=== Container Performance Analysis for $CONTAINER_NAME ==="

# Basic container info
echo "\n--- Container Info ---"
docker inspect "$CONTAINER_NAME" | jq '.[0] | {
    "State": .State.Status,
    "Memory": .HostConfig.Memory,
    "CPUs": .HostConfig.NanoCpus,
    "RestartCount": .RestartCount,
    "StartedAt": .State.StartedAt
}'

# Resource usage
echo "\n--- Resource Usage ---"
docker stats "$CONTAINER_NAME" --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}\t{{.NetIO}}\t{{.BlockIO}}"

# Process information
echo "\n--- Process Information ---"
docker exec "$CONTAINER_NAME" ps aux --sort=-%cpu | head -10

# Memory analysis
echo "\n--- Memory Analysis ---"
docker exec "$CONTAINER_NAME" cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|Buffers|Cached"

# Network connections
echo "\n--- Network Connections ---"
docker exec "$CONTAINER_NAME" ss -tuln | head -20

# Disk I/O
echo "\n--- Disk I/O ---"
docker exec "$CONTAINER_NAME" iostat -x 1 3 2>/dev/null || echo "iostat not available"

# Application-specific metrics (if available)
echo "\n--- Application Metrics ---"
curl -s "http://localhost:$(docker port $CONTAINER_NAME | grep -o '0.0.0.0:[0-9]*' | cut -d: -f2 | head -1)/metrics" | head -20 || echo "No metrics endpoint available"

# Recent logs
echo "\n--- Recent Logs (last 50 lines) ---"
docker logs --tail 50 "$CONTAINER_NAME"

echo "\n=== Analysis Complete ==="
```

#### Network Troubleshooting
```bash
#!/bin/bash
# network-debug.sh - Network connectivity debugging

SERVICE1=$1
SERVICE2=$2

if [ -z "$SERVICE1" ] || [ -z "$SERVICE2" ]; then
    echo "Usage: $0 <service1> <service2>"
    exit 1
fi

echo "=== Network Debugging: $SERVICE1 -> $SERVICE2 ==="

# Get container IPs
IP1=$(docker inspect "$SERVICE1" | jq -r '.[0].NetworkSettings.IPAddress')
IP2=$(docker inspect "$SERVICE2" | jq -r '.[0].NetworkSettings.IPAddress')

echo "$SERVICE1 IP: $IP1"
echo "$SERVICE2 IP: $IP2"

# Check network connectivity
echo "\n--- Ping Test ---"
docker exec "$SERVICE1" ping -c 3 "$SERVICE2" || echo "Ping failed"

# Check port connectivity
echo "\n--- Port Connectivity ---"
PORT=$(docker inspect "$SERVICE2" | jq -r '.[0].Config.ExposedPorts | keys[0]' | cut -d'/' -f1)
if [ "$PORT" != "null" ]; then
    docker exec "$SERVICE1" nc -zv "$SERVICE2" "$PORT" || echo "Port $PORT not reachable"
fi

# DNS resolution
echo "\n--- DNS Resolution ---"
docker exec "$SERVICE1" nslookup "$SERVICE2" || echo "DNS resolution failed"

# Network configuration
echo "\n--- Network Configuration ---"
docker exec "$SERVICE1" ip route show
docker exec "$SERVICE1" ip addr show

# Firewall rules (if iptables available)
echo "\n--- Firewall Rules ---"
docker exec "$SERVICE1" iptables -L 2>/dev/null || echo "iptables not available"

echo "\n=== Network Debug Complete ==="
```

### 2. Log Analysis and Debugging

#### Advanced Log Analysis
```bash
#!/bin/bash
# log-analyzer.sh - Advanced log analysis tool

CONTAINER=$1
TIME_RANGE=${2:-"1h"}

if [ -z "$CONTAINER" ]; then
    echo "Usage: $0 <container_name> [time_range]"
    exit 1
fi

echo "=== Log Analysis for $CONTAINER (last $TIME_RANGE) ==="

# Get logs with timestamps
LOGS=$(docker logs --since="$TIME_RANGE" -t "$CONTAINER" 2>&1)

# Error analysis
echo "\n--- Error Summary ---"
echo "$LOGS" | grep -i error | wc -l | xargs echo "Total errors:"
echo "$LOGS" | grep -i error | cut -d' ' -f3- | sort | uniq -c | sort -nr | head -10

# Warning analysis
echo "\n--- Warning Summary ---"
echo "$LOGS" | grep -i warn | wc -l | xargs echo "Total warnings:"
echo "$LOGS" | grep -i warn | cut -d' ' -f3- | sort | uniq -c | sort -nr | head -10

# Response time analysis (for web applications)
echo "\n--- Response Time Analysis ---"
echo "$LOGS" | grep -E "[0-9]+ms|[0-9]+s" | grep -oE "[0-9]+ms|[0-9]+s" | sort -n | tail -10

# Memory usage patterns
echo "\n--- Memory Usage Patterns ---"
echo "$LOGS" | grep -i "memory\|mem\|oom" | tail -10

# Database connection issues
echo "\n--- Database Connection Issues ---"
echo "$LOGS" | grep -i "database\|connection\|timeout" | tail -10

# Recent critical events
echo "\n--- Recent Critical Events ---"
echo "$LOGS" | grep -iE "critical|fatal|panic|exception" | tail -20

echo "\n=== Log Analysis Complete ==="
```

### 3. System-Level Troubleshooting

#### Docker Daemon Debugging
```bash
#!/bin/bash
# docker-system-debug.sh - System-level Docker debugging

echo "=== Docker System Debugging ==="

# Docker daemon status
echo "\n--- Docker Daemon Status ---"
systemctl status docker

# Docker system information
echo "\n--- Docker System Info ---"
docker system df
docker system events --since 1h --until now | tail -20

# Resource usage
echo "\n--- System Resource Usage ---"
df -h | grep -E "/$|/var"
free -h
top -bn1 | head -20

# Docker daemon logs
echo "\n--- Docker Daemon Logs (last 50 lines) ---"
journalctl -u docker --since "1 hour ago" --no-pager | tail -50

# Network issues
echo "\n--- Network Configuration ---"
ip addr show docker0
iptables -t nat -L DOCKER

# Storage driver issues
echo "\n--- Storage Driver Info ---"
docker info | grep -A 10 "Storage Driver"

# Container states
echo "\n--- Container States ---"
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"

echo "\n=== System Debug Complete ==="
```

---

## Best Practices for Production

### Security
1. **Use minimal base images** (alpine, distroless)
2. **Run as non-root users** with specific UIDs
3. **Implement proper secrets management**
4. **Regular security scanning** and updates
5. **Network segmentation** and isolation
6. **Resource limits** and quotas
7. **Read-only filesystems** where possible
8. **Security monitoring** and alerting

### Performance
1. **Optimize Dockerfile layers** for caching
2. **Use multi-stage builds** for smaller images
3. **Implement proper health checks**
4. **Monitor resource usage** continuously
5. **Use appropriate restart policies**
6. **Optimize storage drivers** and volumes
7. **Implement proper logging** strategies
8. **Regular performance testing**

### Reliability
1. **Implement circuit breakers** and retries
2. **Use proper dependency management**
3. **Implement graceful shutdowns**
4. **Regular backup strategies**
5. **Disaster recovery planning**
6. **Monitoring and alerting**
7. **Automated recovery procedures**
8. **Regular testing of failure scenarios**

---

## Common Mistakes

❌ **Running containers as root in production**
```dockerfile
# Don't do this in production
FROM ubuntu:latest
COPY app /app
CMD ["/app"]
```
✅ **Use non-root users**
```dockerfile
FROM ubuntu:latest
RUN useradd -r -s /bin/false appuser
COPY --chown=appuser:appuser app /app
USER appuser
CMD ["/app"]
```

❌ **No resource limits**
```yaml
services:
  web:
    image: myapp
    # No resource limits - can consume all host resources
```
✅ **Set appropriate resource limits**
```yaml
services:
  web:
    image: myapp
    mem_limit: 512m
    cpus: 1.0
    pids_limit: 100
```

❌ **Ignoring security scanning**
```bash
# Building and deploying without security checks
docker build -t myapp .
docker push myapp
```
✅ **Include security scanning in pipeline**
```bash
docker build -t myapp .
trivy image myapp
docker push myapp
```

---

## Practice Projects

### Project 1: Enterprise Security Implementation
Implement comprehensive security:
- Multi-layered security architecture
- Secrets management with Vault
- Network segmentation and policies
- Security scanning and compliance
- Incident response procedures
- Security monitoring and alerting

### Project 2: High-Performance Microservices
Optimize for performance:
- Load testing and optimization
- Resource tuning and limits
- Caching strategies
- Database optimization
- CDN integration
- Performance monitoring

### Project 3: Disaster Recovery System
Build resilient infrastructure:
- Multi-region deployment
- Automated backup and restore
- Failover mechanisms
- Data replication strategies
- Recovery time optimization
- Business continuity planning

---

## Related Technologies
- **Kubernetes** - Container orchestration
- **Service Mesh** - Istio, Linkerd
- **Infrastructure as Code** - Terraform, Ansible
- **CI/CD Platforms** - Jenkins, GitLab CI, GitHub Actions
- **Monitoring** - Prometheus, Grafana, ELK Stack
- **Security** - Vault, Falco, Twistlock

---

## Q&A Section

**Q: How do I optimize Docker images for production?**
A: Use multi-stage builds, minimal base images (alpine/distroless), optimize layer caching, remove unnecessary files, and implement proper security practices.

**Q: What's the best way to handle secrets in production?**
A: Use external secret management systems (Vault, AWS Secrets Manager), Docker secrets in swarm mode, or Kubernetes secrets. Never embed secrets in images.

**Q: How do I troubleshoot container networking issues?**
A: Check DNS resolution, test connectivity with ping/telnet, verify network configurations, examine iptables rules, and use network debugging tools.

**Q: What monitoring metrics are most important for containers?**
A: CPU/memory usage, network I/O, disk I/O, container health status, application-specific metrics, and error rates.

**Q: How do I implement proper logging in containerized applications?**
A: Use structured logging, centralized log aggregation (ELK/EFK stack), proper log levels, log rotation, and correlation IDs for tracing.

**Q: What's the best approach for container security scanning?**
A: Integrate scanning into CI/CD pipelines, use multiple scanning tools (Trivy, Clair, Snyk), scan both images and running containers, and maintain vulnerability databases.

**Q: How do I handle database migrations in containerized environments?**
A: Use init containers, migration services, or application startup scripts with proper dependency management and rollback capabilities.

**Q: What's the recommended approach for container backup and recovery?**
A: Implement automated volume backups, database dumps, configuration backups, test recovery procedures regularly, and maintain offsite backups.

---

*Congratulations! You've completed the Docker learning path. You now have the skills to design, implement, and maintain enterprise-grade containerized applications with proper security, performance optimization, and troubleshooting capabilities.*