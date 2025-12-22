# real time shell scripts usecases with example codes

{% stepper %}
{% step %}
### Automated Backup Script

Create a script to automatically backup files or directories to a specified location.

Example Code:

```bash
#!/bin/bash
backup_dir="/path/to/backup"
source_dir="/path/to/source"
timestamp=$(date +"%Y%m%d%H%M%S")
tar -czf "$backup_dir/backup_$timestamp.tar.gz" "$source_dir"
```
{% endstep %}

{% step %}
### Log Rotation Script

Write a script to rotate log files to prevent them from growing too large.

Example Code:

```bash
#!/bin/bash
log_file="/path/to/logfile.log"
max_size=1000000 # 1MB
if [ $(wc -c < "$log_file") -gt $max_size ]; then
    mv "$log_file" "$log_file.old"
    touch "$log_file"
fi
```
{% endstep %}

{% step %}
### System Monitoring Script

Develop a script to monitor system resource usage and send alerts if thresholds are exceeded.

Example Code:

```bash
#!/bin/bash
cpu_threshold=90
mem_threshold=90
cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
mem_usage=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
if (( $(echo "$cpu_usage > $cpu_threshold" | bc -l) )) || (( $(echo "$mem_usage > $mem_threshold" | bc -l) )); then
    echo "High CPU or memory usage detected!"
    # Send alert here
fi
```
{% endstep %}

{% step %}
### Database Backup Script

Write a script to backup a database and compress the backup file.

Example Code (for MySQL):

```bash
#!/bin/bash
db_user="username"
db_pass="password"
db_name="database_name"
backup_dir="/path/to/backup"
timestamp=$(date +"%Y%m%d%H%M%S")
mysqldump -u "$db_user" -p"$db_pass" "$db_name" | gzip > "$backup_dir/db_backup_$timestamp.sql.gz"
```
{% endstep %}

{% step %}
### File Transfer Script

Develop a script to transfer files securely between servers using SCP or SFTP.

Example Code (for SCP):

```bash
#!/bin/bash
source_file="/path/to/source/file"
destination_server="user@hostname:/path/to/destination/"
scp "$source_file" "$destination_server"
```
{% endstep %}

{% step %}
### User Account Management Script

Create a script to add, modify, or delete user accounts on a Linux system.

Example Code:

```bash
#!/bin/bash
action="$1"
username="$2"
case $action in
    "add")
        useradd "$username"
        ;;
    "modify")
        usermod -s /bin/bash "$username"
        ;;
    "delete")
        userdel "$username"
        ;;
    *)
        echo "Usage: $0 {add|modify|delete} username"
        exit 1
        ;;
esac
```
{% endstep %}

{% step %}
### Web Server Log Analysis Script

Write a script to analyze web server logs and generate reports on traffic, errors, etc.

Example Code:

```bash
#!/bin/bash
access_log="/var/log/apache/access.log"
error_log="/var/log/apache/error.log"

# Analyze access log
echo "Top 10 IP addresses:"
awk '{print $1}' "$access_log" | sort | uniq -c | sort -nr | head -n 10

# Analyze error log
echo "Errors by type:"
awk '{print $9}' "$error_log" | sort | uniq -c | sort -nr
```
{% endstep %}

{% step %}
### Automated Deployment Script

Develop a script to automate the deployment of applications or configurations to multiple servers.

Example Code:

```bash
#!/bin/bash
servers=("server1" "server2" "server3")
for server in "${servers[@]}"; do
    scp "app.tar.gz" "user@$server:/path/to/destination/"
    ssh "user@$server" "tar -xzvf /path/to/destination/app.tar.gz -C /path/to/app"
    # Additional deployment steps here
done
```
{% endstep %}

{% step %}
### Continuous Integration Script

Create a script to run tests and build artifacts as part of a continuous integration process.

Example Code:

```bash
#!/bin/bash

# Run tests
pytest

# Build artifact
npm run build
```
{% endstep %}

