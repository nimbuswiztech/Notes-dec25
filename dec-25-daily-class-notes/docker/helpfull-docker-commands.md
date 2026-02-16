# helpfull docker commands

| Category              | Command                                    | Description                                           | Example                                    | Use case                                           |
| --------------------- | ------------------------------------------ | ----------------------------------------------------- | ------------------------------------------ | -------------------------------------------------- |
| Basic Info            | `docker --help`                            | Display all available Docker commands and options     | `docker --help`                            | Getting started - see all available commands       |
| Basic Info            | `docker version`                           | Show Docker version information for client and server | `docker version`                           | Check installed Docker version                     |
| Basic Info            | `docker info`                              | Display system-wide Docker information                | `docker info`                              | Get detailed Docker system information             |
| Image Management      | `docker images`                            | List all Docker images on your system                 | `docker images`                            | See all downloaded/built images                    |
| Image Management      | `docker pull`                              | Download an image from Docker registry (Docker Hub)   | `docker pull ubuntu:latest`                | Download images before using them                  |
| Image Management      | `docker build`                             | Build an image from a Dockerfile                      | `docker build -t myapp:latest .`           | Create custom images from Dockerfile               |
| Image Management      | `docker rmi`                               | Remove one or more images                             | `docker rmi image_id`                      | Delete unwanted images to save space               |
| Image Management      | `docker image ls --filter "dangling=true"` | List dangling images (untagged images)                | `docker image ls --filter "dangling=true"` | Find images that need cleanup (from our history)   |
| Image Management      | `docker image prune`                       | Remove dangling images                                | `docker image prune`                       | Clean up unused dangling images (from our history) |
| Container Management  | `docker run`                               | Create and start a new container from an image        | `docker run -d -p 8080:80 nginx`           | Launch applications in containers                  |
| Container Management  | `docker ps`                                | List running containers                               | `docker ps`                                | See what containers are currently running          |
| Container Management  | `docker ps -a`                             | List all containers (running and stopped)             | `docker ps -a`                             | See all containers including stopped ones          |
| Container Management  | `docker start`                             | Start a stopped container                             | `docker start container_name`              | Resume a previously stopped container              |
| Container Management  | `docker stop`                              | Stop a running container                              | `docker stop container_name`               | Gracefully shutdown a running container            |
| Container Management  | `docker restart`                           | Restart a container                                   | `docker restart container_name`            | Restart container after configuration changes      |
| Container Management  | `docker rm`                                | Remove one or more containers                         | `docker rm container_name`                 | Delete stopped containers to free space            |
| Container Interaction | `docker exec -it`                          | Execute commands inside a running container           | `docker exec -it container_name /bin/bash` | Access container shell for debugging/configuration |
| Container Interaction | `docker logs`                              | Fetch logs from a container                           | `docker logs -f container_name`            | Debug issues by viewing container logs             |
| Container Interaction | `docker inspect`                           | Get detailed information about a container or image   | `docker inspect container_name`            | Get detailed configuration and status info         |
| Container Interaction | `docker stats`                             | Display live resource usage statistics                | `docker stats`                             | Monitor container resource consumption             |
| Container Interaction | `docker cp`                                | Copy files between container and host                 | `docker cp file.txt container_name:/path/` | Transfer files to/from containers                  |
| Network Management    | `docker network ls`                        | List all Docker networks                              | `docker network ls`                        | See available networks for containers              |
| Network Management    | `docker network create`                    | Create a new network                                  | `docker network create mynetwork`          | Create custom networks for container communication |
| Volume Management     | `docker volume ls`                         | List all Docker volumes                               | `docker volume ls`                         | See persistent storage volumes                     |
| Volume Management     | `docker volume create`                     | Create a new volume                                   | `docker volume create myvolume`            | Create persistent storage for containers           |
| Cleanup               | `docker container prune`                   | Remove all stopped containers                         | `docker container prune`                   | Clean up stopped containers (from our history)     |
| Cleanup               | `docker system prune`                      | Remove unused data (containers, networks, images)     | `docker system prune`                      | General cleanup of unused Docker resources         |
| Cleanup               | `docker system df`                         | Show Docker disk usage                                | `docker system df`                         | Check how much disk space Docker is using          |

### Essential Command Patterns to Practice

Basic Workflow Commands:

{% code title="Basic workflow" %}
```bash
# Pull, run, and manage containers
docker pull nginx
docker run -d --name webserver -p 8080:80 nginx
docker ps
docker logs webserver
docker stop webserver
docker rm webserver
```
{% endcode %}

Debugging and Inspection:

{% code title="Debugging and inspection" %}
```bash
# Access container shell and inspect
docker exec -it container_name /bin/bash
docker inspect container_name
docker stats
```
{% endcode %}

Cleanup and Maintenance:

{% code title="Cleanup and maintenance" %}
```bash
# Regular cleanup routine
docker container prune
docker image prune
docker system prune
docker system df
```
{% endcode %}
