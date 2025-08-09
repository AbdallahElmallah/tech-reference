# Level 3: Docker Compose and Scripting

## Prerequisites
- Level 2 completion (Docker images, Dockerfiles, deployment strategies)
- Understanding of multi-tier application architecture
- Basic knowledge of YAML syntax
- Familiarity with networking concepts and environment variables
- Experience with shell scripting (bash/PowerShell)

## Problem Statement
As a backend engineer, you need to:
- **Orchestrate multi-container applications** with Docker Compose
- **Write maintainable compose files** for different environments
- **Manage service dependencies** and startup order
- **Configure networking and volumes** for complex applications
- **Automate deployment workflows** with scripts
- **Integrate with CI/CD pipelines** for automated deployments
- **Handle environment-specific configurations** efficiently

These skills enable you to manage complex, multi-service applications and automate deployment processes.

---

## Key Concepts

### 1. Docker Compose Fundamentals

#### Basic Compose File Structure
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
    depends_on:
      - database
      - redis
    networks:
      - app-network
    volumes:
      - ./logs:/app/logs

  database:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

#### Essential Compose Commands
```bash
# Start all services
docker-compose up

# Start in detached mode
docker-compose up -d

# Build and start
docker-compose up --build

# Start specific services
docker-compose up web database

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View running services
docker-compose ps

# View logs
docker-compose logs
docker-compose logs -f web

# Execute commands in services
docker-compose exec web bash
docker-compose exec database psql -U user -d myapp

# Scale services
docker-compose up --scale web=3

# Restart services
docker-compose restart
docker-compose restart web
```

### 2. Advanced Compose Configurations

#### Multi-Environment Setup
```yaml
# docker-compose.yml (base)
version: '3.8'

services:
  web:
    build: .
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - database
    networks:
      - app-network

  database:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-myapp}
      POSTGRES_USER: ${POSTGRES_USER:-user}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    networks:
      - app-network

networks:
  app-network:
```

```yaml
# docker-compose.override.yml (development)
version: '3.8'

services:
  web:
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev
    environment:
      - DEBUG=*

  database:
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

volumes:
  postgres_dev_data:
```

```yaml
# docker-compose.prod.yml (production)
version: '3.8'

services:
  web:
    restart: unless-stopped
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  database:
    restart: unless-stopped
    volumes:
      - postgres_prod_data:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - web
    restart: unless-stopped

volumes:
  postgres_prod_data:
```

#### Complex Application Stack
```yaml
# Full-stack application with monitoring
version: '3.8'

services:
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    ports:
      - "80:80"
    depends_on:
      - api
    networks:
      - frontend-network
      - backend-network

  # API Gateway
  api-gateway:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/api-gateway.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - frontend-network
      - backend-network

  # Microservices
  user-service:
    build: ./services/user-service
    environment:
      - DATABASE_URL=postgresql://user:pass@user-db:5432/users
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      - user-db
      - redis
    networks:
      - backend-network
      - database-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  product-service:
    build: ./services/product-service
    environment:
      - DATABASE_URL=postgresql://user:pass@product-db:5432/products
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    depends_on:
      - product-db
      - elasticsearch
    networks:
      - backend-network
      - database-network
    deploy:
      replicas: 2

  order-service:
    build: ./services/order-service
    environment:
      - DATABASE_URL=postgresql://user:pass@order-db:5432/orders
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - order-db
      - rabbitmq
    networks:
      - backend-network
      - database-network

  # Databases
  user-db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - user_db_data:/var/lib/postgresql/data
    networks:
      - database-network

  product-db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: products
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - product_db_data:/var/lib/postgresql/data
    networks:
      - database-network

  order-db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - order_db_data:/var/lib/postgresql/data
    networks:
      - database-network

  # Cache and Message Queue
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend-network

  rabbitmq:
    image: rabbitmq:3-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    ports:
      - "15672:15672"  # Management UI
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - backend-network

  # Search
  elasticsearch:
    image: elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    networks:
      - backend-network

  # Monitoring
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - monitoring-network
      - backend-network

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
    depends_on:
      - prometheus
    networks:
      - monitoring-network

  # Logging
  elasticsearch-logs:
    image: elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - xpack.security.enabled=false
    volumes:
      - elasticsearch_logs_data:/usr/share/elasticsearch/data
    networks:
      - logging-network

  logstash:
    image: logstash:8.5.0
    volumes:
      - ./logging/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch-logs
    networks:
      - logging-network
      - backend-network

  kibana:
    image: kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch-logs:9200
    depends_on:
      - elasticsearch-logs
    networks:
      - logging-network

volumes:
  user_db_data:
  product_db_data:
  order_db_data:
  redis_data:
  rabbitmq_data:
  elasticsearch_data:
  prometheus_data:
  grafana_data:
  elasticsearch_logs_data:

networks:
  frontend-network:
  backend-network:
  database-network:
  monitoring-network:
  logging-network:
```

