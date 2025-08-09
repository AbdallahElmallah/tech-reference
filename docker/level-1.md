# Level 1: Docker Fundamentals - Using, Logs, Start/Stop Service

## Prerequisites
- Basic command line knowledge (Windows PowerShell, Linux terminal)
- Understanding of applications and services
- Basic knowledge of operating systems
- Familiarity with software installation processes

## Problem Statement
As a backend engineer, you need to:
- **Understand containerization** and how it differs from traditional deployment
- **Manage Docker containers** effectively in development and testing
- **Debug applications** running in containers using logs
- **Control container lifecycle** (start, stop, restart, remove)
- **Work with existing images** from Docker Hub and registries
- **Ensure consistent environments** across development team

These skills form the foundation for modern application deployment and development workflows.

---

## Key Concepts

### 1. Understanding Docker and Containers

#### What is Docker?
```bash
# Docker is a containerization platform that packages applications
# and their dependencies into lightweight, portable containers

# Key benefits:
# - Consistency across environments
# - Isolation between applications
# - Resource efficiency
# - Easy deployment and scaling
```

#### Containers vs Virtual Machines
```bash
# Virtual Machine:
# Host OS -> Hypervisor -> Guest OS -> Application
# - Heavy resource usage
# - Slower startup
# - Full OS overhead

# Container:
# Host OS -> Docker Engine -> Container -> Application
# - Lightweight
# - Fast startup
# - Shared OS kernel
```

### 2. Docker Installation and Setup

#### Windows Installation
```powershell
# Download Docker Desktop from docker.com
# Install and restart system

# Verify installation
docker --version
docker info

# Test with hello-world
docker run hello-world
```

#### Linux Installation (Ubuntu/Debian)
```bash
# Update package index
sudo apt-get update

# Install required packages
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
# Logout and login again

# Verify installation
docker --version
docker run hello-world
```

### 3. Basic Docker Commands

#### Running Containers
```bash
# Basic run command
docker run nginx

# Run in detached mode (background)
docker run -d nginx

# Run with custom name
docker run -d --name my-nginx nginx

# Run with port mapping
docker run -d -p 8080:80 --name web-server nginx

# Run with environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql:8.0

# Run with volume mounting
docker run -d -v /host/path:/container/path nginx

# Run interactively with terminal
docker run -it ubuntu bash

# Run and remove after exit
docker run --rm -it ubuntu bash
```

#### Container Management
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start a stopped container
docker start container_name_or_id

# Stop a running container
docker stop container_name_or_id

# Restart a container
docker restart container_name_or_id

# Pause/unpause a container
docker pause container_name_or_id
docker unpause container_name_or_id

# Remove a container
docker rm container_name_or_id

# Remove a running container (force)
docker rm -f container_name_or_id

# Remove all stopped containers
docker container prune
```

### 4. Working with Docker Logs

#### Viewing Container Logs
```bash
# View logs of a container
docker logs container_name_or_id

# Follow logs in real-time
docker logs -f container_name_or_id

# Show last N lines
docker logs --tail 50 container_name_or_id

# Show logs since specific time
docker logs --since "2024-01-01T00:00:00" container_name_or_id

# Show logs until specific time
docker logs --until "2024-01-01T23:59:59" container_name_or_id

# Show timestamps
docker logs -t container_name_or_id

# Combine options
docker logs -f --tail 100 -t my-app
```

#### Log Management Best Practices
```bash
# Configure log driver (in docker run)
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# View log driver info
docker inspect container_name | grep LogConfig -A 10

# Clear container logs (careful!)
sudo truncate -s 0 /var/lib/docker/containers/container_id/container_id-json.log
```

### 5. Container Inspection and Debugging

#### Inspecting Containers
```bash
# Get detailed container information
docker inspect container_name_or_id

# Get specific information using format
docker inspect --format='{{.State.Status}}' container_name
docker inspect --format='{{.NetworkSettings.IPAddress}}' container_name

# View container processes
docker top container_name_or_id

# View container resource usage
docker stats container_name_or_id

# View container filesystem changes
docker diff container_name_or_id
```

#### Executing Commands in Running Containers
```bash
# Execute command in running container
docker exec container_name_or_id ls -la

# Get interactive shell
docker exec -it container_name_or_id bash
docker exec -it container_name_or_id sh  # for alpine images

# Execute as specific user
docker exec -it --user root container_name_or_id bash

