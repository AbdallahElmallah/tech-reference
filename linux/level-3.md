# Level 3: Advanced Administration - Scripting, Installation

## Prerequisites
- Completion of Level 1 (Linux basics, SSH, file management) and Level 2 (permissions, logs, Git)
- Comfortable with command-line operations and system administration
- Understanding of file permissions, process management, and version control
- Basic programming concepts and logic
- Access to a Linux system with sudo privileges

## Problem Statement
As a backend engineer, you need to:
- **Automate repetitive tasks** using shell scripting and automation tools
- **Manage software installation** and package dependencies efficiently
- **Configure system services** and manage system startup processes
- **Implement deployment automation** for applications and infrastructure
- **Create maintenance scripts** for system health and performance
- **Manage multiple environments** (development, staging, production)
- **Implement backup and recovery** automation
- **Monitor and alert** on system conditions automatically

These advanced skills enable you to build robust, maintainable infrastructure and streamline development workflows.

---

## Key Concepts

### 1. Shell Scripting Fundamentals

#### Basic Script Structure
```bash
#!/bin/bash
# Shebang line - specifies interpreter

# Script metadata
# Author: Your Name
# Date: 2024-01-15
# Purpose: Example script structure
# Version: 1.0

# Exit on any error
set -e

# Variables
SCRIPT_NAME=$(basename "$0")
SCRIPT_DIR=$(dirname "$0")
DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Functions
log_message() {
    echo "[$DATE] $1" | tee -a "/var/log/$SCRIPT_NAME.log"
}

error_exit() {
    log_message "ERROR: $1"
    exit 1
}

# Main script logic
log_message "Script started"

# Your code here

log_message "Script completed successfully"
exit 0
```

#### Variables and Parameters
```bash
#!/bin/bash

# Variable assignment (no spaces around =)
NAME="John Doe"
COUNT=10
FILE_PATH="/tmp/data.txt"

# Environment variables
export DATABASE_URL="postgresql://localhost:5432/mydb"
export LOG_LEVEL="INFO"

# Command substitution
CURRENT_DATE=$(date +"%Y-%m-%d")
FILE_COUNT=$(ls -1 | wc -l)
DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')

# Parameter variables
echo "Script name: $0"
echo "First parameter: $1"
echo "Second parameter: $2"
echo "All parameters: $@"
echo "Number of parameters: $#"
echo "Exit status of last command: $?"
echo "Process ID: $$"

# Default values
ENVIRONMENT=${1:-"development"}
PORT=${2:-8080}
DEBUG=${DEBUG:-false}

# Array variables
SERVERS=("web1" "web2" "db1" "cache1")
echo "First server: ${SERVERS[0]}"
echo "All servers: ${SERVERS[@]}"
echo "Number of servers: ${#SERVERS[@]}"

# String manipulation
FILENAME="document.pdf"
echo "Extension: ${FILENAME##*.}"     # pdf
echo "Name without extension: ${FILENAME%.*}"  # document
echo "Uppercase: ${NAME^^}"          # JOHN DOE
echo "Lowercase: ${NAME,,}"          # john doe
```

#### Control Structures
```bash
#!/bin/bash

# Conditional statements
if [ "$1" = "start" ]; then
    echo "Starting service..."
elif [ "$1" = "stop" ]; then
    echo "Stopping service..."
elif [ "$1" = "restart" ]; then
    echo "Restarting service..."
else
    echo "Usage: $0 {start|stop|restart}"
    exit 1
fi

# File and directory tests
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "Nginx config exists"
fi

if [ -d "/var/log/myapp" ]; then
    echo "Log directory exists"
else
    mkdir -p "/var/log/myapp"
fi

if [ -x "/usr/bin/docker" ]; then
    echo "Docker is installed and executable"
fi

# Numeric comparisons
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
    echo "High CPU usage: $CPU_USAGE%"
fi

# String comparisons
if [[ "$ENVIRONMENT" =~ ^(prod|production)$ ]]; then
    echo "Production environment detected"
fi

# Loops
# For loop with list
for server in web1 web2 db1; do
    echo "Checking server: $server"
    ping -c 1 "$server" > /dev/null && echo "$server is up" || echo "$server is down"
done

# For loop with range
for i in {1..5}; do
    echo "Iteration $i"
done

# For loop with files
for file in /var/log/*.log; do
    if [ -f "$file" ]; then
        echo "Processing log file: $file"
        # Process file here
    fi
done

# While loop
counter=1
while [ $counter -le 10 ]; do
    echo "Counter: $counter"
    ((counter++))
done

# While loop reading file
while IFS= read -r line; do
    echo "Processing: $line"
done < "/etc/hosts"

# Case statement
case "$1" in
    "start")
        echo "Starting application"
        ;;
    "stop")
        echo "Stopping application"
        ;;
    "status")
        echo "Checking application status"
        ;;
    *)
        echo "Unknown command: $1"
        exit 1
        ;;
esac
```

