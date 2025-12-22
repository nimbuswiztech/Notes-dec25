# Day 4

## DevOps Linux Commands: Complete Guide with Practical Use Cases

### Part 1: File Ownership and Permissions

#### chown - Change File Owner

**Commands:** `chown new_name file` | `chown new_name:new_grp file`

**DevOps Use Cases:**

1.  **Application Deployment Pipeline**

    ```bash
    chown ubuntu:webapp /opt/application/config.yml
    ```

    After deploying an application artifact, you need to ensure the web application user owns the config files. This prevents permission errors when the application process tries to read configuration.
2.  **Docker Container Access Control**

    ```bash
    chown -R appuser:appgroup /home/appuser/workspace
    ```

    When mounting volumes into containers, change ownership recursively so the containerized process can access files without permission denied errors.
3.  **Backup Restore Operations**

    ```bash
    chown ubuntu:ubuntu /home/ubuntu/restored_backup/
    ```

    After restoring files from backup, reassign ownership to ensure the correct user can access them. This is critical in disaster recovery scenarios.
4.  **Multi-tenant Infrastructure**

    ```bash
    chown customer1:tenant1_group /var/www/customer1/
    chown customer2:tenant2_group /var/www/customer2/
    ```

    In a multi-tenant Kubernetes setup, assign file ownership per tenant to maintain isolation and security boundaries.

***

### Part 2: Text Processing - sed (Stream Editor)

#### sed - On-the-fly Text Replacement

**Basic Syntax:** `sed 's/old_pattern/new_pattern/g' filename`

#### Global Replacement: sed 's/old\_pattern/new\_pattern/g'

**Command:** `sed 's/samsung/sony/g' newfile`

**DevOps Use Cases:**

1.  **Configuration Templating in CI/CD**

    ```bash
    sed 's/ENVIRONMENT=dev/ENVIRONMENT=prod/g' app.conf > app.conf.prod
    ```

    Before deploying to production, replace environment-specific values in config files without manual editing.
2.  **Log Sanitization**

    ```bash
    sed 's/password=[^[:space:]]*/password=****/g' debug.log
    ```

    Remove sensitive data from logs before shipping them to centralized logging (ELK Stack, Splunk).
3.  **Batch Configuration Updates**

    ```bash
    find /etc/app/configs -name "*.yaml" -exec sed -i 's/old_database_url/new_database_url/g' {} \;
    ```

    Update database connection strings across multiple environment config files in one operation.

#### In-Place Editing: sed -i

**Command:** `sed -i 's/Testing/development/g' file2`

**DevOps Use Cases:**

1.  **Infrastructure as Code (IaC) Updates**

    ```bash
    sed -i 's/instance_type = "t2.medium"/instance_type = "t3.medium"/g' terraform/main.tf
    ```

    Update Terraform files in-place to scale up instance types across environments.
2.  **Kubernetes Manifest Updates**

    ```bash
    sed -i 's/image: app:v1.0/image: app:v1.1/g' deployment.yaml
    ```

    Update container image versions in Kubernetes manifests before applying with `kubectl apply`.
3.  **Jenkins Pipeline Configuration**

    ```bash
    sed -i "s/BUILD_NUMBER=.*/BUILD_NUMBER=${BUILD_NUMBER}/g" .env
    ```

    Update environment variables in-place during CI/CD pipeline execution.

#### Case-Insensitive Replacement: sed 's/pattern/replacement/gi'

**Command:** `sed 's/testing/development/gi' file2`

**DevOps Use Cases:**

1.  **Application Log Filtering (Case-Insensitive)**

    ```bash
    sed 's/error\|ERROR\|Error/CRITICAL_ERROR/gi' app.log > sanitized.log
    ```

    Normalize error messages regardless of case in application logs for consistent monitoring alerts.

#### Replacement on Specific Line: sed '2s/pattern/replacement/gi'

**Command:** `sed '2s/testing/development/gi' file2`

**DevOps Use Cases:**

