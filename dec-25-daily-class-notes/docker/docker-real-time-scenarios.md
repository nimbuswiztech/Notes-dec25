# docker real time scenarios

## Docker Real-Time Scenarios: Comprehensive Teaching Guide

### Docker Images vs Containers: The Foundation

Core Concept

Understanding the difference between Docker images and containers is fundamental to mastering containerization. A Docker image serves as a read-only template containing application code, runtime, libraries, and dependencies, while a Docker container is a running instance of that image.

Real-World Analogy: Cookie Cutter vs Cookies

* Image = Cookie Cutter (template/blueprint)
* Container = Actual Cookie (running instance)
* You can make multiple cookies from one cutter
* Each cookie is independent
* The cutter remains unchanged regardless of how many cookies you make

***

### Real-Time Dockerfile Examples

Scenario 1: Multi-Stage Node.js E-commerce Application

{% code title="Dockerfile (Node.js multi-stage)" %}
```
```
{% endcode %}

```dockerfile
# Multi-stage Node.js Web Application Dockerfile
FROM node:18-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

Key Features:

* Multi-stage build for smaller production images
* Non-root user for enhanced security
* Health check for container monitoring
* Optimized dependency installation

Scenario 2: Java Spring Boot Banking API

{% code title="Dockerfile (Spring Boot)" %}
```
```
{% endcode %}

```dockerfile
FROM openjdk:17-jdk-slim AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN apt-get update && apt-get install -y maven
RUN mvn clean package -DskipTests

FROM openjdk:17-jre-slim AS production
RUN addgroup --system spring && adduser --system spring --ingroup spring
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN chown spring:spring app.jar
USER spring:spring
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

***

### Docker Compose for Microservices Architecture

Complete E-commerce Platform Example

{% code title="docker-compose.yml" %}
```
```
{% endcode %}

```yaml
version: '3.8'

services:
  # Database Service
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: securepassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  # Redis Cache
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # Backend API Service
  api:
    build: 
      context: ./backend
      dockerfile: Dockerfile
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://appuser:securepassword@postgres:5432/appdb
      REDIS_URL: redis://redis:6379
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

Key Features:

* Service dependencies with health checks
* Custom networks for secure service communication
* Persistent data storage with volumes
* Proper startup sequencing

***

### Debugging and Troubleshooting Scenarios

Essential Debugging Commands

Container Log Analysis

{% code title="Container logs and inspect" %}
```bash
# Check container logs
docker logs container_name
docker logs -f container_name  # Follow logs in real-time
docker logs --tail 50 container_name  # Last 50 lines

# Inspect container details
docker inspect container_name
docker inspect container_name | grep -i health
docker inspect container_name | grep -i status
```
{% endcode %}

Interactive Debugging

{% code title="Interactive debugging" %}
```bash
# Execute commands inside container
docker exec -it container_name /bin/bash
docker exec -it container_name /bin/sh
docker exec container_name ps aux  # Check running processes

# Check container resource usage
docker stats container_name
docker stats --no-stream  # Single snapshot
```
{% endcode %}

Network Troubleshooting

{% code title="Network troubleshooting" %}
```bash
# Debug networking issues
docker network ls
docker network inspect network_name
docker exec container_name nslookup another_container
docker exec container_name ping container_name
```
{% endcode %}

Health Check Implementation and Debugging

Basic Health Check Implementation

{% code title="HEALTHCHECK example" %}
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
{% endcode %}

Health Check Debugging Commands

{% code title="Health check debugging" %}
```bash
# Manual health check execution
docker exec container_name curl -f http://localhost:8080/health

# Check health status
docker ps  # Shows health status in the output
docker inspect container_name --format='{{.State.Health.Status}}'

# View detailed health check logs
docker inspect container_name --format='{{range .State.Health.Log}}{{.Output}}{{end}}'
```
{% endcode %}

Common Issues and Solutions

<details>

<summary>Issue: Container Exits Immediately</summary>

* Diagnosis: Container's main process terminates
* Solution: Ensure the main process runs continuously
* Debug command: docker logs container\_name

</details>

<details>

<summary>Issue: Port Binding Failures</summary>

* Diagnosis: Port already in use by another service
* Solution: Use different port or stop conflicting service
* Debug command: netstat -tulpn | grep :8080

</details>

<details>

<summary>Issue: Network Connectivity Issues</summary>

* Diagnosis: Containers cannot communicate
* Solution: Ensure containers are on the same network
* Debug command: docker exec app\_container ping db\_container

</details>

<details>

<summary>Issue: Resource Exhaustion</summary>

* Diagnosis: Out of disk space or memory
* Solution: Clean up unused resources and set resource limits

Debug commands:

{% code title="Resource cleanup" %}
```bash
docker system df
docker system prune -a
```
{% endcode %}

</details>

<details>

<summary>Issue: Permission Denied Errors</summary>

* Diagnosis: Incorrect file permissions or user context
* Solution: Fix ownership or run as correct user
* Debug command: docker exec container\_name ls -la /app

</details>

***

### Production Security Best Practices

Security Hardening Techniques

1. Use Non-Root Users

{% code title="Non-root user in Dockerfile" %}
```dockerfile
RUN adduser --disabled-password --gecos '' appuser
USER appuser
```
{% endcode %}

2. Implement Resource Limits

{% code title="Resource limits example" %}
```bash
docker run --memory=1g --cpus=2 my-app
```
{% endcode %}

3. Use Read-Only Filesystems

{% code title="Read-only filesystem example" %}
```bash
docker run --read-only --tmpfs /tmp my-app
```
{% endcode %}

4. Regular Security Scanning

* Scan base images for vulnerabilities
* Use trusted registries
* Implement Docker Content Trust

5. Secret Management

* Never hardcode secrets in Dockerfiles
* Use dedicated secret management tools
* Set proper file permissions

***

### Performance Optimization Strategies

Multi-Stage Builds

{% code title="Multi-stage build example" %}
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]
```
{% endcode %}

Container Monitoring

{% code title="Monitoring commands" %}
```bash
# Real-time resource monitoring
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Container health monitoring
docker inspect --format='{{.State.Health.Status}}' $(docker ps -q)
```
{% endcode %}

***

### Hands-On Demonstration Steps

{% stepper %}
{% step %}
### Images vs Containers Demo

```bash
# Demonstrate the concept
docker pull nginx:alpine
docker images
docker run -d --name demo1 -p 8080:80 nginx:alpine
docker run -d --name demo2 -p 8081:80 nginx:alpine
docker ps
# Show same image, different containers
```
{% endstep %}

{% step %}
### Real Application Build

* Create a simple Node.js application
* Write a production-ready Dockerfile
* Build and run the container
* Demonstrate debugging techniques
* Show health check implementation
{% endstep %}

{% step %}
### Microservices Demo

* Use Docker Compose to show service communication
* Demonstrate health checks and dependencies
* Show scaling and load balancing
* Implement service discovery
{% endstep %}

{% step %}
### Troubleshooting Session

* Intentionally introduce common issues
* Show systematic debugging process
* Fix issues step by step
* Demonstrate monitoring and logging
{% endstep %}
{% endstepper %}

***

This guide provides practical, real-time scenarios to help learners understand Docker concepts through examples covering image vs container fundamentals, production Dockerfiles, Compose-based microservices, debugging, security, and performance practices.
