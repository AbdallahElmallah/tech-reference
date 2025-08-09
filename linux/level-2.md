# Level 2: System Administration - Permissions, Logs, Commands, Git

## Prerequisites
- Completion of Level 1 (Linux basics, SSH, file management)
- Comfortable with command-line navigation and basic file operations
- Understanding of users, groups, and basic system concepts
- Git installed on your system
- Access to a Linux system with sudo privileges

## Problem Statement
As a backend engineer, you need to:
- **Manage file and directory permissions** to secure applications and data
- **Monitor and analyze system logs** for troubleshooting and security
- **Master advanced Linux commands** for efficient system administration
- **Use Git effectively** for version control and collaboration
- **Understand process management** and system monitoring
- **Configure system services** and manage system resources
- **Implement security best practices** for production environments

These skills are essential for deploying, maintaining, and securing backend applications in production environments.

---

## Key Concepts

### 1. File Permissions and Ownership

#### Understanding Permission System
```bash
# Permission format: rwxrwxrwx
# r = read (4), w = write (2), x = execute (1)
# First triplet: owner, second: group, third: others

# View permissions
ls -l file.txt
# -rw-r--r-- 1 user group 1024 Jan 15 10:30 file.txt
#  ^^^^^^^^^    ^    ^     ^    ^   ^  ^     ^
#  |||||||||    |    |     |    |   |  |     └── filename
#  |||||||||    |    |     |    |   |  └────── time
#  |||||||||    |    |     |    |   └───────── date
#  |||||||||    |    |     |    └───────────── size
#  |||||||||    |    |     └────────────────── group
#  |||||||||    |    └──────────────────────── owner
#  |||||||||    └───────────────────────────── link count
#  ||||||||└─── others permissions (r--)
#  |||||└────── group permissions (r--)
#  ||└─────────── owner permissions (rw-)
#  |└──────────── file type (- = regular file)
#  └───────────── permission string

# Numeric permissions
# 755 = rwxr-xr-x (owner: read+write+execute, group/others: read+execute)
# 644 = rw-r--r-- (owner: read+write, group/others: read only)
# 600 = rw------- (owner: read+write, no access for others)
```

#### Changing Permissions
```bash
# Using chmod with symbolic notation
chmod u+x file.txt              # Add execute for user (owner)
chmod g-w file.txt              # Remove write for group
chmod o+r file.txt              # Add read for others
chmod a+x file.txt              # Add execute for all (user+group+others)
chmod u+rw,g+r,o-rwx file.txt   # Multiple changes

# Using chmod with numeric notation
chmod 755 script.sh             # rwxr-xr-x
chmod 644 config.txt            # rw-r--r--
chmod 600 private_key           # rw-------
chmod 777 shared_file           # rwxrwxrwx (dangerous!)

# Recursive permission changes
chmod -R 755 /path/to/directory/
chmod -R u+x /path/to/scripts/

# Special permissions
chmod +t /tmp/shared            # Sticky bit (only owner can delete)
chmod g+s /shared/directory     # Set group ID
chmod u+s /usr/bin/sudo         # Set user ID (SUID)
```

#### Ownership Management
```bash
# Change ownership
chown user:group file.txt       # Change owner and group
chown user file.txt             # Change owner only
chown :group file.txt           # Change group only
chown -R user:group directory/  # Recursive ownership change

# Change group only
chgrp group file.txt
chgrp -R group directory/

# View current user and groups
whoami                          # Current user
groups                          # Groups current user belongs to
id                              # Detailed user and group info
id username                     # Info for specific user
```

### 2. System Logs and Monitoring

#### Log File Locations
```bash
# Common log locations
/var/log/syslog                 # System messages (Debian/Ubuntu)
/var/log/messages               # System messages (RedHat/CentOS)
/var/log/auth.log               # Authentication logs
/var/log/kern.log               # Kernel messages
/var/log/dmesg                  # Boot messages
/var/log/cron.log               # Cron job logs
/var/log/mail.log               # Mail server logs
/var/log/apache2/               # Apache web server logs
/var/log/nginx/                 # Nginx web server logs
/var/log/mysql/                 # MySQL database logs

# Application-specific logs
/var/log/application/           # Custom application logs
~/.local/share/logs/            # User application logs
/opt/app/logs/                  # Third-party application logs
```

