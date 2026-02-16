# multi stage docker file

### What are Docker Multi-Stage Builds?

Docker multi-stage builds are a powerful feature that allows you to use multiple `FROM` statements in a single Dockerfile. Each `FROM` instruction begins a new stage in the build process, enabling you to create optimized, production-ready images by separating build dependencies from runtime requirements.

Key Benefits:

* **Reduced Image Size**: Eliminate build tools and dependencies from final image (50-70% size reduction)
* **Enhanced Security**: Minimize attack surface by excluding unnecessary components
* **Improved Build Efficiency**: Leverage Docker's layer caching and parallel execution
* **Simplified Workflow**: Single Dockerfile instead of multiple files

### How Multi-Stage Builds Work

{% stepper %}
{% step %}
### Build stage

Uses full-featured base image with all build tools.
{% endstep %}

{% step %}
### Runtime stage

Uses minimal base image and copies only necessary artifacts.

Each stage can copy artifacts from previous stages using `COPY --from=<stage-name>`.
{% endstep %}
{% endstepper %}

### Real-Time Examples and Demos

## Multistage Dockerfile for Building and Running hello-world

This Dockerfile performs the following steps in distinct stages:

{% stepper %}
{% step %}
### Clone

Clone the Git repository.
{% endstep %}

{% step %}
### Build

Build the project using Maven.
{% endstep %}

{% step %}
### Package

Package the generated WAR into an Apache Tomcat base image.
{% endstep %}

{% step %}
### Expose & Run

Expose the HTTP port and run Tomcat.
{% endstep %}
{% endstepper %}

Save this as `Dockerfile` in an empty host folder and build with `docker build -t spm-hello-world .`.

{% code title="Dockerfile" %}
```dockerfile
# Use a base image that supports Java and Apache Tomcat
FROM maven:3.9.1-eclipse-temurin-17 AS builder

# Install git
RUN apt-get update \
    && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

# Set the working directory
WORKDIR /app   

# Clone the repository
RUN git clone https://github.com/nimbuswiztech/spm-hello-world.git

# Set working directory
WORKDIR /app/spm-hello-world

# Build the WAR
RUN mvn clean install -DskipTests

# Stage 2: Package into Tomcat
FROM tomcat:10.1-jdk17 AS final

# Remove the default Tomcat webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy the webapp.war file to the webapps directory of Tomcat
COPY --from=builder /app/spm-hello-world/webapp/target/webapp.war /usr/local/tomcat/webapps/ROOT.war

# Expose the default Tomcat port (8080)
EXPOSE 8080

# Start Tomcat when the container starts
CMD ["catalina.sh", "run"]
```
{% endcode %}

### Usage

{% stepper %}
{% step %}
### Build the image

```bash
docker build -t spm-hello-world .
```
{% endstep %}

{% step %}
### Run the container

```bash
docker run -d -p 8080:8080 --name hello-world-app spm-hello-world
```
{% endstep %}

{% step %}
### Access the application

Open your browser at: `http://localhost:8080/`
{% endstep %}
{% endstepper %}

***

This multistage Dockerfile keeps the final image minimal by discarding build tools and source code, copying only the generated `WAR` into Tomcat.
