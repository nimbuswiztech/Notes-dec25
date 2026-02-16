# container lifecycle

_**Docker Container Lifecycle Management**_

Docker is a containerization platform that allows you to develop, ship, and run applications inside containers. We can run multiple containers on a single host at the same time. Containers are very fast and boot up quickly because, unlike virtual machines, they do not require the extra load of a hypervisor because they run directly within the host machine’s kernel. This document discusses _**Docker Container Lifecycle Management**_.

## Docker Container Lifecycle Management

The Docker Container Lifecycle describes the various stages that occur when we create a Docker container. Common states include:

* **Created**: A container that has been created but not started
* **Running**: A container running with all its processes
* **Paused**: A container whose processes have been paused
* **Stopped**: A container whose processes have been stopped
* **Deleted**: A container in a dead state

![Docker Container Lifecycle](../../.gitbook/assets/1p2T79jQpvRm1b06dv4tbzA.jpeg)

Docker Container Lifecycle Management is the process of managing the states of Docker containers. We must ensure that the containers are operational or that they are destroyed if they are no longer useful. Below are common commands for managing the Docker lifecycle.

The lifecycle of a docker container consists of five states:

{% stepper %}
{% step %}
### Created state

In the created state, a docker container is created from a docker image.

```bash
docker create --name <name-of-container> <docker-image-name>
```
{% endstep %}

{% step %}
### Running state

When the docker container is running, it begins executing the commands shown in the image. The `docker run` command is used to start a docker container.

```bash
docker run <container-id>
# or
docker run <container-name>
```

If no container is present, the `docker run` command creates one. In this case, the container creation step can be skipped.
{% endstep %}

{% step %}
### Paused state

The current command(s) in the docker container are paused when it is in the paused state. To pause a running container, use the `docker pause` command.

```bash
docker pause <container-id or container-name>
```

{% hint style="info" %}
`docker pause` pauses all the processes in the container by sending SIGSTOP to those processes.
{% endhint %}
{% endstep %}

{% step %}
### Unpaused state

When the container is unpaused, it resumes executing the commands that were paused. Docker sends SIGCONT to resume the processes.

```bash
docker unpause <container-id or container-name>
```
{% endstep %}

{% step %}
### Stopped state

The container’s main process is gracefully shut down in the stopped state. Docker sends SIGTERM for graceful shutdown and SIGKILL to kill the container’s main process if necessary. To stop a container, use the `docker stop` command.

```bash
docker stop <container-id or container-name>
```

Restarting a docker container is effectively a `docker stop` followed by a `docker run` (i.e., stop and run phases).
{% endstep %}

{% step %}
### Killed / Deleted state

The container’s main processes are terminated abruptly in the killed state. Docker sends a SIGKILL signal to the container’s main process to terminate it.

```bash
docker kill <container-id or container-name>
```
{% endstep %}
{% endstepper %}

## Difference between Docker create, Docker start and Docker run

* **Docker create** creates a new container based on the specified image, but does not start it immediately.
* **Docker start** is used to start or restart any previously stopped container. If you used `docker create` to make a container, you can start it with `docker start`.
* **Docker run** is a combination of `create` and `start`: it creates a new container and immediately starts it. If the specified image is not present locally, `docker run` can pull the image from Docker Hub.

## Difference between Docker pause and Docker stop

* **docker pause** suspends all processes in the specified containers (uses SIGSTOP). The container’s memory remains allocated and will be used again when the container is resumed.
* **docker stop** sends SIGTERM to the main process inside the container to request graceful shutdown, and after a timeout it sends SIGKILL if the process hasn't exited. Stopping releases the memory used by the container.

<details>

<summary>Signals referenced</summary>

* SIGTERM: termination signal intended to allow graceful shutdown/cleanup.
* SIGKILL: forces immediate termination of the process.
* SIGSTOP: halts the process (pause). Paused processes can be resumed via SIGCONT.

</details>

## Conclusion

**Docker** is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications.

[https://medium.com/tag/cloud-computing?source=post\_page-----f3575e31fc59---------------------------------------](https://medium.com/tag/cloud-computing?source=post_page-----f3575e31fc59---------------------------------------)