1.  **Surgical Config File Updates**

    ```bash
    sed '3s/debug=true/debug=false/gi' docker-compose.yml
    ```

    Update only the 3rd line where the debug flag is set, avoiding unintended changes elsewhere.
2.  **Kubernetes Service Port Updates**

    ```bash
    sed '5s/port: 8080/port: 9090/gi' service.yaml
    ```

    Change a specific service port definition on line 5 without touching other configurations.

#### Range-Based Replacement: sed '10,30s/pattern/replacement/gi'

**Command:** `sed '10,30s/testing/development/gi' file2`

**DevOps Use Cases:**

1.  **Large Configuration Files**

    ```bash
    sed '50,100s/old_domain/new_domain/gi' nginx.conf > nginx.conf.updated
    ```

    Update domain references only in the server block (lines 50-100) of a large Nginx config file.
2.  **Log File Analysis**

    ```bash
    sed '1000,2000s/WARN/CRITICAL/gi' system.log
    ```

    Update severity levels only for a specific section of logs during a troubleshooting window.

#### From Line to End: sed '30,$s/pattern/replacement/gi'

**Command:** `sed '30,$s/testing/development/gi' file2`

**DevOps Use Cases:**

1.  **Configuration Migration**

    ```bash
    sed '20,$s/mysql_host=localhost/mysql_host=10.0.1.5/gi' config.yaml
    ```

    Update all database references from line 20 onwards in a configuration file during infrastructure migration.

#### Print Specific Line: sed -n '98p'

**Commands:** `sed -n '98p' file` | `sed -n '2p' file2`

**DevOps Use Cases:**

1.  **Extract Build Information**

    ```bash
    sed -n '5p' build.log | grep -o "v[0-9]\+\.[0-9]\+\.[0-9]\+"
    ```

    Extract version information from a specific line in build logs for versioning purposes.
2.  **Kubernetes Event Extraction**

    ```bash
    kubectl describe pod mypod | sed -n '50p'
    ```

    Extract a specific line from pod events for debugging.

#### Print Line Range: sed -n '10,20p'

**Command:** `sed -n '10,20p' file`

**DevOps Use Cases:**

1.  **Log Excerpt Extraction**

    ```bash
    sed -n '1000,1050p' /var/log/syslog > error_context.log
    ```

    Extract 50 lines of context around an error timestamp for detailed analysis.
2.  **Terraform Plan Analysis**

    ```bash
    terraform plan -out=tfplan > plan.txt
    sed -n '10,50p' plan.txt
    ```

    Review only the resource changes section of Terraform plan output.

#### Delete Specific Line: sed '2d'

**Command:** `sed '2d' file2`

**DevOps Use Cases:**

1.  **Remove Configuration Comments**

    ```bash
    sed '1d' app.conf
    ```

    Remove a deprecated configuration line that breaks newer application versions.
2.  **Clean Kubernetes Manifest**

    ```bash
    sed '15d' deployment.yaml
    ```

    Remove a specific annotation or label from a Kubernetes manifest.

#### Delete Line Number: sed '100d'

**Command:** `sed '100d' file`

**DevOps Use Cases:**

1.  **Fix Malformed Config**

    ```bash
    sed '45d' broken_config.yaml | kubectl apply -f -
    ```

    Remove a problematic line from config file and apply directly to Kubernetes.

***

### Part 3: Column Extraction - cut

#### Extract Single Column: cut -d " " -f1

**Commands:** `cut -d " " -f1 filename` | `cut -d " " -f3 filename`

**DevOps Use Cases:**

1.  **Extract IPs from Server List**

    ```bash
    ps aux | grep java | cut -d " " -f1
    ```

    Extract usernames running Java processes for audit purposes.
2.  **Parse AWS EC2 Instance IDs**

    ```bash
    aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId]' --output text | tr '\t' '\n' | cut -d " " -f1
    ```

    Extract instance IDs for batch operations.

#### Multiple Columns: cut -d " " -f1,2

**Command:** `cut -d " " -f1,2 filename`

**DevOps Use Cases:**

