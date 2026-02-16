# docker storage

### Types of Docker Storage

Comparison of Docker Storage Types: Volume Mounts vs Bind Mounts vs tmpfs Mounts

{% stepper %}
{% step %}
### Volume Mounts (Recommended for Production)

Volume mounts are **Docker-managed storage** stored in dedicated directories, typically `/var/lib/docker/volumes/` on Linux systems:

{% code title="Create and use a named volume" %}
```bash
# Create named volume
docker volume create my-app-data

# Mount volume to container
docker run -v my-app-data:/app/data my-image
```
{% endcode %}

**Best for**: Production databases, application data, essential caches
{% endstep %}

{% step %}
### Bind Mounts (Ideal for Development)

Bind mounts create **direct links** between host directories and containers:

{% code title="Bind mount examples" %}
```bash
# Mount current directory to container
docker run -v "$(pwd)":/app my-dev-image

# Mount specific host path
docker run -v /home/user/code:/app/src my-image
```
{% endcode %}

**Best for**: Development environments, configuration files, real-time code editing
{% endstep %}

{% step %}
### tmpfs Mounts (For Temporary Data)

tmpfs mounts store data in **host system memory (RAM)**, providing ultra-fast access but no persistence:

{% code title="tmpfs examples" %}
```bash
# Create tmpfs mount
docker run --tmpfs /tmp my-image
docker run --mount type=tmpfs,dst=/app/cache my-image
```
{% endcode %}

**Best for**: Temporary files, sensitive data, high-performance caching
{% endstep %}
{% endstepper %}

***

### Real-World Scenarios & Examples

#### Scenario: Web Development Environment

**Problem**: Developer needs to edit code without rebuilding Docker images constantly

**Solution**: Bind mount for live code updates

{% code title="Dockerfile (development)" %}
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
EXPOSE 3000
CMD ["npm", "run", "dev"]
```
{% endcode %}

{% code title="Run container with live reload" %}
```bash
# Development with live reload
docker run -d \
  --name dev-server \
  -p 3000:3000 \
  -v "$(pwd):/app" \
  -v /app/node_modules \
  my-dev-image

# Code changes reflect immediately without rebuilds
```
{% endcode %}

#### Scenario: MySQL Database with Persistent Storage

**Problem**: Database data must survive container updates and restarts

**Solution**: Named volume for database files

{% code title="MySQL persistent volume" %}
```bash
# Create persistent volume for MySQL
docker volume create mysql-data

# Run MySQL with persistent storage
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secure123 \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=userpass \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0
```
{% endcode %}

Testing persistence:

{% code title="Test persistence" %}
```bash
# Create test data
docker exec mysql-db mysql -u root -psecure123 \
  -e "INSERT INTO myapp.users (name) VALUES ('John Doe');"

# Stop and remove container
docker stop mysql-db && docker rm mysql-db

# Start new container with same volume
docker run -d --name mysql-new -v mysql-data:/var/lib/mysql mysql:8.0

# Data still exists!
```
{% endcode %}

#### Scenario: Multi-Container Application with Shared Storage

**Problem**: Multiple containers need access to shared configuration or content

**Solution**: Shared named volume

{% code title="docker-compose.yml (shared volume)" %}
```yaml
version: '3.8'
services:
  web1:
    image: nginx:alpine
    volumes:
      - shared-content:/usr/share/nginx/html
    ports:
      - "8081:80"

  web2:
    image: nginx:alpine
    volumes:
      - shared-content:/usr/share/nginx/html
    ports:
      - "8082:80"

  content-manager:
    image: alpine
    volumes:
      - shared-content:/data
    command: sh -c "while true; do echo 'Updated: $(date)' > /data/index.html; sleep 60; done"

volumes:
  shared-content:
```
{% endcode %}

#### Scenario: Log Aggregation System

**Problem**: Application logs need centralized collection and analysis

**Solution**: Volume for log persistence

{% code title="Log aggregation example" %}
```bash
# Create log volume
docker volume create app-logs

# Application container with log volume
docker run -d \
  --name my-app \
  -v app-logs:/var/log/app \
  my-application

# Log analysis container
docker run -d \
  --name log-analyzer \
  -v app-logs:/logs:ro \
  logstash:latest
```
{% endcode %}

***

### Complete Docker Compose Example

Here's a **production-ready multi-service application** demonstrating various volume types:

{% code title="docker-compose.yml - Complete Blog Application" %}
```yaml
# docker-compose.yml - Complete Blog Application
version: '3.8'

services:
  # Database with persistent storage
  database:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass123
    volumes:
      - db-data:/var/lib/mysql              # Named volume for data
      - ./mysql-config:/etc/mysql/conf.d:ro # Bind mount for config
    restart: unless-stopped

  # WordPress application
  wordpress:
    image: wordpress:latest
    depends_on:
      - database
    environment:
      WORDPRESS_DB_HOST: database:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass123
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-content:/var/www/html/wp-content # Named volume for uploads
      - ./wp-themes:/var/www/html/wp-content/themes # Bind mount for themes
    restart: unless-stopped

  # Reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro # Bind mount for config
      - wp-content:/var/www/html/wp-content:ro # Shared content volume
      - ssl-certs:/etc/nginx/ssl              # Named volume for certificates
    depends_on:
      - wordpress
    restart: unless-stopped

  # Redis for caching (tmpfs for session data)
  redis:
    image: redis:alpine
    tmpfs:
      - /tmp/redis-data  # tmpfs for temporary session storage
    command: redis-server --save 20 1 --loglevel warning
    restart: unless-stopped