### 3. Environment Management

#### Environment Files
```bash
# .env (development)
NODE_ENV=development
DATABASE_URL=postgresql://user:devpass@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev-secret-key
DB_PASSWORD=devpass
RABBITMQ_PASSWORD=devpass
GRAFANA_PASSWORD=admin
```

```bash
# .env.staging
NODE_ENV=staging
DATABASE_URL=postgresql://user:stagingpass@db:5432/myapp_staging
REDIS_URL=redis://redis:6379
JWT_SECRET=staging-secret-key
DB_PASSWORD=stagingpass
RABBITMQ_PASSWORD=stagingpass
GRAFANA_PASSWORD=stagingadmin
```

```bash
# .env.production
NODE_ENV=production
DATABASE_URL=postgresql://user:${PROD_DB_PASSWORD}@db:5432/myapp_prod
REDIS_URL=redis://redis:6379
JWT_SECRET=${PROD_JWT_SECRET}
DB_PASSWORD=${PROD_DB_PASSWORD}
RABBITMQ_PASSWORD=${PROD_RABBITMQ_PASSWORD}
GRAFANA_PASSWORD=${PROD_GRAFANA_PASSWORD}
```

#### Environment-Specific Commands
```bash
# Development
docker-compose --env-file .env up

# Staging
docker-compose --env-file .env.staging -f docker-compose.yml -f docker-compose.staging.yml up

# Production
docker-compose --env-file .env.production -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 4. Service Dependencies and Health Checks

#### Advanced Dependency Management
```yaml
version: '3.8'

services:
  web:
    build: .
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  database:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

volumes:
  postgres_data:
```

#### Wait Scripts for Dependencies
```bash
#!/bin/bash
# wait-for-it.sh - Wait for service to be available

HOST=$1
PORT=$2
TIMEOUT=${3:-30}

echo "Waiting for $HOST:$PORT..."

for i in $(seq $TIMEOUT); do
    if nc -z $HOST $PORT; then
        echo "$HOST:$PORT is available"
        exit 0
    fi
    echo "Waiting... ($i/$TIMEOUT)"
    sleep 1
done

echo "Timeout waiting for $HOST:$PORT"
exit 1
```

```dockerfile
# Use wait script in Dockerfile
FROM node:16-alpine

WORKDIR /app

# Install netcat for wait script
RUN apk add --no-cache netcat-openbsd

COPY wait-for-it.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/wait-for-it.sh

COPY package*.json ./
RUN npm ci --only=production

COPY . .

CMD ["sh", "-c", "wait-for-it.sh database 5432 -- npm start"]
```

### 5. Networking and Volumes

#### Advanced Networking
```yaml
version: '3.8'

services:
  web:
    build: .
    networks:
      - frontend
      - backend
    ports:
      - "8080:8080"

  api:
    build: ./api
    networks:
      backend:
        aliases:
          - api-server
      database:
        aliases:
          - api-client

  database:
    image: postgres:14-alpine
    networks:
      - database
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  backend:
    driver: bridge
    internal: true
  database:
    driver: bridge
    internal: true
```

#### Volume Management
```yaml
version: '3.8'