1.  **Extract User and PID from Process List**

    ```bash
    ps aux | cut -d " " -f1,2
    ```

    Extract username and PID pairs for process killing scripts in automation.
2.  **Parse HTTP Access Logs**

    ```bash
    cut -d " " -f1,9 access.log
    ```

    Extract IP addresses and HTTP status codes for traffic analysis.

***

### Part 4: Advanced Text Processing - awk

#### Print Specific Field: awk -F " " '{print $2}'

**Commands:** `awk -F " " '{print $2}' test1` | `awk -F " " '{print $3}' test1`

**DevOps Use Cases:**

1.  **Extract Memory Usage from top**

    ```bash
    top -bn1 | grep "^%Mem" | awk -F " " '{print $2}'
    ```

    Extract memory percentage for monitoring and alerting in Prometheus exporter scripts.
2.  **Parse Kubernetes Pod Resources**

    ```bash
    kubectl top pods | awk -F " " '{print $2}'
    ```

    Extract CPU usage per pod for capacity planning.

#### Last Field: awk -F " " '{print $NF}'

**Command:** `awk -F " " '{print $NF}' test1`

**DevOps Use Cases:**

1.  **Extract Process Names**

    ```bash
    ps aux | awk -F " " '{print $NF}' | sort | uniq -c
    ```

    Count frequency of running processes for anomaly detection.
2.  **Get Domain from URLs in Logs**

    ```bash
    cat access.log | awk -F " " '{print $NF}' | sort | uniq -c | sort -rn
    ```

    Identify most requested domains from web server logs.

#### Second-to-Last Field: awk -F " " '{print $(NF-1)}'

**Command:** `awk -F " " '{print $(NF-1)}' test1`

**DevOps Use Cases:**

1.  **Extract HTTP Status Codes**

    ```bash
    curl -v https://example.com 2>&1 | grep "< HTTP" | awk '{print $(NF-1)}'
    ```

    Extract HTTP status from curl verbose output for health checks.

#### Multiple Field Output: awk -F " " '{print $1,$3}'

**Command:** `awk -F " " '{print $1,$3}' test1`

**DevOps Use Cases:**

1.  **Extract Critical Log Data**

    ```bash
    cat system.log | awk -F " " '{print $1,$3}' | grep "ERROR"
    ```

    Extract timestamp and error code for incident tracking.
2.  **Kubernetes Event Parsing**

    ```bash
    kubectl get events | awk -F " " '{print $1,$3}'
    ```

    Extract namespace and event reason for cluster monitoring.

***

### Part 5: Process Management

#### List All Processes: ps -ef

**Command:** `ps -ef`

**DevOps Use Cases:**

1.  **Container Health Check Script**

    ```bash
    ps -ef | grep -c "app_process" > /tmp/process_count
    if [ $(cat /tmp/process_count) -eq 0 ]; then
      systemctl restart app_service
    fi
    ```

    Monitor critical application processes and restart if they crash.

#### Find Specific Process: ps -ef|grep -i "process\_name"

**Command:** `ps -ef|grep -i "process_name/service_name"`

**DevOps Use Cases:**

1.  **CI/CD Pipeline Dependency Check**

    ```bash
    ps -ef | grep -i "docker" && echo "Docker running" || echo "Docker not running"
    ```

    Verify Docker daemon is running before starting CI/CD jobs.
2.  **Load Balancer Health Verification**

    ```bash
    ps -ef | grep -i "nginx" | grep -v grep
    ```

    Check if Nginx is running for load balancer verification in health checks.

#### Multiple Process Grep: ps -ef|grep -ie "process1" -ie "process2"

**Command:** `ps -ef|grep -ie "process1" -ie "process2"`

**DevOps Use Cases:**

1.  **Multi-Service Monitoring**

    ```bash
    ps -ef | grep -ie "java" -ie "node" -ie "python"
    ```

    Check if all required application services are running in a microservices environment.

#### Kill Process: kill -9 "pid"

**Command:** `kill -9 "pid"`

**DevOps Use Cases:**

