# learn docker beginner to expert

**What is Docker?**

Docker is a set of platform as a service products that use OS-level virtualisation to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. All containers are run by a single [operating-system kernel](https://en.wikipedia.org/wiki/Kernel_\(operating_system\)) and are thus more lightweight than [virtual machines](https://en.wikipedia.org/wiki/Virtual_machine).[\[8\]](https://en.wikipedia.org/wiki/Docker_\(software\)#cite_note-what-is-a-container-9)

_What is a Container?_

Docker Container is a standardised unit which can be created on the fly to deploy a particular application or environment. It could be an Ubuntu container, CentOs container, etc. to fulfill the requirement from an operating system point of view. Also, it could be an application oriented container like CakePHP container or a Tomcat-Ubuntu container etc.

### Docker concepts

Docker is a platform for developers and sysadmins to **build, share, and run** applications with containers. The use of containers to deploy applications is called _containerisation_. Containers are not new, but their use for easily deploying applications is.

Containerisation is increasingly popular because containers are:

* Flexible: Even the most complex applications can be containerised.
* Lightweight: Containers leverage and share the host kernel, making them much more efficient in terms of system resources than virtual machines.
* Portable: You can build locally, deploy to the cloud, and run anywhere.
* Loosely coupled: Containers are highly self sufficient and encapsulated, allowing you to replace or upgrade one without disrupting others.
* Scalable: You can increase and automatically distribute container replicas across a datacenter.
* Secure: Containers apply aggressive constraints and isolations to processes without any configuration required on the part of the user.

Welcome! The Docker Get Started Tutorial teaches you how to:

* Set up your Docker environment (on this page)
* [Build an image and run it as one container](https://docs.docker.com/get-started/part2/)
* [Set up and use a Kubernetes environment on your development machine](https://docs.docker.com/get-started/part3/)
* [Set up and use a Swarm environment on your development machine](https://docs.docker.com/get-started/part4/)
* [Share your containerized applications on Docker Hub](https://docs.docker.com/get-started/part5/)

### Images and containers

Fundamentally, a container is nothing but a running process, with some added encapsulation features applied to it in order to keep it isolated from the host and from other containers. One of the most important aspects of container isolation is that each container interacts with its own, private filesystem; this filesystem is provided by a Docker **image**. An image includes everything needed to run an application — the code or binary, runtimes, dependencies, and any other filesystem objects required.

### Containers and virtual machines

A container runs _natively_ on Linux and shares the kernel of the host machine with other containers. It runs a discrete process, taking no more memory than any other executable, making it lightweight.

By contrast, a **virtual machine** (VM) runs a full-blown “guest” operating system with _virtual_ access to host resources through a hypervisor. In general, VMs incur a lot of overhead beyond what is being consumed by your application logic.

Summary of key points so far:

* Virtual Machines are slow and take a lot of time to boot.
* Containers are fast and boot quickly as they use the host operating system and share relevant libraries.
* Containers do not waste or block host resources unlike virtual machines.
* Containers have isolated libraries and binaries specific to the application they are running.
* Containers are handled by a containerisation engine.
* Docker is one of the containerisation platforms which can be used to create and run containers.

Why do we use Docker?

Docker containers are lightweight and easy to create and deploy. Docker provides containers that package an entire runtime environment — the application, all its dependencies, libraries, binaries and configuration files — into a single bundle. Each application runs separately from the others. Docker solves dependency problems by keeping dependencies contained inside the containers.

Benefits of using Containers over Virtual Machines

* Unlike VMs that run a Guest OS via a hypervisor, Docker containers run directly on a host server (for Linux) using a Docker engine, making them faster and more lightweight.
* Docker containers can be more easily integrated compared to VMs.
* Fully virtualized systems (VMs) provide more isolation but require more resources. Docker offers less isolation but much lower resource requirements, allowing thousands of containers on a host.
* A VM can take a minute or more to start, while a Docker container usually starts in a fraction of a second.
* Containers are easier to break out of than a VM.
* No need to preallocate RAM for containers; they utilize only the amount required.

How does Docker work?

Docker has a **Docker Engine**, a client-server application with three main components:

* A server: a long-running daemon process (Docker daemon).
* A client: Docker CLI (Command Line Interface).
* A REST API: used to communicate between the Docker CLI and the Docker daemon.

The Docker daemon receives commands from the client and manages Docker objects such as images, containers, networks, and volumes. The Docker client and daemon can run on the same system, or a Docker client can connect to a remote Docker daemon. They communicate using a REST API over UNIX sockets or a network interface.

* On Linux, the Docker host runs the Docker daemon and the Docker client is accessed from the terminal.
* On Windows/macOS, Docker Desktop provides the environment. (Older setups used Docker Toolbox with client, Compose, Kitematic, Machine, and VirtualBox.)

Technology used in Docker

Docker is written in the Go programming language. Docker leverages Linux kernel features:

* namespaces: Provide isolated workspace called containers. When a container runs, Docker creates a set of namespaces for it, limiting access to resources within those namespaces.
* cgroups (control groups): Limit and isolate resource usage (CPU, memory, Disk I/O, network) for groups of processes. cgroups allow Docker to share available hardware resources and optionally enforce limits.
* UnionFS (Union file systems): File systems that create layers, making them lightweight and fast. Docker uses UnionFS to provide layered filesystem building blocks for containers.

Docker Engine combines namespaces, cgroups, and UnionFS into a wrapper called a **container format**. The default container format is `libcontainer`.

### Install Docker Desktop

The best way to get started developing containerized applications is with Docker Desktop (macOS or Windows). Docker Desktop makes it easy to set up Kubernetes or Swarm locally. Follow the installation instructions for your OS:

* [OSX](https://docs.docker.com/docker-for-mac/install/)
* [Windows](https://docs.docker.com/docker-for-windows/install/)

{% stepper %}
{% step %}
### Common Docker commands

This section covers daily-use Docker commands and their basic usage.

#### docker --version

This command shows the currently installed Docker version.

#### docker pull

Usage:

{% code title="Pull an image" %}
```
```
{% endcode %}

```bash
docker pull <image name>
```

Pulls images from the Docker repository (hub.docker.com).

#### docker run

Usage:

{% code title="Run a container" %}
```
```
{% endcode %}

```bash
docker run -it -d <image name>
```

Creates a container from an image.
{% endstep %}

{% step %}
### Inspecting and accessing containers

#### docker ps

Lists running containers.

#### docker ps -a

Shows all running and exited containers.

#### docker exec

Usage:

{% code title="Exec into container" %}
```
```
{% endcode %}

```bash
docker exec -it <container id> bash
```

Accesses a running container's shell.
{% endstep %}

{% step %}
### Stopping and killing containers

#### docker stop

Usage:

{% code title="Stop container gracefully" %}
```
```
{% endcode %}

```bash
docker stop <container id>
```

Stops a running container (allows graceful shutdown).

#### docker kill

Usage:

{% code title="Kill container immediately" %}
```
```
{% endcode %}

```bash
docker kill <container id>
```

Kills a container immediately (no graceful shutdown).
{% endstep %}

{% step %}
### Images and repositories

#### docker commit

Usage:

{% code title="Create image from container" %}
```
```
{% endcode %}

```bash
docker commit <container id> <username/imagename>
```

Creates a new image from a modified container on the local system.

#### docker login

Logs into Docker Hub.

#### docker push

Usage:

{% code title="Push image to Docker Hub" %}
```
```
{% endcode %}

```bash
docker push <username/image name>
```

Pushes an image to Docker Hub.

#### docker images

Lists all locally stored Docker images.

#### docker rmi

Usage:

{% code title="Remove image" %}
```
```
{% endcode %}

```bash
docker rmi <image-id>
```

Deletes an image from local storage.
{% endstep %}

{% step %}
### Removing containers and building images

#### docker rm

Usage:

{% code title="Remove stopped container" %}
```
```
{% endcode %}

```bash
docker rm <container id>
```

Deletes a stopped container.

#### docker build

Usage:

{% code title="Build image from Dockerfile" %}
```
```
{% endcode %}

```bash
docker build <path to docker file>
```

Builds an image from a specified Dockerfile.
{% endstep %}
{% endstepper %}

### Creating Our First Docker Application

Suppose we have a PHP application and want to deploy it to staging or production. Ensure a Docker configuration is included at the application root.

{% stepper %}
{% step %}
### Create a Dockerfile in your application

Create a file named `Dockerfile` at the root of your application containing:

{% code title="Dockerfile" %}
```
```
{% endcode %}

```dockerfile
FROM PHP:7.2-Apache
COPY src/ /var/www/html/
EXPOSE 80
```

This configures PHP 7.2 with Apache, copies the app from `src/` to `/var/www/html/`, and exposes port 80.
{% endstep %}

{% step %}
### Installing Docker on Staging or Production Server

* macOS: https://docs.docker.com/docker-for-mac/install/
* Windows: https://docs.docker.com/docker-for-windows/install/
{% endstep %}

{% step %}
### Running Docker

After installation, start Docker (Docker Desktop / service). On GUI installs, this typically involves launching the Docker application.
{% endstep %}

{% step %}
### Deploying Your Application

1. Navigate to the project directory in a terminal and create a Docker image:

{% code title="Build image" %}
```
```
{% endcode %}

```bash
docker build -t <name-to-give-your-image> .
```

2. Run a container from that image and map host port 9090 to container port 80:

{% code title="Run container" %}
```
```
{% endcode %}

```bash
docker run -p 9090:80 <name-to-give-your-container>
```

* `9090` is the port to access the application on the host.
* `80` is the port exposed by the container.
{% endstep %}
{% endstepper %}

### Additional useful Docker commands

Stopping a running container:

{% code title="Stop container" %}
```bash
docker stop <id-of-container>
```
{% endcode %}

Starting a stopped container:

{% code title="Start container" %}
```bash
docker start <id-of-container>
```
{% endcode %}

Removing an image:

{% code title="Remove image" %}
```bash
docker rmi <id-of-image>
```
{% endcode %}

Removing a container:

{% code title="Remove container" %}
```bash
docker rm <id-of-container>
```
{% endcode %}