#### Log Analysis Commands
```bash
# Real-time log monitoring
tail -f /var/log/syslog
tail -f /var/log/auth.log
multitail /var/log/syslog /var/log/auth.log  # Multiple files

# Search and filter logs
grep "error" /var/log/syslog
grep -i "failed login" /var/log/auth.log
grep -E "(error|warning|critical)" /var/log/syslog
grep -v "INFO" /var/log/application.log     # Exclude INFO messages

# Time-based filtering
grep "Jan 15" /var/log/syslog
grep "2024-01-15 14:" /var/log/application.log

# Advanced log analysis
awk '/error/ {print $1, $2, $3, $NF}' /var/log/syslog  # Extract specific fields
sed -n '/14:00/,/15:00/p' /var/log/syslog              # Time range

# Log statistics
grep -c "error" /var/log/syslog             # Count errors
grep "error" /var/log/syslog | wc -l        # Alternative count
cut -d' ' -f1-3 /var/log/syslog | sort | uniq -c  # Count by timestamp
```

#### System Monitoring
```bash
# System resource monitoring
top                             # Real-time process viewer
htop                            # Enhanced process viewer
iotop                           # I/O monitoring
iftop                           # Network monitoring

# Memory and CPU
free -h                         # Memory usage
cat /proc/meminfo               # Detailed memory info
cat /proc/cpuinfo               # CPU information
uptime                          # System uptime and load
w                               # Who is logged in and what they're doing

# Disk usage
df -h                           # Filesystem usage
du -sh /path/*                  # Directory sizes
lsof                            # List open files
lsof /path/to/file              # What's using a specific file

# Network monitoring
netstat -tulpn                  # Network connections
ss -tulpn                       # Modern alternative to netstat
lsof -i :80                     # What's using port 80
```

### 3. Advanced Linux Commands

#### Text Processing
```bash
# grep - pattern searching
grep "pattern" file.txt
grep -r "pattern" /path/        # Recursive search
grep -n "pattern" file.txt      # Show line numbers
grep -A 3 -B 3 "pattern" file   # Show 3 lines before/after match
grep -E "(pattern1|pattern2)"   # Extended regex

# sed - stream editor
sed 's/old/new/' file.txt       # Replace first occurrence per line
sed 's/old/new/g' file.txt      # Replace all occurrences
sed -i 's/old/new/g' file.txt   # Edit file in-place
sed -n '10,20p' file.txt        # Print lines 10-20
sed '/pattern/d' file.txt       # Delete lines matching pattern

# awk - text processing
awk '{print $1}' file.txt       # Print first column
awk '{print $1, $3}' file.txt   # Print columns 1 and 3
awk '/pattern/ {print $0}' file # Print lines matching pattern
awk '{sum+=$1} END {print sum}' # Sum first column
awk -F: '{print $1}' /etc/passwd # Use : as field separator

# sort and uniq
sort file.txt                   # Sort lines
sort -n numbers.txt             # Numeric sort
sort -r file.txt                # Reverse sort
sort -k2 file.txt               # Sort by second column
uniq file.txt                   # Remove duplicate lines
sort file.txt | uniq -c         # Count occurrences

# cut - extract columns
cut -d: -f1 /etc/passwd         # Extract first field (username)
cut -c1-10 file.txt             # Extract characters 1-10
cut -d, -f2,4 data.csv          # Extract columns 2 and 4 from CSV
```

#### File Operations
```bash
# find - powerful file search
find /path -name "*.txt"        # Find by name
find /path -type f -size +1M    # Files larger than 1MB
find /path -mtime -7            # Modified in last 7 days
find /path -user username       # Files owned by user
find /path -perm 755            # Files with specific permissions
find /path -name "*.log" -exec rm {} \;  # Find and delete

# xargs - build command lines
find /path -name "*.tmp" | xargs rm     # Delete found files
echo "file1 file2 file3" | xargs ls -l # Apply ls to each file
cat filelist.txt | xargs chmod 644      # Apply chmod to files in list

# tar - archive operations
tar -czf archive.tar.gz directory/      # Create compressed archive
tar -xzf archive.tar.gz                 # Extract compressed archive
tar -tzf archive.tar.gz                 # List archive contents
tar -czf backup-$(date +%Y%m%d).tar.gz /important/data/

# rsync - efficient file synchronization
rsync -av source/ destination/          # Archive mode, verbose
rsync -av --delete source/ dest/        # Delete files not in source
rsync -av -e ssh source/ user@host:dest/ # Sync over SSH
rsync -av --exclude="*.tmp" source/ dest/ # Exclude patterns
```