# Execute with environment variables
docker exec -it -e VAR=value container_name_or_id bash
```

### 6. Working with Docker Images

#### Image Management
```bash
# List local images
docker images
docker image ls

# Search for images on Docker Hub
docker search nginx

# Pull an image from registry
docker pull nginx:latest
docker pull nginx:1.21-alpine

# Remove an image
docker rmi image_name_or_id

# Remove unused images
docker image prune

# Remove all unused images
docker image prune -a

# View image history
docker history nginx:latest

# Get image information
docker inspect nginx:latest
```

---

## Hands-on Examples

### Example 1: Running a Web Server

```bash
# Step 1: Run Nginx web server
docker run -d -p 8080:80 --name my-webserver nginx:latest

# Step 2: Verify it's running
docker ps

# Step 3: Test the web server
# Open browser to http://localhost:8080
# Or use curl
curl http://localhost:8080

# Step 4: View logs
docker logs my-webserver

# Step 5: Follow logs in real-time
docker logs -f my-webserver

# Step 6: Execute commands inside container
docker exec -it my-webserver bash
# Inside container:
ls /usr/share/nginx/html/
cat /etc/nginx/nginx.conf
exit

# Step 7: Stop and remove
docker stop my-webserver
docker rm my-webserver
```

### Example 2: Database Container Management

```bash
# Step 1: Run MySQL database
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=secretpassword \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=testuser \
  -e MYSQL_PASSWORD=testpass \
  -p 3306:3306 \
  mysql:8.0

# Step 2: Check if it's running
docker ps

# Step 3: View startup logs
docker logs my-mysql

# Step 4: Wait for MySQL to be ready
docker logs -f my-mysql | grep "ready for connections"

# Step 5: Connect to MySQL
docker exec -it my-mysql mysql -u root -p
# Enter password: secretpassword
# Inside MySQL:
SHOW DATABASES;
USE testdb;
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50));
INSERT INTO users VALUES (1, 'John Doe');
SELECT * FROM users;
exit

# Step 6: Monitor resource usage
docker stats my-mysql

# Step 7: Backup database (example)
docker exec my-mysql mysqldump -u root -psecretpassword testdb > backup.sql

# Step 8: Stop and remove
docker stop my-mysql
docker rm my-mysql
```

### Example 3: Application Development Container

```bash
# Step 1: Run Node.js development environment
docker run -d \
  --name node-dev \
  -p 3000:3000 \
  -v "$(pwd):/app" \
  -w /app \
  node:16-alpine \
  sh -c "npm install && npm start"

# Step 2: Monitor application logs
docker logs -f node-dev

# Step 3: Debug inside container
docker exec -it node-dev sh
# Inside container:
ps aux
ls -la
npm list
exit

# Step 4: Restart application
docker restart node-dev

# Step 5: View detailed container info
docker inspect node-dev

# Step 6: Check network connectivity
docker exec node-dev ping google.com

# Step 7: Clean up
docker stop node-dev
docker rm node-dev
```

---

## Service Management Patterns

### Container Health Checks
```bash
# Run container with health check
docker run -d \
  --name healthy-app \
  --health-cmd="curl -f http://localhost:80 || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  nginx

# Check health status
docker ps
docker inspect healthy-app | grep Health -A 10
```

### Container Restart Policies
```bash
# Always restart
docker run -d --restart always --name persistent-app nginx

# Restart unless stopped
docker run -d --restart unless-stopped --name semi-persistent-app nginx

# Restart on failure
docker run -d --restart on-failure:3 --name fault-tolerant-app nginx

# No restart (default)
docker run -d --restart no --name temporary-app nginx
```

### Resource Limits
```bash
# Limit memory and CPU
docker run -d \
  --name limited-app \
  --memory="512m" \
  --cpus="1.5" \
  nginx

# Monitor resource usage
docker stats limited-app
```

---

## Troubleshooting Common Issues

### Container Won't Start
```bash
# Check container status
docker ps -a

# View container logs
docker logs container_name

# Inspect container configuration
docker inspect container_name

# Try running interactively
docker run -it image_name bash
```

### Port Conflicts
```bash
# Check what's using a port
netstat -tulpn | grep :8080  # Linux
netstat -ano | findstr :8080  # Windows

# Use different port
docker run -d -p 8081:80 nginx

# Check Docker port mappings
docker port container_name
```

### Permission Issues
```bash
# Run as specific user
docker run -it --user $(id -u):$(id -g) ubuntu bash