1.  **Force Kill Hung Deployment Process**

    ```bash
    old_pid=$(ps aux | grep "deployment_script" | grep -v grep | awk '{print $2}')
    kill -9 $old_pid
    ```

    Force terminate a stuck deployment to allow new deployment to proceed.
2.  **Cleanup on Container Exit**

    ```bash
    docker stop container_name
    docker ps -aq | xargs docker rm
    ```

    Clean up zombie processes after container termination.

#### Service Control Commands

**Commands:**

* `service service_name stop`
* `service service_name start`
* `service service_name restart`

**DevOps Use Cases:**

1.  **Blue-Green Deployment**

    ```bash
    service app_service stop
    deploy_new_version
    service app_service start
    ```

    Stop old application, deploy new version, and start service.
2.  **Zero-Downtime Database Migration**

    ```bash
    service mysql stop
    perform_migration
    service mysql start
    ```

    Coordinate service downtime with migration windows.

#### Process by User: ps -u "user\_name"

**Commands:** `ps -u "user_name"` | `ps -u "ubuntu"`

**DevOps Use Cases:**

1.  **Identify Resource-Heavy Users**

    ```bash
    ps -u ubuntu --sort=-%mem | head -5
    ```

    Find top 5 memory-consuming processes for a specific user in performance troubleshooting.
2.  **Container User Process Tracking**

    ```bash
    ps -u appuser | wc -l
    ```

    Count total processes running under application user for capacity monitoring.

#### Add New User: adduser username

**Command:** `adduser username`

**DevOps Use Cases:**

1.  **Automation User Creation**

    ```bash
    adduser cicd_user
    usermod -aG docker cicd_user
    usermod -aG wheel cicd_user
    ```

    Create dedicated user for CI/CD pipeline with required group memberships.
2.  **Multi-Tenant Isolation**

    ```bash
    adduser tenant1_user
    adduser tenant2_user
    ```

    Create isolated users for different customers in SaaS platforms.

***

### Part 6: File Search - find

#### Basic File/Directory Search: find . -iname "test"

**Commands:**

* `find . -iname "test"` (case-insensitive)
* `find . -iname "dir"`
* `find /Users/hareeshab/Documents -iname "dir2"`

**DevOps Use Cases:**

1.  **Locate Configuration Files**

    ```bash
    find /etc -iname "app.conf"
    ```

    Find configuration files for backup and version control in multi-server environments.
2.  **Docker Volume Cleanup**

    ```bash
    find /var/lib/docker/volumes -iname "unused*"
    ```

    Find and audit unused Docker volumes for cleanup.

#### Find by Type - Files: find . -type f -iname "test"

**Command:** `find . -type f -iname "test"`

**DevOps Use Cases:**

1.  **Locate Application Logs**

    ```bash
    find /var/log -type f -iname "*error*" -mtime -1
    ```

    Find error logs modified in last 24 hours for incident investigation.

#### Find by Type - Directories: find . -type d -iname "test"

**Command:** `find . -type d -iname "test"`

**DevOps Use Cases:**

1.  **Kubernetes Secret Volume Mounts**

    ```bash
    find / -type d -iname "secrets" 2>/dev/null
    ```

    Locate secret directories in container filesystem for security audits.

#### Find by Modification Time (Days): find . -type f -mtime -10

**Commands:**

* `find . -type f -mtime -10` (modified within 10 days)
* `find . -type d -mtime -10`
* `find . -type d -mtime +5` (modified more than 5 days ago)
* `find . -type f -mtime +20`

**DevOps Use Cases:**

1.  **Log Rotation Automation**

    ```bash
    find /var/log/app -type f -mtime +7 -exec gzip {} \;
    ```

    Find and compress logs older than 7 days for storage optimization.
2.  **Backup Verification**

    ```bash
    find /backup -type f -mtime -1
    ```

    Verify that daily backups were created in the last 24 hours.

#### Find by Modification Time (Minutes): find . -type f -mmin -5

**Commands:**