#### Functions and Error Handling
```bash
#!/bin/bash

# Function definition
backup_database() {
    local db_name="$1"
    local backup_dir="$2"
    local timestamp=$(date +"%Y%m%d_%H%M%S")
    
    # Validate parameters
    if [ -z "$db_name" ] || [ -z "$backup_dir" ]; then
        echo "Error: Missing required parameters"
        echo "Usage: backup_database <db_name> <backup_dir>"
        return 1
    fi
    
    # Create backup directory if it doesn't exist
    mkdir -p "$backup_dir"
    
    # Perform backup
    local backup_file="$backup_dir/${db_name}_${timestamp}.sql"
    if pg_dump "$db_name" > "$backup_file"; then
        echo "Database backup successful: $backup_file"
        return 0
    else
        echo "Database backup failed"
        return 1
    fi
}

# Error handling with trap
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/script_temp_*
    echo "Cleanup completed"
}

# Set trap for cleanup on exit
trap cleanup EXIT

# Set trap for error handling
error_handler() {
    echo "Error occurred in script at line $1"
    exit 1
}
trap 'error_handler $LINENO' ERR

# Logging function
log() {
    local level="$1"
    shift
    local message="$@"
    local timestamp=$(date +"%Y-%m-%d %H:%M:%S")
    echo "[$timestamp] [$level] $message" | tee -a "/var/log/script.log"
}

# Usage examples
log "INFO" "Script started"

if backup_database "myapp" "/backups"; then
    log "INFO" "Backup completed successfully"
else
    log "ERROR" "Backup failed"
    exit 1
fi

log "INFO" "Script completed"
```

### 2. Package Management

#### APT (Debian/Ubuntu)
```bash
# Update package lists
sudo apt update

# Upgrade installed packages
sudo apt upgrade
sudo apt full-upgrade          # More comprehensive upgrade

# Install packages
sudo apt install nginx
sudo apt install nginx mysql-server php-fpm
sudo apt install -y git curl wget  # Auto-confirm installation

# Search for packages
apt search nginx
apt show nginx                  # Show package details

# Remove packages
sudo apt remove nginx
sudo apt purge nginx            # Remove package and config files
sudo apt autoremove             # Remove unused dependencies

# Package information
dpkg -l                         # List installed packages
dpkg -l | grep nginx            # Find specific package
dpkg -L nginx                   # List files installed by package
dpkg -S /usr/sbin/nginx         # Find which package owns file

# Hold packages (prevent updates)
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
sudo apt-mark showhold

# Repository management
sudo add-apt-repository ppa:ondrej/php
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys KEY_ID

# Clean package cache
sudo apt clean
sudo apt autoclean
```

#### YUM/DNF (RedHat/CentOS/Fedora)
```bash
# Update system
sudo yum update                 # CentOS/RHEL
sudo dnf update                 # Fedora

# Install packages
sudo yum install nginx
sudo dnf install nginx mysql-server

# Search packages
yum search nginx
dnf search nginx

# Package information
yum info nginx
yum list installed
yum list installed | grep nginx

# Remove packages
sudo yum remove nginx
sudo dnf remove nginx

# Repository management
sudo yum-config-manager --add-repo https://repo.example.com/repo.repo
sudo dnf config-manager --add-repo https://repo.example.com/repo.repo

# Clean cache
sudo yum clean all
sudo dnf clean all

# History
yum history
dnf history
```

#### Snap Packages
```bash
# Install snap
sudo apt install snapd          # Ubuntu/Debian
sudo yum install snapd           # CentOS/RHEL

# Snap operations
sudo snap install code --classic
sudo snap install docker
sudo snap list
sudo snap refresh
sudo snap remove code

# Snap information
snap find editor
snap info code
```

#### Flatpak
```bash
# Install Flatpak
sudo apt install flatpak

# Add Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Install applications
flatpak install flathub org.gimp.GIMP
flatpak list
flatpak run org.gimp.GIMP
flatpak uninstall org.gimp.GIMP
```

### 3. System Services and Startup

#### Systemd Service Management
```bash
# Service operations
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx

# Enable/disable services
sudo systemctl enable nginx     # Start at boot
sudo systemctl disable nginx    # Don't start at boot
sudo systemctl is-enabled nginx

# List services
sudo systemctl list-units --type=service
sudo systemctl list-units --type=service --state=running
sudo systemctl list-units --type=service --state=failed

# Service logs
sudo journalctl -u nginx
sudo journalctl -u nginx -f     # Follow logs
sudo journalctl -u nginx --since "1 hour ago"
sudo journalctl -u nginx --since "2024-01-15 14:00:00"

# System state
sudo systemctl get-default       # Current target
sudo systemctl set-default multi-user.target
sudo systemctl isolate rescue.target
```

#### Creating Custom Services
```bash
# Create service file
sudo tee /etc/systemd/system/myapp.service << 'EOF'
[Unit]
Description=My Application
After=network.target
Requires=postgresql.service

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start.sh
ExecReload=/bin/kill -HUP $MAINPID
Restart=always
RestartSec=10
Environment=NODE_ENV=production
EnvironmentFile=/opt/myapp/.env

# Security settings
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/myapp/logs /opt/myapp/data

[Install]
WantedBy=multi-user.target
EOF

# Reload systemd and start service
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

#### Cron Jobs and Scheduling
```bash
# Edit crontab
crontab -e                      # Edit user crontab
sudo crontab -e                 # Edit root crontab
crontab -l                      # List crontab entries
crontab -r                      # Remove crontab

