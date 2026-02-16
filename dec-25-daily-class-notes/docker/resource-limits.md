# resource limits

## Docker Container Resource Limits

When running Docker containers, limiting CPU and memory consumption is essential for maintaining system stability and ensuring fair resource distribution. Let me explain this concept through real-world examples that will make it easy for your students to understand.

### Why Resource Limits Matter

{% hint style="info" %}
Think of a Restaurant Kitchen Analogy:

Imagine you're managing a busy restaurant kitchen where multiple chefs (containers) work on different dishes (applications). Without proper rules, one chef might monopolize all the stoves and ingredients, leaving other chefs unable to complete their dishes. This would slow down the entire kitchen operation and upset customers.

Similarly, Docker containers running without resource limits can consume all available CPU and memory, causing other containers to starve for resources and potentially crashing the entire system.
{% endhint %}

### Real-World Scenarios Where Limits Are Critical

{% stepper %}
{% step %}
### University Web Server

A university hosts multiple student project websites on one server. Without limits, one student's poorly optimized application could consume all server resources, making other websites inaccessible.
{% endstep %}

{% step %}
### Development Environment

A team runs multiple microservices locally for development. One service with a memory leak could consume all available RAM, freezing the developer's computer.
{% endstep %}

{% step %}
### Production Environment

An e-commerce platform runs multiple services (user authentication, payment processing, inventory management). During a traffic spike, one service shouldn't monopolize resources and impact others.
{% endstep %}
{% endstepper %}

***

## Setting Memory Limits

Memory limits are arguably the most important to set because memory leaks can bring down entire systems.

### Basic Memory Limit Examples

```
# Limit container to 512MB of memory
docker run -m 512m nginx

# Limit to 2GB of memory
docker run --memory=2g nginx

# Prevent swap usage (recommended for production)
docker run -m 512m --memory-swap=512m nginx
```

### Memory Limit Demonstration

Here's a practical example for students:

```
# Run a web application with 256MB memory limit
docker run -d \
  --name memory-limited-app \
  --memory=256m \
  -p 8080:80 \
  nginx

# Monitor the container's resource usage
docker stats memory-limited-app
```

<details>

<summary>What happens when the limit is exceeded?</summary>

When a container hits its memory limit, the Linux Out-of-Memory (OOM) killer terminates the container. This prevents one container from crashing the entire host system.

</details>

***

## Setting CPU Limits

CPU limits control how much processing power a container can use. Unlike memory limits, CPU limits affect performance rather than causing crashes.

### CPU Limit Examples

```
# Limit container to 1.5 CPU cores
docker run --cpus=1.5 nginx

# Limit to 50% of one CPU core
docker run --cpus=0.5 nginx

# Set CPU priority (relative to other containers)
docker run --cpu-shares=512 nginx
```

### Understanding CPU Limit Types

* Hard Limits (`--cpus`): The container will never use more than the specified amount.
* Soft Limits (`--cpu-shares`): Only matters when there's CPU contention. If no other containers need CPU, the container can use more.

### Practical CPU Example

```
# Run a CPU-intensive application with limits
docker run -d \
  --name cpu-limited-app \
  --cpus=0.5 \
  --memory=512m \
  python:3.9 python -c "while True: pass"
```

***

## Combining CPU and Memory Limits

In real-world scenarios, you'll typically set both CPU and memory limits together:

```
# Web application with combined limits
docker run -d \
  --name webapp \
  --cpus=2 \
  --memory=1g \
  -p 8080:80 \
  nginx

# Development database with conservative limits
docker run -d \
  --name dev-database \
  --cpus=1 \
  --memory=512m \
  -e POSTGRES_PASSWORD=secret \
  postgres:13
```

***

## Using Docker Compose for Resource Management

For multi-container applications, use Docker Compose to define resource limits:

```
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '1'
          memory: 512M
  
  database:
    image: postgres:13
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
    environment:
      POSTGRES_PASSWORD: secret
```

***

## Monitoring Resource Usage

Teach students to monitor container resource consumption using the `docker stats` command:

```
# Monitor all running containers
docker stats

# Monitor specific container
docker stats webapp

# Get resource usage in JSON format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.MemPerc}}"
```

***

## Dynamic Resource Updates

You can modify resource limits on running containers:

```
bash# Update memory limit of running container
docker update --memory=1g webapp

# Update CPU limits
docker update --cpus=2 webapp
```

***

## Resource Reservations

Set minimum guaranteed resources using reservations in Docker Compose:

```
textdeploy:
  resources:
    limits:
      memory: 1G
    reservations:
      memory: 512M
```