### 4. Process and Service Management

#### Process Control
```bash
# Process information
ps aux                          # All processes
ps aux | grep nginx             # Find specific process
pgrep nginx                     # Get PID by name
pidof nginx                     # Alternative to pgrep

# Process tree
pstree                          # Show process hierarchy
pstree -p                       # Show with PIDs
pstree username                 # Show for specific user

# Killing processes
kill PID                        # Terminate process
kill -9 PID                     # Force kill
killall nginx                   # Kill all nginx processes
pkill -f "python script.py"     # Kill by command pattern

# Process priority
nice -n 10 command              # Run with lower priority
renice 5 PID                    # Change priority of running process

# Background jobs
command &                       # Run in background
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
bg %1                           # Send job 1 to background
disown %1                       # Detach job from shell
```

#### Service Management (systemd)
```bash
# Service status and control
sudo systemctl status nginx     # Check service status
sudo systemctl start nginx      # Start service
sudo systemctl stop nginx       # Stop service
sudo systemctl restart nginx    # Restart service
sudo systemctl reload nginx     # Reload configuration

# Enable/disable services
sudo systemctl enable nginx     # Enable at boot
sudo systemctl disable nginx    # Disable at boot
sudo systemctl is-enabled nginx # Check if enabled

# Service logs
sudo journalctl -u nginx        # View service logs
sudo journalctl -u nginx -f     # Follow service logs
sudo journalctl -u nginx --since "1 hour ago"
sudo journalctl -u nginx --since "2024-01-15 14:00:00"

# List services
sudo systemctl list-units --type=service
sudo systemctl list-units --type=service --state=running
sudo systemctl list-units --type=service --state=failed
```

### 5. Git Version Control

#### Git Configuration
```bash
# Initial setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main
git config --global core.editor nano

# View configuration
git config --list
git config user.name
git config --global --list

# SSH key setup for Git
ssh-keygen -t ed25519 -C "your.email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub  # Copy this to GitHub/GitLab
```

#### Basic Git Operations
```bash
# Repository initialization
git init                        # Initialize new repository
git clone https://github.com/user/repo.git  # Clone repository
git clone git@github.com:user/repo.git      # Clone with SSH

# Basic workflow
git status                      # Check repository status
git add file.txt                # Stage specific file
git add .                       # Stage all changes
git add -A                      # Stage all changes including deletions
git commit -m "Commit message"  # Commit staged changes
git commit -am "Message"        # Stage and commit modified files

# Viewing history
git log                         # View commit history
git log --oneline               # Compact log
git log --graph --oneline       # Visual branch history
git log -p                      # Show patches (diffs)
git log --since="2 weeks ago"   # Time-based filtering
git show commit_hash            # Show specific commit

# Working with remotes
git remote -v                   # List remotes
git remote add origin https://github.com/user/repo.git
git push origin main            # Push to remote
git pull origin main            # Pull from remote
git fetch origin                # Fetch without merging
```

#### Branching and Merging
```bash
# Branch operations
git branch                      # List local branches
git branch -a                   # List all branches
git branch feature-branch       # Create new branch
git checkout feature-branch     # Switch to branch
git checkout -b new-branch      # Create and switch to branch
git switch main                 # Modern way to switch branches
git switch -c new-branch        # Create and switch (modern)

# Merging
git checkout main
git merge feature-branch        # Merge feature branch into main
git merge --no-ff feature-branch # Force merge commit

# Rebasing
git rebase main                 # Rebase current branch onto main
git rebase -i HEAD~3            # Interactive rebase last 3 commits

# Branch cleanup
git branch -d feature-branch    # Delete merged branch
git branch -D feature-branch    # Force delete branch
git push origin --delete feature-branch  # Delete remote branch
```

#### Advanced Git Operations
```bash
# Stashing changes
git stash                       # Stash current changes
git stash push -m "Work in progress"  # Stash with message
git stash list                  # List stashes
git stash pop                   # Apply and remove latest stash
git stash apply stash@{1}       # Apply specific stash
git stash drop stash@{1}        # Delete specific stash

# Undoing changes
git checkout -- file.txt        # Discard changes to file
git reset HEAD file.txt         # Unstage file
git reset --soft HEAD~1         # Undo last commit, keep changes staged
git reset --hard HEAD~1         # Undo last commit, discard changes
git revert commit_hash          # Create new commit that undoes changes

# Tagging
git tag v1.0.0                  # Create lightweight tag
git tag -a v1.0.0 -m "Version 1.0.0"  # Create annotated tag
git push origin v1.0.0          # Push tag to remote
git push origin --tags          # Push all tags

# Searching and blame
git grep "function_name"        # Search for text in repository
git blame file.txt              # Show who changed each line
git log -S "function_name"      # Find commits that added/removed text
```