# Crontab format: minute hour day month weekday command
# Examples:
0 2 * * *     /home/user/backup.sh              # Daily at 2 AM
*/15 * * * *  /home/user/check_health.sh         # Every 15 minutes
0 0 1 * *     /home/user/monthly_report.sh       # Monthly on 1st
0 9 * * 1-5   /home/user/weekday_task.sh         # Weekdays at 9 AM

# System-wide cron
sudo vim /etc/crontab
ls /etc/cron.d/
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

# Anacron for systems that aren't always on
sudo vim /etc/anacrontab

# At command for one-time scheduling
echo "/home/user/task.sh" | at 15:30
echo "/home/user/task.sh" | at "15:30 tomorrow"
atq                             # List scheduled jobs
atrm 1                          # Remove job 1
```

### 4. Automation and Deployment Scripts

#### Application Deployment Script
```bash
#!/bin/bash

# Application deployment script
set -e

# Configuration
APP_NAME="myapp"
APP_USER="myapp"
APP_DIR="/opt/$APP_NAME"
BACKUP_DIR="/backups/$APP_NAME"
GIT_REPO="git@github.com:company/myapp.git"
BRANCH="main"
SERVICE_NAME="$APP_NAME"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Logging function
log() {
    echo -e "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

log_info() {
    log "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    log "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    log "${RED}[ERROR]${NC} $1"
}

# Error handling
error_exit() {
    log_error "$1"
    exit 1
}

# Check if running as root
if [ "$EUID" -eq 0 ]; then
    error_exit "This script should not be run as root"
fi

# Check if user exists
if ! id "$APP_USER" &>/dev/null; then
    error_exit "User $APP_USER does not exist"
fi

# Pre-deployment checks
pre_deployment_checks() {
    log_info "Running pre-deployment checks..."
    
    # Check if service is running
    if systemctl is-active --quiet "$SERVICE_NAME"; then
        log_info "Service $SERVICE_NAME is running"
    else
        log_warn "Service $SERVICE_NAME is not running"
    fi
    
    # Check disk space
    DISK_USAGE=$(df "$APP_DIR" | awk 'NR==2 {print $5}' | sed 's/%//')
    if [ "$DISK_USAGE" -gt 90 ]; then
        error_exit "Disk usage is $DISK_USAGE%. Deployment aborted."
    fi
    
    # Check if Git repo is accessible
    if ! git ls-remote "$GIT_REPO" &>/dev/null; then
        error_exit "Cannot access Git repository: $GIT_REPO"
    fi
    
    log_info "Pre-deployment checks passed"
}

# Create backup
create_backup() {
    log_info "Creating backup..."
    
    local backup_name="${APP_NAME}_$(date +%Y%m%d_%H%M%S)"
    local backup_path="$BACKUP_DIR/$backup_name"
    
    mkdir -p "$BACKUP_DIR"
    
    if [ -d "$APP_DIR" ]; then
        tar -czf "$backup_path.tar.gz" -C "$(dirname "$APP_DIR")" "$(basename "$APP_DIR")"
        log_info "Backup created: $backup_path.tar.gz"
    else
        log_warn "Application directory does not exist, skipping backup"
    fi
}

# Deploy application
deploy_application() {
    log_info "Deploying application..."
    
    # Stop service
    log_info "Stopping service $SERVICE_NAME..."
    sudo systemctl stop "$SERVICE_NAME" || log_warn "Failed to stop service"
    
    # Create application directory
    sudo mkdir -p "$APP_DIR"
    sudo chown "$APP_USER:$APP_USER" "$APP_DIR"
    
    # Clone or update repository
    if [ -d "$APP_DIR/.git" ]; then
        log_info "Updating existing repository..."
        cd "$APP_DIR"
        sudo -u "$APP_USER" git fetch origin
        sudo -u "$APP_USER" git reset --hard "origin/$BRANCH"
    else
        log_info "Cloning repository..."
        sudo -u "$APP_USER" git clone "$GIT_REPO" "$APP_DIR"
        cd "$APP_DIR"
        sudo -u "$APP_USER" git checkout "$BRANCH"
    fi
    
    # Install dependencies
    if [ -f "package.json" ]; then
        log_info "Installing Node.js dependencies..."
        sudo -u "$APP_USER" npm ci --production
    elif [ -f "requirements.txt" ]; then
        log_info "Installing Python dependencies..."
        sudo -u "$APP_USER" pip install -r requirements.txt
    elif [ -f "pom.xml" ]; then
        log_info "Building Java application..."
        sudo -u "$APP_USER" mvn clean package -DskipTests
    fi
    
    # Set permissions
    sudo chown -R "$APP_USER:$APP_USER" "$APP_DIR"
    sudo chmod +x "$APP_DIR/bin/"* 2>/dev/null || true
    
    log_info "Application deployed successfully"
}

# Start services
start_services() {
    log_info "Starting services..."
    
    # Start service
    sudo systemctl start "$SERVICE_NAME"
    
    # Wait for service to start
    sleep 5
    
    # Check if service is running
    if systemctl is-active --quiet "$SERVICE_NAME"; then
        log_info "Service $SERVICE_NAME started successfully"
    else
        error_exit "Failed to start service $SERVICE_NAME"
    fi
}

# Health check
health_check() {
    log_info "Performing health check..."
    
    local max_attempts=30
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f -s http://localhost:8080/health >/dev/null 2>&1; then
            log_info "Health check passed"
            return 0
        fi
        
        log_info "Health check attempt $attempt/$max_attempts failed, retrying..."
        sleep 2
        ((attempt++))
    done
    
    error_exit "Health check failed after $max_attempts attempts"
}

# Cleanup old backups
cleanup_backups() {
    log_info "Cleaning up old backups..."
    
    # Keep only last 5 backups
    find "$BACKUP_DIR" -name "${APP_NAME}_*.tar.gz" -type f | \
        sort -r | tail -n +6 | xargs -r rm -f
    
    log_info "Backup cleanup completed"
}

# Main deployment process
main() {
    log_info "Starting deployment of $APP_NAME..."
    
    pre_deployment_checks
    create_backup
    deploy_application
    start_services
    health_check
    cleanup_backups
    
    log_info "Deployment completed successfully!"
}

# Run main function
main "$@"
```

#### System Maintenance Script
```bash
#!/bin/bash

# System maintenance script
set -e

# Configuration
LOG_FILE="/var/log/system_maintenance.log"
MAX_DISK_USAGE=85
MAX_MEMORY_USAGE=90
MAX_LOAD_AVERAGE=4.0
EMAIL_RECIPIENT="admin@example.com"

# Logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# System updates
update_system() {
    log "Starting system updates..."
    
    if command -v apt >/dev/null 2>&1; then
        sudo apt update
        sudo apt upgrade -y
        sudo apt autoremove -y
        sudo apt autoclean
    elif command -v yum >/dev/null 2>&1; then
        sudo yum update -y
        sudo yum autoremove -y
        sudo yum clean all
    elif command -v dnf >/dev/null 2>&1; then
        sudo dnf update -y
        sudo dnf autoremove -y
        sudo dnf clean all
    fi
    
    log "System updates completed"
}

# Clean temporary files
clean_temp_files() {
    log "Cleaning temporary files..."
    
    # Clean /tmp (files older than 7 days)
    find /tmp -type f -atime +7 -delete 2>/dev/null || true
    
    # Clean user temp directories
    find /home/*/tmp -type f -atime +7 -delete 2>/dev/null || true
    
    # Clean log files (compress files older than 30 days)
    find /var/log -name "*.log" -type f -mtime +30 -exec gzip {} \; 2>/dev/null || true
    
    # Clean old log files (delete compressed files older than 90 days)
    find /var/log -name "*.gz" -type f -mtime +90 -delete 2>/dev/null || true
    
    log "Temporary file cleanup completed"
}

# Check disk usage
check_disk_usage() {
    log "Checking disk usage..."
    
    while read -r filesystem usage mountpoint; do
        usage_num=$(echo "$usage" | sed 's/%//')
        if [ "$usage_num" -gt "$MAX_DISK_USAGE" ]; then
            log "WARNING: Disk usage on $mountpoint is $usage (threshold: $MAX_DISK_USAGE%)"
            echo "High disk usage on $mountpoint: $usage" | \
                mail -s "Disk Usage Alert - $(hostname)" "$EMAIL_RECIPIENT" 2>/dev/null || true
        fi
    done < <(df -h | awk 'NR>1 {print $1, $5, $6}')
    
    log "Disk usage check completed"
}

# Check memory usage
check_memory_usage() {
    log "Checking memory usage..."
    
    memory_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    if [ "$memory_usage" -gt "$MAX_MEMORY_USAGE" ]; then
        log "WARNING: Memory usage is $memory_usage% (threshold: $MAX_MEMORY_USAGE%)"
        echo "High memory usage: $memory_usage%" | \
            mail -s "Memory Usage Alert - $(hostname)" "$EMAIL_RECIPIENT" 2>/dev/null || true
    fi
    
    log "Memory usage check completed"
}

# Check system load
check_system_load() {
    log "Checking system load..."
    
    load_average=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    if (( $(echo "$load_average > $MAX_LOAD_AVERAGE" | bc -l) )); then
        log "WARNING: Load average is $load_average (threshold: $MAX_LOAD_AVERAGE)"
        echo "High system load: $load_average" | \
            mail -s "Load Average Alert - $(hostname)" "$EMAIL_RECIPIENT" 2>/dev/null || true
    fi
    
    log "System load check completed"
}

# Check failed services
check_failed_services() {
    log "Checking for failed services..."
    
    failed_services=$(systemctl list-units --state=failed --no-legend | awk '{print $1}')
    if [ -n "$failed_services" ]; then
        log "WARNING: Failed services detected: $failed_services"
        echo "Failed services on $(hostname): $failed_services" | \
            mail -s "Failed Services Alert - $(hostname)" "$EMAIL_RECIPIENT" 2>/dev/null || true
    fi
    
    log "Failed services check completed"
}

# Generate system report
generate_report() {
    log "Generating system report..."
    
    report_file="/tmp/system_report_$(date +%Y%m%d).txt"
    
    {
        echo "System Report - $(date)"
        echo "=============================="
        echo
        echo "System Information:"
        hostnamectl
        echo
        echo "Uptime:"
        uptime
        echo
        echo "Disk Usage:"
        df -h
        echo
        echo "Memory Usage:"
        free -h
        echo
        echo "Top Processes by CPU:"
        ps aux --sort=-%cpu | head -10
        echo
        echo "Top Processes by Memory:"
        ps aux --sort=-%mem | head -10
        echo
        echo "Network Connections:"
        ss -tulpn | head -20
        echo
        echo "Recent Log Entries:"
        journalctl --since "24 hours ago" --no-pager | tail -20
    } > "$report_file"
    
    log "System report generated: $report_file"
}

# Main function
main() {
    log "Starting system maintenance..."
    
    update_system
    clean_temp_files
    check_disk_usage
    check_memory_usage
    check_system_load
    check_failed_services
    generate_report
    
    log "System maintenance completed"
}

# Run main function
main "$@"
```

### 5. Environment Management

#### Environment Configuration
```bash
#!/bin/bash

# Environment management script

# Environment detection
detect_environment() {
    if [ -f "/etc/environment" ]; then
        source /etc/environment
    fi
    
    # Check for environment indicators
    if [ "$HOSTNAME" = "prod-server" ] || [[ "$HOSTNAME" =~ prod-.* ]]; then
        echo "production"
    elif [ "$HOSTNAME" = "staging-server" ] || [[ "$HOSTNAME" =~ staging-.* ]]; then
        echo "staging"
    elif [ "$HOSTNAME" = "dev-server" ] || [[ "$HOSTNAME" =~ dev-.* ]]; then
        echo "development"
    else
        echo "unknown"
    fi
}

# Load environment-specific configuration
load_environment_config() {
    local env="$1"
    local config_dir="/etc/myapp"
    
    # Load base configuration
    if [ -f "$config_dir/base.conf" ]; then
        source "$config_dir/base.conf"
    fi
    
    # Load environment-specific configuration
    if [ -f "$config_dir/$env.conf" ]; then
        source "$config_dir/$env.conf"
    fi
    
    # Load local overrides
    if [ -f "$config_dir/local.conf" ]; then
        source "$config_dir/local.conf"
    fi
}

# Set environment variables
set_environment_variables() {
    local env="$1"
    
    export ENVIRONMENT="$env"
    export LOG_LEVEL="${LOG_LEVEL:-INFO}"
    export DATABASE_URL="${DATABASE_URL:-postgresql://localhost:5432/myapp_$env}"
    export REDIS_URL="${REDIS_URL:-redis://localhost:6379/0}"
    export SECRET_KEY="${SECRET_KEY:-$(openssl rand -hex 32)}"
    
    case "$env" in
        "production")
            export DEBUG="false"
            export LOG_LEVEL="WARN"
            export WORKERS="4"
            ;;
        "staging")
            export DEBUG="false"
            export LOG_LEVEL="INFO"
            export WORKERS="2"
            ;;
        "development")
            export DEBUG="true"
            export LOG_LEVEL="DEBUG"
            export WORKERS="1"
            ;;
    esac
}

# Create environment-specific directories
setup_directories() {
    local env="$1"
    local app_user="myapp"
    
    local directories=(
        "/var/log/myapp/$env"
        "/var/lib/myapp/$env"
        "/etc/myapp/$env"
        "/tmp/myapp/$env"
    )
    
    for dir in "${directories[@]}"; do
        sudo mkdir -p "$dir"
        sudo chown "$app_user:$app_user" "$dir"
        sudo chmod 755 "$dir"
    done
}

# Usage
ENVIRONMENT=$(detect_environment)
load_environment_config "$ENVIRONMENT"
set_environment_variables "$ENVIRONMENT"
setup_directories "$ENVIRONMENT"

echo "Environment: $ENVIRONMENT"
echo "Debug mode: $DEBUG"
echo "Log level: $LOG_LEVEL"
echo "Database URL: $DATABASE_URL"
```

---

## Hands-on Examples

### Example 1: Complete Application Setup Script
```bash
#!/bin/bash

# Complete application setup script
set -e

APP_NAME="mywebapp"
APP_USER="webapp"
APP_DIR="/opt/$APP_NAME"
NGINX_SITE="$APP_NAME"
DOMAIN="myapp.example.com"
GIT_REPO="https://github.com/company/mywebapp.git"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

log() {
    echo -e "[$(date +'%Y-%m-%d %H:%M:%S')] ${GREEN}[INFO]${NC} $1"
}

error() {
    echo -e "[$(date +'%Y-%m-%d %H:%M:%S')] ${RED}[ERROR]${NC} $1" >&2
    exit 1
}

# Check if running as root
[ "$EUID" -eq 0 ] || error "This script must be run as root"

# Update system
log "Updating system packages..."
apt update && apt upgrade -y

# Install required packages
log "Installing required packages..."
apt install -y nginx postgresql postgresql-contrib python3 python3-pip python3-venv git curl

# Create application user
log "Creating application user..."
if ! id "$APP_USER" &>/dev/null; then
    useradd -r -s /bin/bash -d "$APP_DIR" "$APP_USER"
fi

# Create application directory
log "Setting up application directory..."
mkdir -p "$APP_DIR"
chown "$APP_USER:$APP_USER" "$APP_DIR"

# Clone application
log "Cloning application repository..."
sudo -u "$APP_USER" git clone "$GIT_REPO" "$APP_DIR"
cd "$APP_DIR"

# Set up Python virtual environment
log "Setting up Python virtual environment..."
sudo -u "$APP_USER" python3 -m venv venv
sudo -u "$APP_USER" ./venv/bin/pip install -r requirements.txt

# Set up database
log "Setting up PostgreSQL database..."
sudo -u postgres createuser "$APP_USER"
sudo -u postgres createdb "$APP_NAME" -O "$APP_USER"

# Create environment file
log "Creating environment configuration..."
cat > "$APP_DIR/.env" << EOF
DATABASE_URL=postgresql://$APP_USER@localhost/$APP_NAME
SECRET_KEY=$(openssl rand -hex 32)
DEBUG=False
ALLOWED_HOSTS=$DOMAIN,localhost
EOF
chown "$APP_USER:$APP_USER" "$APP_DIR/.env"
chmod 600 "$APP_DIR/.env"

# Create systemd service
log "Creating systemd service..."
cat > "/etc/systemd/system/$APP_NAME.service" << EOF
[Unit]
Description=$APP_NAME Web Application
After=network.target postgresql.service
Requires=postgresql.service

[Service]
Type=simple
User=$APP_USER
Group=$APP_USER
WorkingDirectory=$APP_DIR
EnvironmentFile=$APP_DIR/.env
ExecStart=$APP_DIR/venv/bin/python manage.py runserver 127.0.0.1:8000
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Configure Nginx
log "Configuring Nginx..."
cat > "/etc/nginx/sites-available/$NGINX_SITE" << EOF
server {
    listen 80;
    server_name $DOMAIN;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
    
    location /static/ {
        alias $APP_DIR/static/;
        expires 30d;
    }
    
    location /media/ {
        alias $APP_DIR/media/;
        expires 30d;
    }
}
EOF

# Enable Nginx site
ln -sf "/etc/nginx/sites-available/$NGINX_SITE" "/etc/nginx/sites-enabled/"
nginx -t

# Start and enable services
log "Starting services..."
systemctl daemon-reload
systemctl enable "$APP_NAME"
systemctl start "$APP_NAME"
systemctl enable nginx
systemctl restart nginx

# Set up log rotation
log "Setting up log rotation..."
cat > "/etc/logrotate.d/$APP_NAME" << EOF
/var/log/$APP_NAME/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 644 $APP_USER $APP_USER
    postrotate
        systemctl reload $APP_NAME
    endscript
}
EOF

# Create backup script
log "Creating backup script..."
cat > "/usr/local/bin/backup-$APP_NAME.sh" << 'EOF'
#!/bin/bash
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR"

# Database backup
pg_dump "$APP_NAME" | gzip > "$BACKUP_DIR/${APP_NAME}_db_$DATE.sql.gz"

# Application backup
tar -czf "$BACKUP_DIR/${APP_NAME}_app_$DATE.tar.gz" -C "/opt" "$APP_NAME"

# Clean old backups (keep 7 days)
find "$BACKUP_DIR" -name "${APP_NAME}_*" -mtime +7 -delete
EOF
chmod +x "/usr/local/bin/backup-$APP_NAME.sh"

# Add backup to crontab
echo "0 2 * * * /usr/local/bin/backup-$APP_NAME.sh" | crontab -

log "Application setup completed successfully!"
log "Application URL: http://$DOMAIN"
log "Service status: systemctl status $APP_NAME"
log "Nginx status: systemctl status nginx"
log "Logs: journalctl -u $APP_NAME -f"
```

### Example 2: Monitoring and Alerting Script
```bash
#!/bin/bash

# Comprehensive monitoring script
set -e

# Configuration
CONFIG_FILE="/etc/monitoring/config.conf"
LOG_FILE="/var/log/monitoring.log"
ALERT_EMAIL="admin@example.com"
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

# Default thresholds
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90
LOAD_THRESHOLD=5.0

# Load configuration if exists
[ -f "$CONFIG_FILE" ] && source "$CONFIG_FILE"

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Send alert via email
send_email_alert() {
    local subject="$1"
    local message="$2"
    echo "$message" | mail -s "$subject" "$ALERT_EMAIL" 2>/dev/null || true
}

# Send alert via Slack
send_slack_alert() {
    local message="$1"
    local color="$2"
    
    if [ -n "$SLACK_WEBHOOK" ]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"attachments\":[{\"color\":\"$color\",\"text\":\"$message\"}]}" \
            "$SLACK_WEBHOOK" 2>/dev/null || true
    fi
}

# Check CPU usage
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    cpu_usage=${cpu_usage%.*}  # Remove decimal part
    
    if [ "$cpu_usage" -gt "$CPU_THRESHOLD" ]; then
        local message="High CPU usage detected: ${cpu_usage}% (threshold: ${CPU_THRESHOLD}%)"
        log "ALERT: $message"
        send_email_alert "CPU Alert - $(hostname)" "$message"
        send_slack_alert "🚨 $message on $(hostname)" "danger"
        return 1
    fi
    
    log "CPU usage: ${cpu_usage}% (OK)"
    return 0
}

# Check memory usage
check_memory() {
    local memory_usage=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')
    
    if [ "$memory_usage" -gt "$MEMORY_THRESHOLD" ]; then
        local message="High memory usage detected: ${memory_usage}% (threshold: ${MEMORY_THRESHOLD}%)"
        log "ALERT: $message"
        send_email_alert "Memory Alert - $(hostname)" "$message"
        send_slack_alert "🚨 $message on $(hostname)" "danger"
        return 1
    fi
    
    log "Memory usage: ${memory_usage}% (OK)"
    return 0
}

# Check disk usage
check_disk() {
    local alerts=0
    
    while read -r filesystem usage mountpoint; do
        usage_num=$(echo "$usage" | sed 's/%//')
        if [ "$usage_num" -gt "$DISK_THRESHOLD" ]; then
            local message="High disk usage detected: $mountpoint at $usage (threshold: ${DISK_THRESHOLD}%)"
            log "ALERT: $message"
            send_email_alert "Disk Alert - $(hostname)" "$message"
            send_slack_alert "🚨 $message on $(hostname)" "danger"
            ((alerts++))
        else
            log "Disk usage $mountpoint: $usage (OK)"
        fi
    done < <(df -h | awk 'NR>1 && $5!="-" {print $1, $5, $6}')
    
    return $alerts
}

# Check system load
check_load() {
    local load_1min=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
    
    if (( $(echo "$load_1min > $LOAD_THRESHOLD" | bc -l) )); then
        local message="High system load detected: $load_1min (threshold: $LOAD_THRESHOLD)"
        log "ALERT: $message"
        send_email_alert "Load Alert - $(hostname)" "$message"
        send_slack_alert "🚨 $message on $(hostname)" "danger"
        return 1
    fi
    
    log "System load: $load_1min (OK)"
    return 0
}

# Check services
check_services() {
    local services=("nginx" "postgresql" "redis" "myapp")
    local failed_services=()
    
    for service in "${services[@]}"; do
        if systemctl is-active --quiet "$service"; then
            log "Service $service: running (OK)"
        else
            log "ALERT: Service $service is not running"
            failed_services+=("$service")
        fi
    done
    
    if [ ${#failed_services[@]} -gt 0 ]; then
        local message="Failed services detected: ${failed_services[*]}"
        send_email_alert "Service Alert - $(hostname)" "$message"
        send_slack_alert "🚨 $message on $(hostname)" "danger"
        return 1
    fi
    
    return 0
}

# Check network connectivity
check_network() {
    local hosts=("8.8.8.8" "google.com" "github.com")
    local failed_hosts=()
    
    for host in "${hosts[@]}"; do
        if ping -c 1 -W 5 "$host" >/dev/null 2>&1; then
            log "Network connectivity to $host: OK"
        else
            log "ALERT: Cannot reach $host"
            failed_hosts+=("$host")
        fi
    done
    
    if [ ${#failed_hosts[@]} -gt 0 ]; then
        local message="Network connectivity issues: ${failed_hosts[*]}"
        send_email_alert "Network Alert - $(hostname)" "$message"
        send_slack_alert "🚨 $message on $(hostname)" "danger"
        return 1
    fi
    
    return 0
}

# Generate status report
generate_status_report() {
    local report_file="/tmp/status_report_$(date +%Y%m%d_%H%M%S).txt"
    
    {
        echo "System Status Report - $(date)"
        echo "====================================="
        echo
        echo "System Information:"
        echo "Hostname: $(hostname)"
        echo "Uptime: $(uptime)"
        echo "Kernel: $(uname -r)"
        echo
        echo "Resource Usage:"
        echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}')"
        echo "Memory: $(free -h | awk 'NR==2{print $3"/"$2}')"
        echo "Load Average: $(uptime | awk -F'load average:' '{print $2}')"
        echo
        echo "Disk Usage:"
        df -h
        echo
        echo "Network Connections:"
        ss -tulpn | head -10
        echo
        echo "Top Processes:"
        ps aux --sort=-%cpu | head -10
    } > "$report_file"
    
    log "Status report generated: $report_file"
    
    # Send report via email
    if [ -f "$report_file" ]; then
        mail -s "Status Report - $(hostname)" "$ALERT_EMAIL" < "$report_file" 2>/dev/null || true
    fi
}

# Main monitoring function
main() {
    log "Starting system monitoring..."
    
    local alerts=0
    
    check_cpu || ((alerts++))
    check_memory || ((alerts++))
    check_disk || ((alerts++))
    check_load || ((alerts++))
    check_services || ((alerts++))
    check_network || ((alerts++))
    
    if [ $alerts -eq 0 ]; then
        log "All checks passed - system is healthy"
        send_slack_alert "✅ System health check passed on $(hostname)" "good"
    else
        log "$alerts alerts detected"
        generate_status_report
    fi
    
    log "Monitoring completed"
}

# Run main function
main "$@"
```

---

## Best Practices

### Shell Scripting
1. **Use set -e**: Exit on any error
2. **Quote Variables**: Always quote variables to prevent word splitting
3. **Use Functions**: Break complex scripts into functions
4. **Error Handling**: Implement proper error handling and logging
5. **Documentation**: Add comments and usage instructions
6. **Testing**: Test scripts in non-production environments first

### Package Management
1. **Keep Systems Updated**: Regularly update packages for security
2. **Use Package Managers**: Prefer package managers over manual installation
3. **Pin Critical Versions**: Pin versions for production systems
4. **Repository Management**: Use trusted repositories only
5. **Dependency Tracking**: Document and track dependencies

### Service Management
1. **Use Systemd**: Leverage systemd for service management
2. **Security Settings**: Apply security settings in service files
3. **Resource Limits**: Set appropriate resource limits
4. **Monitoring**: Monitor service health and performance
5. **Graceful Shutdown**: Implement proper shutdown procedures

### Automation
1. **Idempotent Scripts**: Make scripts safe to run multiple times
2. **Configuration Management**: Use configuration files for settings
3. **Logging**: Implement comprehensive logging
4. **Rollback Procedures**: Plan for rollback scenarios
5. **Testing**: Automate testing of deployment scripts

---

## Common Mistakes

1. **Not Using set -e**: Scripts continue after errors
2. **Unquoted Variables**: Variables with spaces cause issues
3. **Hardcoded Paths**: Makes scripts inflexible
4. **No Error Handling**: Scripts fail silently
5. **Running as Root**: Unnecessary privilege escalation
6. **No Backup Strategy**: Losing data during deployments
7. **Poor Logging**: Difficult to troubleshoot issues
8. **No Testing**: Scripts fail in production
9. **Ignoring Dependencies**: Missing required packages
10. **No Monitoring**: Issues go undetected

---

## Practice Projects

### Project 1: Complete CI/CD Pipeline
**Objective**: Build a complete deployment pipeline

**Tasks**:
- Create deployment scripts for multiple environments
- Implement automated testing and quality checks
- Set up monitoring and alerting
- Create rollback procedures
- Document the entire process

### Project 2: Infrastructure Automation
**Objective**: Automate server setup and maintenance

**Tasks**:
- Create server provisioning scripts
- Implement automated security hardening
- Set up log aggregation and monitoring
- Create backup and recovery procedures
- Implement configuration management

### Project 3: Multi-Environment Management
**Objective**: Manage multiple application environments

**Tasks**:
- Create environment-specific configurations
- Implement environment promotion workflows
- Set up environment-specific monitoring
- Create data migration scripts
- Implement environment synchronization

---

## Related Levels
- **Level 1**: [Using, SSH, File Management](./level-1.md) - Linux fundamentals
- **Level 2**: [Permissions, Logs, Commands, Git](./level-2.md) - System administration basics
- **Related Topics**: Docker, Kubernetes, Ansible, Terraform, CI/CD

---

## Q&A Section

**Q: How do I make my scripts more secure?**
A: Use `set -e` for error handling, quote all variables, validate inputs, avoid running as root when possible, use temporary files securely, and implement proper logging. Consider using tools like shellcheck for static analysis.

**Q: What's the difference between systemd and traditional init systems?**
A: Systemd provides parallel startup, dependency management, better logging (journald), socket activation, and more advanced service management features compared to traditional SysV init systems.

**Q: How do I handle secrets in deployment scripts?**
A: Use environment variables, external secret management systems (HashiCorp Vault, AWS Secrets Manager), encrypted files, or configuration management tools. Never hardcode secrets in scripts.

**Q: What's the best way to test deployment scripts?**
A: Use virtual machines or containers for testing, implement dry-run modes, use configuration management tools with testing frameworks, and test in staging environments that mirror production.

**Q: How do I handle script dependencies?**
A: Check for required commands at script start, use package managers to install dependencies, document requirements clearly, and consider using containers for consistent environments.

**Q: What's the difference between cron and systemd timers?**
A: Systemd timers offer better logging, dependency management, and integration with systemd services. They're more flexible and provide better error handling than traditional cron jobs.

**Q: How do I implement proper logging in scripts?**
A: Use consistent log formats with timestamps, implement different log levels (INFO, WARN, ERROR), log to both console and files, use syslog for system integration, and implement log rotation.

**Q: What's the best way to handle script configuration?**
A: Use separate configuration files, support environment variables, provide sensible defaults, validate configuration values, and document all configuration options clearly.