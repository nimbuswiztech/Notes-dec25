# docker basics

{% stepper %}
{% step %}
### 1. Introduction to Docker

* **Concept:** Docker is a platform to develop, ship, and run applications in lightweight, portable containers.
* **Key Points:** Containers isolate applications; they share the host OS kernel but run as if on their own machine.
* **Difference vs VMs:** VMs virtualize hardware, containers virtualize OS. Containers are faster and use fewer resources.
* **Use cases:** Simplifying deployment, CI/CD, microservices.
{% endstep %}

{% step %}
### 2. Docker Fundamentals

* **Docker Architecture:**
  * Docker daemon runs on your machine handling container lifecycle.
  * Docker CLI is the command tool you interact with.
  * Images are the blueprint, containers are running instances.
* **Registries:** Docker Hub is a public registry to find and share images.
{% endstep %}

{% step %}
### 3. Setting Up Docker Environment

* **Installation:** Guide links or instructions to install Docker on Windows/Mac/Linux.
*   **First command:**

    <pre data-title="First command"><code>
    </code></pre>

```bash
docker run hello-world
```
{% endstep %}

{% step %}
* This shows Docker is installed correctly.
* **Container management:**
  * Start: `docker start <container>`
  * Stop: `docker stop <container>`
  * Remove: `docker rm <container>`
{% endstep %}
{% endstepper %}

### 4. Working with Docker Images

* **Images & Layers:** Images are made of layers stacked; each command in a Dockerfile adds a layer.
* **Pull from Docker Hub:**

{% code title="Pull image" %}
```bash
docker pull <image>
```
{% endcode %}

* **Build your image:**
  * Dockerfile example:

{% code title="Dockerfile" %}
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
CMD ["nginx", "-g", "daemon off;"]
```
{% endcode %}

* Build:

{% code title="Build image" %}
```bash
docker build -t my-nginx .
```
{% endcode %}

* **Push to Hub:**
  * Tag:

{% code title="Tag image" %}
```bash
docker tag my-nginx username/my-nginx:tag
```
{% endcode %}

* Push:

{% code title="Push image" %}
```bash
docker push username/my-nginx:tag
```
{% endcode %}

### 5. Docker Containers Deep Dive

* **Managing containers:**
  * List running:

{% code title="List running containers" %}
```bash
docker ps
```
{% endcode %}

* Execute command:

{% code title="Exec into container" %}
```bash
docker exec -it <container> bash
```
{% endcode %}

* View logs:

{% code title="View logs" %}
```bash
docker logs <container>
```
{% endcode %}

* **Volumes for data:**
  * Create volume:

{% code title="Create volume" %}
```bash
docker volume create myvolume
```
{% endcode %}

* Run container with volume:

{% code title="Run with volume" %}
```bash
docker run -v myvolume:/data myimage
```
{% endcode %}

* **Networking:** Expose ports with `-p hostPort:containerPort`.

### 6. Docker Compose

* Compose file example:

{% code title="docker-compose.yml" %}
```yaml
version: "3"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```
{% endcode %}

* **Commands:**
  * `docker-compose up` to start
  * `docker-compose down` to stop

### 7. Advanced Docker Concepts

* **Custom networks:** Create bridge networks with:

{% code title="Create network" %}
```bash
docker network create mynetwork
```
{% endcode %}

* **Data management:** Use volumes and bind mounts for persistent data.
* **Security:** Run as non-root user, scan images for vulnerabilities.
* **Orchestration overview:** Brief mention of Docker Swarm and Kubernetes.

### 8. Hands-on Projects

* Guide to build a multi-container app with web and database using Docker Compose.
* Instructions to optimize images by minimizing layer count and clearing caches.

### 9. Troubleshooting & Best Practices

* Common errors: Port conflicts, permission issues, out of disk space.
* Cleanup:

{% code title="Cleanup" %}
```bash
docker system prune
```
{% endcode %}

### 10. Review & Q\&A

* Recap commands, concepts.
* Encourage questions and pointers to official Docker docs and tutorials.

***

### docker run — overview

The docker run command is used to create and start a new container from a specified Docker image. Its basic syntax is:

{% code title="docker run syntax" %}
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
{% endcode %}

At minimum, you must specify an image; options and command arguments allow customization.

#### How the docker run Command Works

* Checks if the image exists locally, pulls if not.
* Creates a container, assigns resources and namespaces.
* Starts the container and executes the specified command (or the image default).
* Optionally attaches your terminal if using interactive mode.

More options can be listed by running:

{% code title="Help" %}
```bash
docker run --help
```
{% endcode %}

This command is foundational for working with Docker containers in all application scenarios.

***

### Common Examples

* Start a container with Ubuntu interactively:

{% code title="Interactive Ubuntu" %}
```bash
docker run -it ubuntu:latest bash
```
{% endcode %}

* Run container in the background (detached mode):

{% code title="Detached nginx" %}
```bash
docker run -d nginx
```
{% endcode %}

* Assign a name to the container:

{% code title="Name container" %}
```bash
docker run --name mynginx nginx
```
{% endcode %}

* Expose container ports to host:

{% code title="Port mapping" %}
```bash
docker run -p 8080:80 nginx
```
{% endcode %}

* Remove container after exit:

{% code title="Auto-remove" %}
```bash
docker run --rm ubuntu echo "Hello World"
```
{% endcode %}

#### Important Options

* `-it`: Interactive and allocates a terminal.
* `-d`: Detached/background mode.
* `--name`: Set container name.
* `-p`: Port mapping.
* `-v`: Mount volumes.
* `--rm`: Auto-remove container after exit.
* `-e`: Set environment variables.
* `--entrypoint`: Override the default entrypoint.
* `--network`: Attach to specified network.

***

### Expose Nginx to your local machine

To expose Nginx running in a Docker container to your local machine, map a host port to container port 80. Example:

{% code title="Run nginx (example)" %}
```bash
docker run -d --name nginx-server -p 8080:80 nginx
```
{% endcode %}

* `-d` runs Nginx in detached mode.
* `--name nginx-server` gives your container a custom name.
* `-p 8080:80` maps host port 8080 to container port 80, exposing Nginx at http://localhost:8080.

To use port 80 on your host, replace 8080 with 80. Run multiple containers by mapping each to a different host port:

{% code title="Multiple nginx instances" %}
```bash
docker run -d --name nginx-alt -p 8081:80 nginx
```
{% endcode %}