{% step %}
### Resource Monitoring Script

Write a script to monitor resource usage of specific processes and take actions if limits are exceeded.

Example Code:

```bash
#!/bin/bash
process_name="my_process"
max_memory=1000000 # 1MB
pid=$(pgrep "$process_name")
if [ -n "$pid" ]; then
    mem_usage=$(ps -p "$pid" -o rss=)
    if [ "$mem_usage" -gt "$max_memory" ]; then
        kill "$pid"
        echo "Process $process_name killed due to high memory usage"
    fi
fi
```
{% endstep %}

{% step %}
### Email Notification Script

Develop a script to send email notifications for various events or alerts.

Example Code:

```bash
#!/bin/bash
subject="Alert: High CPU Usage"
body="CPU usage exceeded 90%"
recipient="user@example.com"
echo "$body" | mail -s "$subject" "$recipient"
```
{% endstep %}

{% step %}
### System Maintenance Script

Create a script to perform routine system maintenance tasks like cleaning up temporary files, updating packages, etc.

Example Code:

```bash
#!/bin/bash

# Clean up temporary files
rm -rf /tmp/*

# Update packages
apt-get update && apt-get upgrade -y
```
{% endstep %}

{% step %}
### Data Encryption Script

Write a script to encrypt sensitive data using GPG or OpenSSL.

Example Code (using GPG):

```bash
#!/bin/bash
gpg --encrypt --recipient recipient@example.com sensitive_file.txt
```
{% endstep %}

{% step %}
### File Integrity Checking Script

Develop a script to check the integrity of files using checksums or cryptographic hashes.

Example Code:

```bash
#!/bin/bash
file="/path/to/file"
checksum=$(md5sum "$file" | awk '{print $1}')
if [ "$checksum" == "expected_checksum" ]; then
    echo "File integrity verified"
else
    echo "File integrity compromised"
fi
```
{% endstep %}

{% step %}
### Service Restart Script

Create a script to restart a service if it becomes unresponsive or crashes.

Example Code:

```bash
#!/bin/bash
service_name="myservice"
if systemctl is-active --quiet "$service_name"; then
    systemctl restart "$service_name"
    echo "Service $service_name restarted"
else
    echo "Service $service_name is not running"
fi
```
{% endstep %}

{% step %}
### Disk Space Monitoring Script

Write a script to monitor disk space usage and send alerts if thresholds are exceeded.

Example Code:

```bash
#!/bin/bash
threshold=90
disk_usage=$(df -h | grep "/dev/sda1" | awk '{print $5}' | tr -d '%')
if [ "$disk_usage" -gt "$threshold" ]; then
    echo "Disk space usage exceeded $threshold%"
    # Send alert here
fi
```
{% endstep %}

{% step %}
### Service Status Monitoring Script

Develop a script to monitor the status of critical services and take actions if they are down.

Example Code:

```bash
#!/bin/bash
services=("apache2" "mysql" "ssh")
for service in "${services[@]}"; do
    if systemctl is-active --quiet "$service"; then
        echo "Service $service is running"
    else
        echo "Service $service is not running"
        # Restart service or send alert here
    fi
done
```
{% endstep %}

{% step %}
### Remote Command Execution Script

Create a script to execute commands on remote servers using SSH.

Example Code:

```bash
#!/bin/bash
server="user@remotehost"
command="uptime"
ssh "$server" "$command"
```
{% endstep %}

{% step %}
### Database Backup Rotation Script

Write a script to rotate database backups to prevent them from accumulating indefinitely.

Example Code:

```bash
#!/bin/bash
backup_dir="/path/to/backups"
max_backups=5
backups=$(ls -t "$backup_dir" | wc -l)
if [ "$backups" -gt "$max_backups" ]; then
    oldest_backup=$(ls -t "$backup_dir" | tail -n 1)
    rm "$backup_dir/$oldest_backup"
fi
```
{% endstep %}