volumes:
  db-data:      # MySQL persistent data
  wp-content:   # WordPress uploads and content
  ssl-certs:    # SSL certificates
```
{% endcode %}

***

### Debugging and Troubleshooting

#### Issue: Permission Denied Errors

**Symptoms**: Cannot write to mounted volume, "Permission denied" errors

Debug Steps:

{% code title="Check permissions and ownership" %}
```bash
# Check file ownership and permissions
ls -la /path/to/volume
docker exec -it container-name ls -la /mounted/path
docker exec -it container-name whoami
```
{% endcode %}

Solutions:

{% code title="Fix permissions and user mapping" %}
```bash
# Fix host directory permissions
sudo chown -R $USER:$USER /host/path
sudo chmod -R 755 /host/path

# Run container with proper user mapping
docker run --user $(id -u):$(id -g) -v volume:/path image

# Use init option for proper permission handling
docker run --init -v volume:/path image
```
{% endcode %}

#### Issue: Data Not Persisting

**Symptoms**: Data disappears after container restart

Debug Steps:

{% code title="Verify mounts and volumes" %}
```bash
# Verify volume is properly mounted
docker inspect container-name | grep -A 10 "Mounts"

# Check volume details
docker volume inspect volume-name

# List all volumes
docker volume ls
```
{% endcode %}

Solutions:

{% code title="Ensure correct mounts and data directories" %}
```bash
# Ensure correct volume mount syntax
docker run -v volume-name:/exact/data/path image

# For databases, use correct data directories:

# MySQL: /var/lib/mysql

# PostgreSQL: /var/lib/postgresql/data

# MongoDB: /data/db
```
{% endcode %}

#### Issue: Bind Mount Path Problems

**Symptoms**: Empty directories, files not visible

Debug Steps:

{% code title="Verify host paths and Docker Desktop sharing" %}
```bash
# Verify host path exists
ls -la /host/path

# Check Docker Desktop file sharing (Windows/Mac)

# Docker Desktop > Settings > Resources > File Sharing

# Use absolute paths
pwd  # Get current directory
```
{% endcode %}

Solutions:

{% code title="Use absolute paths and create host dirs" %}
```bash
# Always use absolute paths for bind mounts
docker run -v /full/path/to/host:/container/path image

# Create host directory if needed
mkdir -p /path/to/host/directory
```
{% endcode %}

#### Issue: Volume Space Issues

**Symptoms**: "No space left on device" errors

Debug Steps:

{% code title="Check disk usage and large volumes" %}
```bash
# Check volume space usage
docker system df -v

# Find large volumes
docker volume ls
du -sh /var/lib/docker/volumes/*
```
{% endcode %}

Solutions:

{% code title="Clean up unused volumes and system" %}
```bash
# Clean up unused volumes
docker volume prune

# Remove specific volume (ensure no containers use it)
docker volume rm volume-name

# Full system cleanup
docker system prune -a --volumes
```
{% endcode %}

***

### Essential Commands Reference

#### Volume Management

{% code title="Volume management commands" %}
```bash
# Create named volume
docker volume create my-volume

# List all volumes  
docker volume ls

# Inspect volume details
docker volume inspect my-volume

# Remove volume (must not be in use)
docker volume rm my-volume

# Remove all unused volumes
docker volume prune
```
{% endcode %}

#### Container Volume Operations

{% code title="Container volume operations" %}
```bash
# Named volume mount
docker run -v volume-name:/container/path image

# Bind mount (development)
docker run -v /host/path:/container/path image

# Read-only mount
docker run -v volume-name:/path:ro image

# tmpfs mount
docker run --tmpfs /tmp image
docker run --mount type=tmpfs,dst=/tmp image

# Multiple volumes
docker run -v vol1:/path1 -v vol2:/path2 image
```
{% endcode %}

#### Debugging Commands

{% code title="Debugging commands" %}
```bash
# Check container mounts
docker inspect container-name | grep -A 20 "Mounts"

# View container logs
docker logs container-name

# Access container filesystem
docker exec -it container-name /bin/bash

# Check volume contents (Linux)
sudo ls -la /var/lib/docker/volumes/volume-name/_data
```
{% endcode %}

***

### Best Practices

* Use Named Volumes for production data that must persist
* Use Bind Mounts for development and configuration files
* Use tmpfs for temporary, sensitive, or high-performance data
* Regular Backups of critical volumes using `docker cp` or volume backup tools
* Proper Permissions - match user IDs between host and container
* Clean Up unused volumes regularly with `docker volume prune`
* Document volume requirements in README and docker-compose files
* Security - never mount sensitive host directories unless necessary