services:
  web:
    build: .
    volumes:
      # Bind mount for development
      - ./src:/app/src:ro
      # Named volume for logs
      - app_logs:/app/logs
      # Anonymous volume for cache
      - /app/cache
      # Shared volume with other services
      - shared_data:/shared

  worker:
    build: ./worker
    volumes:
      - shared_data:/shared
      - worker_data:/data

  database:
    image: postgres:14-alpine
    volumes:
      # Named volume for data persistence
      - postgres_data:/var/lib/postgresql/data
      # Bind mount for initialization scripts
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
      # Bind mount for configuration
      - ./postgres.conf:/etc/postgresql/postgresql.conf:ro

volumes:
  app_logs:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /var/log/myapp
  shared_data:
  worker_data:
  postgres_data:
    external: true  # Use existing volume
```

---

## Automation and Scripting

### 1. Deployment Scripts

#### Bash Deployment Script
```bash
#!/bin/bash
# deploy.sh - Automated deployment script

set -e  # Exit on any error

# Configuration
ENVIRONMENT=${1:-development}
COMPOSE_FILE="docker-compose.yml"
ENV_FILE=".env"
BACKUP_DIR="./backups"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

warn() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING: $1${NC}"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1${NC}"
    exit 1
}

# Validate environment
validate_environment() {
    log "Validating environment: $ENVIRONMENT"
    
    case $ENVIRONMENT in
        development|staging|production)
            ENV_FILE=".env.$ENVIRONMENT"
            if [ "$ENVIRONMENT" != "development" ]; then
                COMPOSE_FILE="$COMPOSE_FILE -f docker-compose.$ENVIRONMENT.yml"
            fi
            ;;
        *)
            error "Invalid environment: $ENVIRONMENT. Use development, staging, or production."
            ;;
    esac
    
    if [ ! -f "$ENV_FILE" ]; then
        error "Environment file not found: $ENV_FILE"
    fi
    
    log "Using environment file: $ENV_FILE"
    log "Using compose file(s): $COMPOSE_FILE"
}

# Backup database
backup_database() {
    if [ "$ENVIRONMENT" = "production" ]; then
        log "Creating database backup..."
        
        mkdir -p "$BACKUP_DIR"
        BACKUP_FILE="$BACKUP_DIR/backup_$(date +%Y%m%d_%H%M%S).sql"
        
        docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE exec -T database \
            pg_dump -U user myapp > "$BACKUP_FILE"
        
        if [ $? -eq 0 ]; then
            log "Database backup created: $BACKUP_FILE"
        else
            error "Database backup failed"
        fi
    fi
}

# Health check
health_check() {
    log "Performing health check..."
    
    local max_attempts=30
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f http://localhost:8080/health >/dev/null 2>&1; then
            log "Health check passed"
            return 0
        fi
        
        warn "Health check attempt $attempt/$max_attempts failed"
        sleep 10
        ((attempt++))
    done
    
    error "Health check failed after $max_attempts attempts"
}

# Rollback function
rollback() {
    warn "Rolling back deployment..."
    
    # Stop current deployment
    docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE down
    
    # Restore from backup if production
    if [ "$ENVIRONMENT" = "production" ] && [ -n "$BACKUP_FILE" ]; then
        log "Restoring database from backup..."
        docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE up -d database
        sleep 10
        docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE exec -T database \
            psql -U user -d myapp < "$BACKUP_FILE"
    fi
    
    error "Deployment rolled back"
}

# Main deployment function
deploy() {
    log "Starting deployment to $ENVIRONMENT environment"
    
    # Validate environment
    validate_environment
    
    # Backup database for production
    backup_database
    
    # Pull latest images
    log "Pulling latest images..."
    docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE pull
    
    # Build custom images
    log "Building custom images..."
    docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE build
    
    # Stop existing services
    log "Stopping existing services..."
    docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE down
    
    # Start services
    log "Starting services..."
    docker-compose --env-file "$ENV_FILE" -f $COMPOSE_FILE up -d
    
    # Wait for services to be ready
    log "Waiting for services to be ready..."
    sleep 30
    
    # Perform health check
    if ! health_check; then
        rollback
    fi
    
    # Clean up old images
    log "Cleaning up old images..."
    docker image prune -f
    
    log "Deployment completed successfully!"
}