* `find . -type f -mmin -5` (modified within 5 minutes)
* `find . -type d -mmin -15`

**DevOps Use Cases:**

1.  **Real-Time File Change Detection**

    ```bash
    find /opt/app/config -type f -mmin -1
    ```

    Detect config file changes in the last minute for configuration reload triggers.
2.  **CI/CD Artifact Verification**

    ```bash
    find /target -type f -mmin -5
    ```

    Verify build artifacts were recently created in the last 5 minutes.

#### Find by Permissions: find . -type f -perm 777

**Commands:**

* `find . -type f -perm 777` (world-readable, writable, executable files)
* `find . -type d -perm 644`
* `find . -perm 755`

**DevOps Use Cases:**

1.  **Security Audit - World-Writable Files**

    ```bash
    find / -type f -perm 777 2>/dev/null
    ```

    Identify world-writable files that could pose security risks in production.
2.  **Kubernetes Secret File Permissions**

    ```bash
    find /var/run/secrets -type f -perm 777
    ```

    Verify secret files have restrictive permissions (not world-readable).

#### Find Empty Files: find . -type f -empty

**Command:** `find . -type f -empty`

**DevOps Use Cases:**

1.  **Cleanup Incomplete Artifacts**

    ```bash
    find /var/cache -type f -empty -exec rm {} \;
    ```

    Remove empty build cache files to free up disk space.

#### Find Non-Empty Files: find . -type f -not -empty

**Command:** `find . -type f -not -empty`

**DevOps Use Cases:**

1.  **Verify Non-Empty Logs**

    ```bash
    find /var/log -type f -not -empty -name "*error*"
    ```

    Ensure error log files have content before archiving.

***

### Part 7: Bulk Operations - xargs

#### Delete Old Files: find . -type f -mtime +2|xargs rm

**Commands:**

* `find . -type f -mtime +2|xargs rm` (delete files older than 2 days)
* `find . -type f -mtime +365|xargs rm`

**DevOps Use Cases:**

1.  **Automated Disk Cleanup**

    ```bash
    find /tmp -type f -mtime +7 | xargs rm -f
    ```

    Delete temporary files older than 7 days to prevent disk space issues.
2.  **Archive Old Build Artifacts**

    ```bash
    find /var/lib/jenkins/workspace -type f -mtime +30 | xargs tar -czf archive.tar.gz
    ```

    Archive old Jenkins build artifacts for long-term storage.

#### Find with Depth Limit: find . -iname "test" -maxdepth 1

**Commands:**

* `find . -type f -iname "test" -maxdepth 1`
* `find . -type d -iname "test" -maxdepth 2`
* `find . -iname "test" -maxdepth 1`

**DevOps Use Cases:**

1.  **Shallow Directory Search**

    ```bash
    find / -maxdepth 2 -iname "docker" -type d
    ```

    Limit deep filesystem searches to improve performance in large filesystems.
2.  **Configuration File Location**

    ```bash
    find /etc -maxdepth 1 -iname "app*" -type f
    ```

    Search only top-level /etc directory for application config files.

***

### Part 8: File Linking

#### Soft Link (Symbolic Link): ln -s test1 softlink

**Commands:**

* `ln -s test1 softlink`
* `ln -s /home/test/test1 softlink`

**DevOps Use Cases:**

1.  **Application Version Management**

    ```bash
    ln -s /opt/app/app-v2.0 /opt/app/current
    ```

    Symlink allows zero-downtime application upgrades by switching the link.
2.  **Kubernetes ConfigMap Mounting**

    ```bash
    ln -s /etc/config/production.conf /opt/app/config.conf
    ```

    Create environment-specific symlinks for flexible configuration management.

#### Hard Link: ln /home/test/test1 hardlink

**Command:** `ln /home/test/test1 hardlink`

**DevOps Use Cases:**

1.  **Disaster Recovery Backup**

    ```bash
    ln /var/lib/mysql/data.db /backup/data.db.hardlink
    ```

    Hard links create instant backups without additional disk space for frequently accessed files.