# Fix file permissions
docker exec -it --user root container_name chown -R user:group /path
```

### Network Connectivity
```bash
# Test network from container
docker exec container_name ping google.com
docker exec container_name nslookup google.com

# Check container IP
docker inspect container_name | grep IPAddress

# Test port connectivity
docker exec container_name telnet host port
```

---

## Best Practices

### Container Management
1. **Use meaningful names** for containers
2. **Always specify image tags** (avoid :latest in production)
3. **Use restart policies** for production containers
4. **Monitor resource usage** regularly
5. **Clean up unused containers** and images

### Logging
1. **Configure log rotation** to prevent disk space issues
2. **Use structured logging** in applications
3. **Centralize logs** for production environments
4. **Monitor log volume** and performance impact
5. **Use appropriate log levels** (debug, info, warn, error)

### Security
1. **Don't run containers as root** when possible
2. **Use official images** from trusted sources
3. **Keep images updated** regularly
4. **Limit container resources** to prevent abuse
5. **Use secrets management** for sensitive data

### Development Workflow
1. **Use volume mounts** for development
2. **Leverage Docker for consistent environments**
3. **Document container requirements**
4. **Use .dockerignore** files
5. **Test containers before deployment**

---

## Common Mistakes

❌ **Running containers without names**
```bash
# Don't do this
docker run -d nginx
```
✅ **Always use meaningful names**
```bash
# Do this instead
docker run -d --name frontend-server nginx
```

❌ **Ignoring container logs**
```bash
# Don't ignore when containers fail
docker run -d my-app  # fails silently
```
✅ **Always check logs when debugging**
```bash
# Check logs immediately
docker run -d --name my-app my-app
docker logs my-app
```

❌ **Using :latest in production**
```bash
# Avoid this in production
docker run -d nginx:latest
```
✅ **Use specific versions**
```bash
# Use specific, tested versions
docker run -d nginx:1.21.6-alpine
```

❌ **Not cleaning up resources**
```bash
# This leads to disk space issues
# Never cleaning up old containers and images
```
✅ **Regular cleanup**
```bash
# Clean up regularly
docker container prune
docker image prune
docker system prune
```

---

## Simple Practice Projects

### Project 1: Web Application Stack
Set up and manage:
- Nginx web server on port 8080
- MySQL database on port 3306
- Practice starting, stopping, and monitoring both
- Create a simple HTML page served by Nginx
- Connect to MySQL and create a test database

### Project 2: Development Environment
Create a development setup:
- Node.js application container
- Redis cache container
- Practice log monitoring and debugging
- Simulate application failures and recovery
- Test different restart policies

### Project 3: Container Lifecycle Management
Practice container operations:
- Create containers with different configurations
- Practice backup and restore procedures
- Test resource limits and monitoring
- Implement health checks
- Document troubleshooting procedures

---

## Related Levels
- **Next:** Level 2 - Images and Deployment (Custom Dockerfiles, image optimization)
- **Related:** Basic networking concepts, Linux command line, Application architecture

---

## Q&A Section

**Q: What's the difference between `docker run` and `docker start`?**
A: `docker run` creates and starts a new container from an image. `docker start` starts an existing stopped container.

**Q: How do I see what's running inside a container?**
A: Use `docker exec -it container_name bash` to get a shell, or `docker top container_name` to see processes.

**Q: Why can't I connect to my containerized application?**
A: Check port mapping with `-p host_port:container_port`, ensure the application binds to 0.0.0.0 (not localhost), and verify firewall settings.

**Q: How do I persist data when a container is removed?**
A: Use volumes (`-v host_path:container_path`) or named volumes to persist data outside the container.

**Q: What does 'detached mode' mean?**
A: Detached mode (`-d`) runs the container in the background, returning control to your terminal immediately.

**Q: How do I stop all running containers?**
A: Use `docker stop $(docker ps -q)` to stop all running containers.

**Q: Why do my logs disappear when I remove a container?**
A: Container logs are stored with the container. Use external logging solutions or volume mounts for persistent logs.

**Q: How do I know if my container is healthy?**
A: Use health checks (`--health-cmd`) and monitor with `docker ps` or `docker inspect` to see health status.

---

*Ready to move to Level 2? You should be comfortable with basic container operations, log management, and troubleshooting before advancing to image creation and deployment strategies.*