---

## Hands-on Examples

### Example 1: Setting Up Secure File Permissions
```bash
# Create a web application directory structure
sudo mkdir -p /var/www/myapp/{public,private,logs,config}
cd /var/www/myapp

# Set appropriate ownership
sudo chown -R www-data:www-data /var/www/myapp
sudo chown -R root:www-data /var/www/myapp/config

# Set secure permissions
sudo chmod 755 /var/www/myapp                    # Directory accessible
sudo chmod 755 /var/www/myapp/public             # Public files readable
sudo chmod 750 /var/www/myapp/private            # Private files restricted
sudo chmod 750 /var/www/myapp/logs               # Logs writable by group
sudo chmod 640 /var/www/myapp/config/*           # Config files read-only

# Create and secure sensitive files
sudo touch /var/www/myapp/config/database.conf
echo "db_password=secret123" | sudo tee /var/www/myapp/config/database.conf
sudo chmod 600 /var/www/myapp/config/database.conf
sudo chown root:root /var/www/myapp/config/database.conf

# Verify permissions
ls -la /var/www/myapp/
ls -la /var/www/myapp/config/
```

### Example 2: Log Analysis and Monitoring Setup
```bash
# Create log analysis script
cat << 'EOF' > ~/log_analyzer.sh
#!/bin/bash

# Log analysis script
LOG_FILE="/var/log/nginx/access.log"
DATE=$(date +"%Y-%m-%d")

echo "=== Nginx Log Analysis for $DATE ==="
echo

# Top 10 IP addresses
echo "Top 10 IP addresses:"
awk '{print $1}' $LOG_FILE | sort | uniq -c | sort -nr | head -10
echo

# Top 10 requested pages
echo "Top 10 requested pages:"
awk '{print $7}' $LOG_FILE | sort | uniq -c | sort -nr | head -10
echo

# HTTP status codes
echo "HTTP status codes:"
awk '{print $9}' $LOG_FILE | sort | uniq -c | sort -nr
echo

# Error count (4xx and 5xx)
ERROR_COUNT=$(awk '$9 ~ /^[45]/ {count++} END {print count+0}' $LOG_FILE)
echo "Total errors (4xx/5xx): $ERROR_COUNT"

# Recent errors
echo "Recent errors:"
awk '$9 ~ /^[45]/ {print $0}' $LOG_FILE | tail -5
EOF

chmod +x ~/log_analyzer.sh

# Run the analysis
~/log_analyzer.sh

# Set up automated monitoring
# Add to crontab: 0 */6 * * * /home/user/log_analyzer.sh > /home/user/log_report.txt
```