***

### Part 9: System Information

#### System Info: uname -a

**Command:** `uname -a`

**DevOps Use Cases:**

1.  **OS Compatibility Check in Terraform**

    ```bash
    KERNEL=$(uname -a | awk '{print $1}')
    if [ "$KERNEL" = "Linux" ]; then
      terraform apply -var="os_type=linux"
    fi
    ```

    Detect OS for platform-specific infrastructure provisioning.

#### OS Release Info: cat /etc/os-release

**Command:** `cat /etc/os-release`

**DevOps Use Cases:**

1.  **Container Base Image Verification**

    ```bash
    docker run ubuntu:20.04 cat /etc/os-release
    ```

    Verify correct OS version in container image for compliance.

#### Hostname: hostname

**Command:** `hostname`

**DevOps Use Cases:**

1.  **Terraform Naming Convention**

    ```bash
    HOSTNAME=$(hostname)
    echo "Deploying to: $HOSTNAME"
    ```

    Use hostname for dynamic configuration and logging in multi-node deployments.

#### Logged-in Users Count: who|wc -l

**Command:** `who|wc -l`

**DevOps Use Cases:**

1.  **Session Monitoring for Security**

    ```bash
    who | wc -l > /var/lib/monitoring/active_sessions
    ```

    Track concurrent user sessions for security compliance.

#### Current User: whoami

**Command:** `whoami`

**DevOps Use Cases:**

1.  **Permission Verification in Scripts**

    ```bash
    if [ "$(whoami)" != "root" ]; then
      echo "This script requires root privileges"
      exit 1
    fi
    ```

    Ensure script runs with correct user privileges before performing privileged operations.

***

### Part 10: Background Process Management

#### No-Hangup Process: nohup ./script.sh &

**Command:** `nohup ./script.sh &`

**DevOps Use Cases:**

1.  **Long-Running Deployment Script**

    ```bash
    nohup ./deploy_microservices.sh > deployment.log 2>&1 &
    ```

    Run deployment that survives SSH session disconnect.
2.  **Background Health Check**

    ```bash
    nohup ./health_check_loop.sh &
    ```

    Continuous health checking that persists even after terminal logout.

***

### Part 11: System Monitoring

#### Real-Time Process Monitoring: top

**Command:** `top`

**DevOps Use Cases:**

1.  **Container Resource Monitoring**

    ```bash
    docker top container_name
    ```

    Monitor process resource usage inside running containers.
2.  **Memory Leak Detection**

    ```bash
    top -bn1 | grep "app_process" | awk '{print $10}'
    ```

    Track memory consumption of application processes in automated monitoring script.

***

### Part 12: Remote Access and File Transfer

#### Secure Shell Access: ssh

**Commands:**

* `ssh username@server`
* `ssh -i demo.pem user@server`
* `ssh -i demo.pem ubuntu@54.26.118.12`

**DevOps Use Cases:**

1.  **Bastion Host Jump through SSH**

    ```bash
    ssh -J bastion_user@bastion_ip ubuntu@private_server_ip
    ```

    Access private servers through jump host in restricted networks.
2.  **Execute Remote Commands in Terraform**

    ```bash
    provisioner "remote-exec" {
      inline = [
        "sudo systemctl restart app-service"
      ]
      connection {
        type        = "ssh"
        user        = "ubuntu"
        private_key = file("~/.ssh/id_rsa")
        host        = self.public_ip
      }
    }
    ```

#### Secure Copy Protocol: scp

**Commands:**

* `scp file_name user@server:/tmp`
* `scp file_name user@server:/home/ubuntu`
* `scp -r directory user@server:/path`

**DevOps Use Cases:**

1.  **Deploy Application Binary**

    ```bash
    scp -i deploy.pem ./app-binary ubuntu@10.0.1.50:/opt/app/
    ```

    Transfer compiled application to production servers.
2.  **Batch Configuration Distribution**

    ```bash
    for server in server1 server2 server3; do
      scp config.yaml ubuntu@$server:/etc/app/
    done
    ```

    Distribute configuration files to multiple servers.

