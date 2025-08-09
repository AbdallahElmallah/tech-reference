# Level 2: Docker Images and Deployment

## Prerequisites
- Level 1 completion (basic Docker operations, container management)
- Understanding of application architecture and dependencies
- Basic knowledge of Linux file systems and permissions
- Familiarity with build processes and package managers

## Problem Statement
As a backend engineer, you need to:
- **Create custom Docker images** for your applications
- **Optimize image size and build time** for efficient deployment
- **Implement multi-stage builds** for production-ready images
- **Manage image registries** and distribution
- **Deploy applications** using various strategies
- **Version and tag images** properly for release management

These skills enable you to package and deploy applications consistently across different environments.

---

## Key Concepts

### 1. Understanding Docker Images and Layers

#### Image Architecture
```bash
# Docker images are built in layers
# Each instruction in Dockerfile creates a new layer
# Layers are cached and reused for efficiency

# View image layers
docker history nginx:alpine

# Inspect image details
docker inspect nginx:alpine

# Analyze image size and layers
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

#### Layer Caching
```dockerfile
# Bad: Changes to code invalidate all subsequent layers
FROM node:16-alpine
COPY . /app
WORKDIR /app
RUN npm install
CMD ["npm", "start"]

# Good: Dependencies cached separately from code
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

### 2. Creating Dockerfiles

#### Basic Dockerfile Structure
```dockerfile
# Use official base image
FROM node:16-alpine

# Set metadata
LABEL maintainer="your-email@company.com"
LABEL version="1.0"
LABEL description="Node.js application"

# Set working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start application
CMD ["npm", "start"]
```

#### Java Spring Boot Dockerfile
```dockerfile
FROM openjdk:17-jdk-alpine AS build

# Install Maven
RUN apk add --no-cache maven

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests

# Production stage
FROM openjdk:17-jre-alpine

WORKDIR /app

# Create user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# JVM optimization
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Start application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

#### Python Flask Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

# Start application
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

### 3. Multi-Stage Builds

#### Advanced Multi-Stage Example
```dockerfile
# Build stage
FROM maven:3.8.6-openjdk-17 AS builder

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn clean package -DskipTests

# Test stage
FROM builder AS tester
RUN mvn test

# Security scan stage
FROM builder AS security
RUN mvn org.owasp:dependency-check-maven:check

# Production stage
FROM openjdk:17-jre-alpine AS production

WORKDIR /app

# Install security updates
RUN apk update && apk upgrade && apk add --no-cache curl

# Create user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Set ownership
RUN chown appuser:appgroup app.jar
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

CMD ["java", "-jar", "app.jar"]
```

#### Node.js Multi-Stage with Testing
```dockerfile
# Base stage with common dependencies
FROM node:16-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Development stage
FROM base AS development
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]

# Test stage
FROM development AS test
RUN npm run test
RUN npm run lint
RUN npm audit --audit-level moderate

# Build stage
FROM development AS build
RUN npm run build

# Production stage
FROM node:16-alpine AS production
WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy production dependencies
COPY --from=base /app/node_modules ./node_modules

# Copy built application
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./

# Set ownership
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
```

### 4. Image Optimization Techniques

#### Size Optimization
```dockerfile
# Use alpine images
FROM node:16-alpine  # ~40MB vs node:16 ~350MB

# Multi-stage builds to exclude build tools
FROM node:16-alpine AS builder
# ... build steps ...

FROM node:16-alpine AS production
COPY --from=builder /app/dist ./dist