### Example 3: Git Workflow for Team Development
```bash
# Initialize project repository
mkdir myproject && cd myproject
git init

# Create initial project structure
mkdir -p src/{main,test} docs config
touch README.md .gitignore
echo "# My Project" > README.md

# Create .gitignore
cat << 'EOF' > .gitignore
# Dependencies
node_modules/
*.jar

# Build outputs
build/
dist/
target/

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Environment files
.env
.env.local

# Logs
*.log
logs/
EOF

# Initial commit
git add .
git commit -m "Initial project setup"

# Add remote repository
git remote add origin git@github.com:username/myproject.git
git push -u origin main

# Create feature branch
git checkout -b feature/user-authentication

# Make changes
echo "class UserAuth {}" > src/main/UserAuth.java
git add src/main/UserAuth.java
git commit -m "Add user authentication class"

# Push feature branch
git push -u origin feature/user-authentication

# Switch back to main and merge
git checkout main
git merge feature/user-authentication
git push origin main

# Clean up
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

---

## Best Practices

### File Permissions
1. **Principle of Least Privilege**: Give minimum necessary permissions
2. **Regular Audits**: Periodically review file permissions
3. **Secure Defaults**: Use 644 for files, 755 for directories
4. **Sensitive Data**: Use 600 for private keys and config files
5. **Web Applications**: Follow web server security guidelines

### Log Management
1. **Centralized Logging**: Use tools like rsyslog or journald
2. **Log Rotation**: Implement logrotate to manage disk space
3. **Real-time Monitoring**: Set up alerts for critical errors
4. **Retention Policies**: Define how long to keep different types of logs
5. **Security Logs**: Monitor authentication and access logs

### Git Workflow
1. **Meaningful Commits**: Write clear, descriptive commit messages
2. **Feature Branches**: Use branches for new features and bug fixes
3. **Regular Commits**: Commit small, logical changes frequently
4. **Code Reviews**: Use pull requests for team collaboration
5. **Backup Strategy**: Push to remote repositories regularly

---

## Common Mistakes

1. **Overly Permissive Permissions**: Using 777 permissions unnecessarily
2. **Ignoring Log Files**: Not monitoring logs for errors and security issues
3. **Poor Git Hygiene**: Large commits, unclear messages, not using branches
4. **Running as Root**: Using root privileges when not necessary
5. **Not Backing Up**: Losing work due to inadequate backup strategies
6. **Hardcoded Credentials**: Storing passwords in configuration files
7. **Ignoring Security Updates**: Not keeping system packages updated
8. **Poor Process Management**: Not properly managing background processes

---

## Practice Projects

### Project 1: System Monitoring Dashboard
**Objective**: Create a comprehensive system monitoring solution

**Tasks**:
- Set up log rotation for application logs
- Create scripts to monitor disk usage, memory, and CPU
- Implement alerting for critical system events
- Use Git to version control monitoring scripts
- Set up automated reports via email or Slack

### Project 2: Secure Application Deployment
**Objective**: Deploy a web application with proper security

**Tasks**:
- Set up proper file permissions for web application
- Configure log files with appropriate access controls
- Implement user and group management
- Create deployment scripts using Git hooks
- Set up SSL certificates and secure configurations

### Project 3: Team Development Workflow
**Objective**: Establish a professional Git workflow

**Tasks**:
- Set up Git repository with proper branching strategy
- Implement pre-commit hooks for code quality
- Create documentation for team Git workflow
- Set up automated testing and deployment
- Practice conflict resolution and code reviews

---

## Related Levels
- **Level 1**: [Using, SSH, File Management](./level-1.md) - Linux fundamentals
- **Level 3**: [Scripting, Installation](./level-3.md) - Automation and advanced administration
- **Related Topics**: Docker, System Administration, DevOps, Security

---

## Q&A Section

**Q: What's the difference between chmod 755 and 644?**
A: 755 (rwxr-xr-x) gives execute permission to owner and read+execute to group/others - used for directories and executable files. 644 (rw-r--r--) gives read+write to owner and read-only to group/others - used for regular files.

**Q: How do I find which process is using a specific port?**
A: Use `sudo lsof -i :PORT` or `sudo netstat -tulpn | grep PORT` or `sudo ss -tulpn | grep PORT`. For example: `sudo lsof -i :80` shows what's using port 80.

**Q: What's the difference between git merge and git rebase?**
A: Merge creates a new commit that combines two branches, preserving history. Rebase replays commits from one branch onto another, creating a linear history. Use merge for feature integration, rebase for cleaning up local commits.

**Q: How do I recover a deleted file in Git?**
A: If committed: `git log --oneline --follow -- filename` to find the commit, then `git checkout commit_hash -- filename`. If not committed but staged: `git reset HEAD filename` then `git checkout -- filename`.

**Q: Why can't I delete a file even with sudo?**
A: Check if the file has immutable attribute (`lsattr filename`), if a process is using it (`lsof filename`), or if it's on a read-only filesystem (`mount | grep ro`). Remove immutable attribute with `sudo chattr -i filename`.

**Q: How do I monitor logs in real-time for multiple services?**
A: Use `sudo journalctl -f -u service1 -u service2` for systemd services, or `multitail /var/log/file1 /var/log/file2` for log files. You can also use `tail -f /var/log/file1 /var/log/file2`.

**Q: What's the best way to handle sensitive data in Git?**
A: Never commit sensitive data. Use environment variables, separate config files (in .gitignore), or tools like git-crypt or git-secret. If accidentally committed, use `git filter-branch` or BFG Repo-Cleaner to remove from history.

**Q: How do I troubleshoot permission denied errors?**
A: Check file permissions (`ls -l`), ownership (`ls -l`), parent directory permissions, SELinux context (`ls -Z`), and whether you're in the right group (`groups`). Use `sudo` if administrative access is needed.