#### Rsync for Efficient Transfer: rsync

**Command:** `rsync file user@server:/home/temp`

**DevOps Use Cases:**

1.  **Incremental Backup**

    ```bash
    rsync -avz --delete /data/ backup@backup_server:/backups/prod_data/
    ```

    Efficiently sync only changed files to backup server, removing deleted files.
2.  **Database Replication**

    ```bash
    rsync -avz --exclude='*.log' /var/lib/mysql/ replica@replica_server:/var/lib/mysql/
    ```

    Sync database files to replica excluding logs.

***

### Part 13: Network Diagnostics

#### Test Connectivity: ping

**Command:** `ping ip_address or ping hostname or ping google.com`

**DevOps Use Cases:**

1.  **Kubernetes Service Discovery Health Check**

    ```bash
    ping -c 1 kubernetes.default.svc.cluster.local
    ```

    Verify DNS resolution in Kubernetes cluster for service discovery.
2.  **Network Availability Script**

    ```bash
    for ip in 10.0.1.1 10.0.1.2 10.0.1.3; do
      ping -c 1 $ip || echo "$ip is unreachable"
    done
    ```

    Check reachability of all servers in a cluster.

#### Telnet Connection Test: telnet

**Command:** `telnet ip or telnet dns_name [port 23]`

**DevOps Use Cases:**

1.  **Database Port Connectivity Check**

    ```bash
    echo "quit" | telnet database.example.com 3306
    ```

    Verify database server is listening on correct port.
2.  **Port Availability Verification**

    ```bash
    telnet load_balancer 443
    ```

    Test SSL/TLS port connectivity before traffic routing.

#### Network Port Monitoring: netstat -na|grep 8080

**Command:** `netstat -na|grep 8080`

**DevOps Use Cases:**

1.  **Service Port Binding Verification**

    ```bash
    netstat -tlnp | grep 8080
    ```

    Verify application is listening on expected port with process PID.
2.  **Connection State Analysis**

    ```bash
    netstat -an | grep ESTABLISHED | wc -l
    ```

    Count active connections to load balancer for traffic analysis.
3.  **Port Conflict Detection**

    ```bash
    if netstat -tln | grep ":8080 "; then
      echo "Port 8080 already in use, deploying to 8081"
    fi
    ```

    Detect port conflicts before starting new services in deployment automation.

***

### Real-World DevOps Pipeline Example

Here's how these commands work together in a complete CI/CD deployment workflow:

```bash
#!/bin/bash
set -e

# 1. Find and verify old deployment files
echo "Cleaning old artifacts..."
find /opt/app -type f -mtime +5 | xargs rm -f

# 2. Extract deployment version from build
VERSION=$(cat build.log | sed -n '10p' | grep -o "v[0-9.]*")

# 3. Copy new application
scp -i deploy.pem ./app-${VERSION} ubuntu@prod-server:/opt/app/

# 4. Update configuration files with new version
sed -i "s/APP_VERSION=.*/APP_VERSION=${VERSION}/g" /opt/app/config.yaml

# 5. Set correct ownership and permissions
ssh -i deploy.pem ubuntu@prod-server "chown app:appgroup /opt/app/app-${VERSION} && chmod 755 /opt/app/app-${VERSION}"

# 6. Kill old process and start new one
ssh -i deploy.pem ubuntu@prod-server "pkill -f 'java -jar' || true && nohup java -jar /opt/app/app-${VERSION} > /tmp/app.log 2>&1 &"

# 7. Verify service is running
ssh -i deploy.pem ubuntu@prod-server "sleep 5 && ps aux | grep -i 'java.*app-${VERSION}' && netstat -tlnp | grep 8080"

echo "Deployment completed successfully for version ${VERSION}"
```

This example demonstrates:

* **find** with **xargs** for cleanup
* **sed** for configuration management
* **scp** for remote file transfer
* **ssh** for remote command execution
* **chown** for permission management
* **ps** and **netstat** for verification