# Trap errors and rollback
trap rollback ERR

# Run deployment
deploy
```

#### PowerShell Deployment Script
```powershell
# deploy.ps1 - PowerShell deployment script

param(
    [Parameter(Mandatory=$false)]
    [ValidateSet("development", "staging", "production")]
    [string]$Environment = "development"
)

$ErrorActionPreference = "Stop"

# Configuration
$ComposeFile = "docker-compose.yml"
$EnvFile = ".env"
$BackupDir = "./backups"

function Write-Log {
    param([string]$Message)
    Write-Host "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] $Message" -ForegroundColor Green
}

function Write-Warning {
    param([string]$Message)
    Write-Host "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] WARNING: $Message" -ForegroundColor Yellow
}

function Write-Error {
    param([string]$Message)
    Write-Host "[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] ERROR: $Message" -ForegroundColor Red
    exit 1
}

function Validate-Environment {
    Write-Log "Validating environment: $Environment"
    
    switch ($Environment) {
        "development" {
            $script:EnvFile = ".env"
        }
        "staging" {
            $script:EnvFile = ".env.staging"
            $script:ComposeFile = "$ComposeFile -f docker-compose.staging.yml"
        }
        "production" {
            $script:EnvFile = ".env.production"
            $script:ComposeFile = "$ComposeFile -f docker-compose.prod.yml"
        }
    }
    
    if (!(Test-Path $EnvFile)) {
        Write-Error "Environment file not found: $EnvFile"
    }
    
    Write-Log "Using environment file: $EnvFile"
    Write-Log "Using compose file(s): $ComposeFile"
}