# Combine RUN commands to reduce layers
RUN apk update && \
    apk add --no-cache curl && \
    rm -rf /var/cache/apk/*

# Use .dockerignore
# .dockerignore file:
node_modules
.git
.gitignore
README.md
.env
.nyc_output
coverage
.coverage
*.log
```

#### .dockerignore Example
```dockerignore
# Version control
.git
.gitignore
.gitattributes

# Dependencies
node_modules
__pycache__
*.pyc
target/
.m2/

# IDE
.vscode
.idea
*.swp
*.swo

# Logs
*.log
logs/

# Testing
coverage/
.nyc_output
.pytest_cache

# Environment
.env
.env.local
.env.*.local

# Documentation
README.md
DOCS.md
*.md

# Build artifacts
dist/
build/
```

### 5. Building and Tagging Images

#### Basic Build Commands
```bash
# Build image with tag
docker build -t my-app:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t my-app:prod .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t my-app:prod .

# Build specific stage
docker build --target production -t my-app:prod .

# Build with no cache
docker build --no-cache -t my-app:latest .

# Build with progress output
docker build --progress=plain -t my-app:latest .
```

#### Advanced Build Techniques
```bash
# Build with BuildKit (faster, more features)
export DOCKER_BUILDKIT=1
docker build -t my-app:latest .

# Build with secrets (BuildKit)
docker build --secret id=mypassword,src=./password.txt -t my-app .

# Build with SSH agent forwarding
docker build --ssh default -t my-app .

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t my-app:latest .
```

#### Tagging Strategies
```bash
# Semantic versioning
docker tag my-app:latest my-app:1.0.0
docker tag my-app:latest my-app:1.0
docker tag my-app:latest my-app:1

# Environment-based tagging
docker tag my-app:latest my-app:dev
docker tag my-app:latest my-app:staging
docker tag my-app:latest my-app:prod

# Git-based tagging
docker tag my-app:latest my-app:$(git rev-parse --short HEAD)
docker tag my-app:latest my-app:$(git describe --tags)

# Date-based tagging
docker tag my-app:latest my-app:$(date +%Y%m%d)
```

### 6. Registry Management

#### Docker Hub Operations
```bash
# Login to Docker Hub
docker login

# Tag for Docker Hub
docker tag my-app:latest username/my-app:latest

# Push to Docker Hub
docker push username/my-app:latest

# Pull from Docker Hub
docker pull username/my-app:latest

# Logout
docker logout
```

#### Private Registry Operations
```bash
# Login to private registry
docker login registry.company.com

# Tag for private registry
docker tag my-app:latest registry.company.com/my-app:latest

# Push to private registry
docker push registry.company.com/my-app:latest

# Pull from private registry
docker pull registry.company.com/my-app:latest
```

#### AWS ECR Example
```bash
# Get login token
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com

# Tag for ECR
docker tag my-app:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest

# Push to ECR
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
```

---

## Hands-on Examples

### Example 1: Spring Boot Application

```dockerfile
# Dockerfile for Spring Boot app
FROM openjdk:17-jdk-alpine AS build

WORKDIR /workspace/app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
RUN ./mvnw dependency:go-offline -B

COPY src src
RUN ./mvnw install -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

FROM openjdk:17-jre-alpine
VOLUME /tmp
ARG DEPENDENCY=/workspace/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app

# Create user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

EXPOSE 8080
ENTRYPOINT ["java","-cp","app:app/lib/*","com.example.Application"]
```

```bash
# Build and run
docker build -t spring-app:latest .
docker run -d -p 8080:8080 --name my-spring-app spring-app:latest

# Test the application
curl http://localhost:8080/actuator/health

# View logs
docker logs -f my-spring-app

# Check image size
docker images spring-app
```

### Example 2: Node.js API with Optimization

```dockerfile
# Multi-stage Node.js build
FROM node:16-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:16-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
RUN npm run test

FROM node:16-alpine AS production
WORKDIR /app

# Install dumb-init
RUN apk add --no-cache dumb-init

# Create user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy dependencies and built app
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./

# Set ownership
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server.js"]
```

```bash
# Build with specific target
docker build --target production -t node-api:prod .

# Run with environment variables
docker run -d \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://user:pass@db:5432/mydb \
  --name api-server \
  node-api:prod

# Monitor health
docker ps
watch -n 5 'docker inspect api-server | grep Health -A 5'
```

### Example 3: Python Flask Application

```dockerfile
# Python Flask with security hardening
FROM python:3.11-slim AS base

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS development
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt
COPY . .
CMD ["flask", "run", "--host=0.0.0.0", "--debug"]

FROM base AS production

# Create non-root user
RUN useradd --create-home --shell /bin/bash app

# Copy application
COPY --chown=app:app . .

# Switch to non-root user
USER app

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

```bash
# Build for different environments
docker build --target development -t flask-app:dev .
docker build --target production -t flask-app:prod .

# Run development version
docker run -d -p 5000:5000 -v $(pwd):/app flask-app:dev

# Run production version
docker run -d -p 5000:5000 flask-app:prod
```

---

## Deployment Strategies

### Blue-Green Deployment
```bash
# Current production (blue)
docker run -d -p 8080:8080 --name app-blue my-app:v1.0

# Deploy new version (green)
docker run -d -p 8081:8080 --name app-green my-app:v2.0

# Test green deployment
curl http://localhost:8081/health

# Switch traffic (update load balancer or reverse proxy)
# Stop blue deployment
docker stop app-blue
docker rm app-blue

# Rename green to blue
docker stop app-green
docker run -d -p 8080:8080 --name app-blue my-app:v2.0
docker rm app-green
```

### Rolling Deployment
```bash
# Script for rolling deployment
#!/bin/bash
IMAGE=$1
INSTANCES=3

for i in $(seq 1 $INSTANCES); do
    echo "Updating instance $i"
    
    # Stop old instance
    docker stop app-$i
    docker rm app-$i
    
    # Start new instance
    docker run -d --name app-$i -p $((8080+$i)):8080 $IMAGE
    
    # Wait for health check
    sleep 30
    
    # Verify health
    if ! curl -f http://localhost:$((8080+$i))/health; then
        echo "Health check failed for instance $i"
        exit 1
    fi
    
    echo "Instance $i updated successfully"
done
```

### Canary Deployment
```bash
# Deploy canary version (10% traffic)
docker run -d -p 8082:8080 --name app-canary my-app:v2.0

# Configure load balancer to send 10% traffic to canary
# Monitor metrics and logs

# If successful, gradually increase traffic
# If issues, rollback immediately
docker stop app-canary
docker rm app-canary
```

---

## Image Security and Best Practices

### Security Scanning
```bash
# Scan image for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image my-app:latest

# Scan with specific severity
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --severity HIGH,CRITICAL my-app:latest

# Generate report
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/output \
  aquasec/trivy image --format json --output /output/report.json my-app:latest
```

### Dockerfile Security Best Practices
```dockerfile
# Use specific versions
FROM node:16.17.0-alpine3.16

# Don't run as root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# Use COPY instead of ADD
COPY package.json .

# Don't expose unnecessary ports
EXPOSE 3000

# Use secrets for sensitive data
# --mount=type=secret,id=mypassword,target=/run/secrets/password

# Minimize attack surface
RUN apk del build-dependencies

# Use read-only filesystem when possible
# docker run --read-only my-app
```

---

## Best Practices

### Dockerfile Best Practices
1. **Use official base images** from trusted sources
2. **Specify exact versions** instead of latest
3. **Use multi-stage builds** to reduce image size
4. **Run as non-root user** for security
5. **Use .dockerignore** to exclude unnecessary files
6. **Combine RUN commands** to reduce layers
7. **Order instructions** by frequency of change
8. **Use COPY instead of ADD** unless you need ADD's features

### Image Management
1. **Tag images semantically** (version numbers)
2. **Use consistent naming conventions**
3. **Implement image scanning** in CI/CD pipeline
4. **Clean up old images** regularly
5. **Use image registries** for distribution
6. **Document image contents** and usage

### Deployment
1. **Test images thoroughly** before production
2. **Use health checks** for reliability
3. **Implement proper logging** strategies
4. **Monitor resource usage**
5. **Plan rollback strategies**
6. **Use secrets management** for sensitive data

---

## Common Mistakes

❌ **Using latest tag in production**
```dockerfile
FROM node:latest  # Unpredictable
```
✅ **Use specific versions**
```dockerfile
FROM node:16.17.0-alpine3.16  # Predictable
```

❌ **Running as root**
```dockerfile
# No USER instruction - runs as root
CMD ["node", "app.js"]
```
✅ **Create and use non-root user**
```dockerfile
RUN adduser -D appuser
USER appuser
CMD ["node", "app.js"]
```

❌ **Inefficient layer caching**
```dockerfile
COPY . .
RUN npm install  # Invalidated on any code change
```
✅ **Optimize for caching**
```dockerfile
COPY package*.json ./
RUN npm install  # Cached unless dependencies change
COPY . .
```

❌ **Large image sizes**
```dockerfile
FROM ubuntu:latest  # Large base image
RUN apt-get update && apt-get install -y nodejs npm
```
✅ **Use minimal base images**
```dockerfile
FROM node:16-alpine  # Much smaller
```

---

## Practice Projects

### Project 1: Microservices Image Pipeline
Create optimized images for:
- User service (Node.js)
- Product service (Java Spring Boot)
- Order service (Python Flask)
- Database migration tool
Implement multi-stage builds and security scanning

### Project 2: CI/CD Image Pipeline
Build a complete pipeline:
- Automated image building
- Security scanning integration
- Multi-environment deployment
- Image promotion workflow
- Rollback capabilities

### Project 3: Image Optimization Challenge
Optimize existing images:
- Reduce image sizes by 50%+
- Improve build times
- Implement proper caching
- Add security hardening
- Document optimization techniques

---

## Related Levels
- **Previous:** Level 1 - Docker Fundamentals
- **Next:** Level 3 - Docker Compose and Scripting
- **Related:** Container orchestration, CI/CD pipelines, Security practices

---

## Q&A Section

**Q: What's the difference between COPY and ADD in Dockerfile?**
A: COPY simply copies files/directories. ADD has additional features like extracting tar files and downloading URLs, but COPY is preferred for simple file copying due to transparency.

**Q: How do I reduce Docker image size?**
A: Use alpine base images, multi-stage builds, combine RUN commands, use .dockerignore, and remove unnecessary packages after installation.

**Q: When should I use multi-stage builds?**
A: Use multi-stage builds when you need build tools that shouldn't be in the final image, when you want to run tests during build, or when you need different configurations for different environments.

**Q: How do I handle secrets in Docker images?**
A: Never put secrets in images. Use Docker secrets, environment variables at runtime, or secret management systems like HashiCorp Vault.

**Q: What's the best way to version Docker images?**
A: Use semantic versioning (1.0.0), include git commit hashes, and maintain both specific versions and environment tags (dev, staging, prod).

**Q: How do I debug a failed Docker build?**
A: Use `docker build --progress=plain` for detailed output, build up to a specific stage, or run intermediate containers interactively to debug issues.

**Q: Should I use one process per container?**
A: Yes, generally follow the "one process per container" principle for better scalability, debugging, and resource management.

**Q: How do I handle file permissions in containers?**
A: Set proper ownership with `chown`, use consistent user IDs between host and container, and avoid running as root when possible.

---

*Ready to move to Level 3? You should be comfortable with creating optimized Dockerfiles, managing images and registries, and implementing basic deployment strategies before advancing to Docker Compose and orchestration.*