{% step %}
### Automated Certificate Renewal Script

Develop a script to automatically renew SSL certificates before they expire.

Example Code:

```bash
#!/bin/bash
certbot renew --quiet
```
{% endstep %}

{% step %}
### File Synchronization Script

Create a script to synchronize files or directories between local and remote locations.

Example Code:

```bash
#!/bin/bash
local_dir="/path/to/local/dir"
remote_dir="user@remotehost:/path/to/remote/dir"
rsync -avz "$local_dir" "$remote_dir"
```
{% endstep %}

{% step %}
### Backup Verification Script

Write a script to verify the integrity of backup files by comparing checksums.

Example Code:

```bash
#!/bin/bash
backup_file="/path/to/backup.tar.gz"
expected_checksum="expected_checksum"
actual_checksum=$(md5sum "$backup_file" | awk '{print $1}')
if [ "$actual_checksum" == "$expected_checksum" ]; then
    echo "Backup integrity verified"
else
    echo "Backup integrity compromised"
fi
```
{% endstep %}

{% step %}
### DNS Record Update Script

Develop a script to update DNS records dynamically using a DNS provider’s API.

Example Code:

```bash
#!/bin/bash
domain="example.com"
record_name="www"
ip=$(curl -s https://api.ipify.org)
curl -X PUT -H "Content-Type: application/json" -d "{\"data\":\"$ip\"}" "https://api.cloudflare.com/client/v4/zones/$zone_id/dns_records/$record_id"
```
{% endstep %}

{% step %}
### Server Health Check Script

Create a script to perform health checks on servers and services periodically.

Example Code:

```bash
#!/bin/bash
servers=("server1" "server2" "server3")
for server in "${servers[@]}"; do
    if ping -c 1 "$server" &> /dev/null; then
        echo "Server $server is reachable"
    else
        echo "Server $server is unreachable"
        # Send alert here
    fi
done
```
{% endstep %}

{% step %}
### Git Repository Backup Script

Write a script to backup Git repositories by cloning them to a backup location.

Example Code:

```bash
#!/bin/bash
git_repo="https://github.com/user/repo.git"
backup_dir="/path/to/backups"
git clone --mirror "$git_repo" "$backup_dir/repo.git"
```
{% endstep %}

{% step %}
### Log Analysis and Reporting Script

Develop a script to analyze logs, extract relevant data, and generate reports.

Example Code:

```bash
#!/bin/bash
log_file="/path/to/logfile.log"
error_count=$(grep -c "ERROR" "$log_file")
warning_count=$(grep -c "WARNING" "$log_file")
echo "Error count: $error_count"
echo "Warning count: $warning_count"
```
{% endstep %}

{% step %}
### File Encryption and Decryption Script

Create a script to encrypt and decrypt files using GPG or OpenSSL.

Example Code:

```bash
#!/bin/bash
gpg --output encrypted_file.txt.gpg --encrypt --recipient recipient@example.com plaintext_file.txt
gpg --output decrypted_file.txt --decrypt encrypted_file.txt.gpg
```
{% endstep %}

{% step %}
### Service Dependency Checking Script

Write a script to check for dependencies before starting a service.

Example Code:

```bash
#!/bin/bash
if ! command -v "dependency_command" &> /dev/null; then
    echo "Dependency not found, exiting"
    exit 1
fi
```
{% endstep %}

{% step %}
### Password Rotation Script

Develop a script to rotate passwords for user accounts at regular intervals.

Example Code:

```bash
#!/bin/bash
user="username"
new_password=$(openssl rand -base64 12)
echo "$user:$new_password" | chpasswd
```
{% endstep %}

{% step %}
### Automated Deployment Rollback Script

Create a script to rollback a deployment to a previous version in case of issues.

Example Code:

```bash
#!/bin/bash
previous_version="v1"
kubectl set image deployment/nginx-deployment nginx=nginx:"$previous_version" --record
```
{% endstep %}
{% endstepper %}