function Backup-Database {
    if ($Environment -eq "production") {
        Write-Log "Creating database backup..."
        
        if (!(Test-Path $BackupDir)) {
            New-Item -ItemType Directory -Path $BackupDir
        }
        
        $BackupFile = "$BackupDir/backup_$(Get-Date -Format 'yyyyMMdd_HHmmss').sql"
        
        $cmd = "docker-compose --env-file `"$EnvFile`" -f $ComposeFile exec -T database pg_dump -U user myapp"
        Invoke-Expression $cmd | Out-File -FilePath $BackupFile
        
        if ($LASTEXITCODE -eq 0) {
            Write-Log "Database backup created: $BackupFile"
        } else {
            Write-Error "Database backup failed"
        }
    }
}

function Test-Health {
    Write-Log "Performing health check..."
    
    $maxAttempts = 30
    $attempt = 1
    
    while ($attempt -le $maxAttempts) {
        try {
            $response = Invoke-WebRequest -Uri "http://localhost:8080/health" -UseBasicParsing
            if ($response.StatusCode -eq 200) {
                Write-Log "Health check passed"
                return $true
            }
        } catch {
            Write-Warning "Health check attempt $attempt/$maxAttempts failed"
        }
        
        Start-Sleep -Seconds 10
        $attempt++
    }
    
    Write-Error "Health check failed after $maxAttempts attempts"
    return $false
}

function Start-Deployment {
    Write-Log "Starting deployment to $Environment environment"
    
    # Validate environment
    Validate-Environment
    
    # Backup database for production
    Backup-Database
    
    # Pull latest images
    Write-Log "Pulling latest images..."
    Invoke-Expression "docker-compose --env-file `"$EnvFile`" -f $ComposeFile pull"
    
    # Build custom images
    Write-Log "Building custom images..."
    Invoke-Expression "docker-compose --env-file `"$EnvFile`" -f $ComposeFile build"
    
    # Stop existing services
    Write-Log "Stopping existing services..."
    Invoke-Expression "docker-compose --env-file `"$EnvFile`" -f $ComposeFile down"
    
    # Start services
    Write-Log "Starting services..."
    Invoke-Expression "docker-compose --env-file `"$EnvFile`" -f $ComposeFile up -d"
    
    # Wait for services to be ready
    Write-Log "Waiting for services to be ready..."
    Start-Sleep -Seconds 30
    
    # Perform health check
    if (!(Test-Health)) {
        Write-Error "Health check failed, deployment aborted"
    }
    
    # Clean up old images
    Write-Log "Cleaning up old images..."
    docker image prune -f
    
    Write-Log "Deployment completed successfully!"
}

# Run deployment
Start-Deployment
```

### 2. CI/CD Integration Scripts

#### GitHub Actions Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Run tests
        run: |
          docker-compose -f docker-compose.test.yml up --build --abort-on-container-exit
          docker-compose -f docker-compose.test.yml down

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to staging
        run: |
          echo "${{ secrets.STAGING_ENV }}" > .env.staging
          ./scripts/deploy.sh staging
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to production
        run: |
          echo "${{ secrets.PRODUCTION_ENV }}" > .env.production
          ./scripts/deploy.sh production
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
```

#### Jenkins Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry.com'
        IMAGE_NAME = 'myapp'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Test') {
            steps {
                script {
                    sh 'docker-compose -f docker-compose.test.yml up --build --abort-on-container-exit'
                    sh 'docker-compose -f docker-compose.test.yml down'
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def imageTag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag} ."
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${imageTag}"
                    env.IMAGE_TAG = imageTag
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    sh './scripts/deploy.sh staging'
                }
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                script {
                    sh './scripts/deploy.sh production'
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker system prune -f'
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check console output at ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

### 3. Monitoring and Maintenance Scripts

#### System Monitoring Script
```bash
#!/bin/bash
# monitor.sh - System monitoring script

SERVICES=("web" "database" "redis" "api")
ALERT_EMAIL="admin@company.com"
LOG_FILE="/var/log/docker-monitor.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

check_service_health() {
    local service=$1
    local health_status
    
    health_status=$(docker-compose ps -q "$service" | xargs docker inspect --format='{{.State.Health.Status}}' 2>/dev/null)
    
    if [ "$health_status" = "healthy" ]; then
        log "✓ $service is healthy"
        return 0
    else
        log "✗ $service is unhealthy (status: $health_status)"
        return 1
    fi
}

check_resource_usage() {
    log "Checking resource usage..."
    
    # Check disk usage
    disk_usage=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$disk_usage" -gt 80 ]; then
        log "WARNING: Disk usage is ${disk_usage}%"
        send_alert "High disk usage: ${disk_usage}%"
    fi
    
    # Check memory usage
    memory_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    if [ "$memory_usage" -gt 80 ]; then
        log "WARNING: Memory usage is ${memory_usage}%"
        send_alert "High memory usage: ${memory_usage}%"
    fi
    
    # Check Docker daemon
    if ! docker info >/dev/null 2>&1; then
        log "ERROR: Docker daemon is not responding"
        send_alert "Docker daemon is not responding"
    fi
}

send_alert() {
    local message=$1
    echo "$message" | mail -s "Docker Alert: $(hostname)" "$ALERT_EMAIL"
    log "Alert sent: $message"
}

restart_unhealthy_services() {
    log "Checking for unhealthy services..."
    
    for service in "${SERVICES[@]}"; do
        if ! check_service_health "$service"; then
            log "Restarting unhealthy service: $service"
            docker-compose restart "$service"
            sleep 30
            
            if check_service_health "$service"; then
                log "Service $service restarted successfully"
            else
                log "Failed to restart service $service"
                send_alert "Failed to restart service $service"
            fi
        fi
    done
}

cleanup_resources() {
    log "Cleaning up Docker resources..."
    
    # Remove stopped containers
    docker container prune -f
    
    # Remove unused images
    docker image prune -f
    
    # Remove unused volumes (be careful!)
    # docker volume prune -f
    
    # Remove unused networks
    docker network prune -f
    
    log "Cleanup completed"
}

main() {
    log "Starting Docker monitoring..."
    
    check_resource_usage
    restart_unhealthy_services
    
    # Run cleanup weekly
    if [ "$(date +%u)" -eq 1 ] && [ "$(date +%H)" -eq 2 ]; then
        cleanup_resources
    fi
    
    log "Monitoring completed"
}

main
```

---

## Best Practices

### Compose File Organization
1. **Use version 3.8+** for latest features
2. **Organize services logically** (frontend, backend, database)
3. **Use meaningful service names**
4. **Implement proper health checks**
5. **Use secrets for sensitive data**
6. **Document complex configurations**

### Environment Management
1. **Separate environment files** for each stage
2. **Use environment variables** for configuration
3. **Never commit secrets** to version control
4. **Validate environment files** before deployment
5. **Use default values** where appropriate

### Scripting
1. **Make scripts idempotent** (safe to run multiple times)
2. **Include proper error handling**
3. **Add logging and monitoring**
4. **Test scripts thoroughly**
5. **Document script usage**

### Security
1. **Use non-root users** in containers
2. **Implement network segmentation**
3. **Scan images for vulnerabilities**
4. **Use secrets management**
5. **Monitor and audit access**

---

## Common Mistakes

❌ **Not using health checks**
```yaml
services:
  web:
    image: myapp
    # No health check - can't detect failures
```
✅ **Implement proper health checks**
```yaml
services:
  web:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

❌ **Hardcoding configuration**
```yaml
services:
  database:
    environment:
      POSTGRES_PASSWORD: hardcoded-password  # Bad!
```
✅ **Use environment variables**
```yaml
services:
  database:
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # Good!
```

❌ **Not handling dependencies**
```yaml
services:
  web:
    depends_on:
      - database  # Only waits for container start, not readiness
```
✅ **Use proper dependency management**
```yaml
services:
  web:
    depends_on:
      database:
        condition: service_healthy  # Waits for health check
```

---

## Practice Projects

### Project 1: E-commerce Microservices
Build a complete e-commerce platform:
- User service (authentication)
- Product catalog service
- Order processing service
- Payment service
- Notification service
- API gateway
- Databases (PostgreSQL, Redis)
- Message queue (RabbitMQ)
- Monitoring (Prometheus, Grafana)

### Project 2: CI/CD Pipeline
Implement automated deployment:
- Multi-environment setup (dev, staging, prod)
- Automated testing and building
- Security scanning
- Database migrations
- Blue-green deployment
- Rollback capabilities

### Project 3: Monitoring and Logging Stack
Set up comprehensive monitoring:
- Application metrics collection
- Log aggregation (ELK stack)
- Alerting system
- Performance monitoring
- Health check dashboard
- Automated incident response

---

## Related Levels
- **Previous:** Level 2 - Images and Deployment
- **Next:** Level 4 - Advanced Docker (Tuning, Security, Troubleshooting)
- **Related:** Kubernetes, Service mesh, Infrastructure as Code

---

## Q&A Section

**Q: What's the difference between `depends_on` and `links`?**
A: `depends_on` controls startup order and is the modern approach. `links` is deprecated and was used for container communication in older Docker versions.

**Q: How do I handle database migrations in Docker Compose?**
A: Use init containers, migration services, or include migration commands in your application startup script with proper dependency management.

**Q: Can I use Docker Compose in production?**
A: Docker Compose is suitable for single-host deployments. For multi-host production environments, consider Kubernetes or Docker Swarm.

**Q: How do I debug networking issues between containers?**
A: Use `docker-compose exec service_name ping other_service`, check network configurations, and verify service discovery is working.

**Q: What's the best way to handle secrets in Compose files?**
A: Use Docker secrets (in swarm mode), external secret management systems, or environment variables loaded from secure sources.

**Q: How do I scale services with Docker Compose?**
A: Use `docker-compose up --scale service_name=3` or define replicas in the compose file (swarm mode).

**Q: Should I use one compose file or multiple?**
A: Use multiple compose files for different environments (base + environment-specific overrides) to maintain flexibility and avoid duplication.

**Q: How do I handle persistent data in containers?**
A: Use named volumes for data that should persist, bind mounts for development, and ensure proper backup strategies for production data.

---

*Ready to move to Level 4? You should be comfortable with multi-container orchestration, environment management, and deployment automation before advancing to advanced Docker topics like performance tuning and security hardening.*