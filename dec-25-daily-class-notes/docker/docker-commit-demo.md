# docker commit demo

## Docker Commit Demonstration: Creating Custom Images with Java and Tomcat

This comprehensive guide demonstrates how to use Docker commit to create a new image from a running container after installing Java and Tomcat on Ubuntu.

### Prerequisites

Before starting, ensure Docker is installed on your system. You can verify this by running:

```
docker --version
```

{% stepper %}
{% step %}
### Pull the Ubuntu Base Image

First, pull the latest Ubuntu image from Docker Hub:

```
docker pull ubuntu:latest
```

This downloads the base Ubuntu image that we'll use as our starting point.
{% endstep %}

{% step %}
### Run Ubuntu Container in Interactive Mode

Start an interactive Ubuntu container that allows you to execute commands inside it:

```
docker run -it --name ubuntu-java-tomcat ubuntu:latest /bin/bash
```

The flags used here are:

* `-i`: Keeps standard input open for interaction
* `-t`: Allocates a pseudo-terminal for command-line interface
* `--name ubuntu-java-tomcat`: Assigns a memorable name to the container
* `/bin/bash`: Starts the Bash shell inside the container
{% endstep %}

{% step %}
### Update the Package Repository

Once inside the container, update the package lists:

```
apt update
apt upgrade -y
```

This ensures you have access to the latest package versions.
{% endstep %}

{% step %}
### Install Java (OpenJDK)

Install the Java Development Kit, which is required for Tomcat:

```
apt install -y default-jdk
```

Verify the Java installation:

```
java -version
```

You should see output similar to OpenJDK version information.
{% endstep %}

{% step %}
### Create Tomcat User and Group

For security purposes, create a dedicated user for running Tomcat:

```
# Create tomcat group
groupadd tomcat

# Create tomcat user with specific settings
useradd -s /bin/false -g tomcat -d /opt/tomcat tomcat

# Create the Tomcat directory
mkdir -p /opt/tomcat
```
{% endstep %}

{% step %}
### Download and Install Apache Tomcat

Navigate to the temporary directory and download Tomcat:

```
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
```

Extract Tomcat to the `/opt/tomcat` directory:

```
tar -xzf apache-tomcat-10.1.26.tar.gz -C /opt/tomcat --strip-components=1
```

Set appropriate permissions:

```
chown -R tomcat: /opt/tomcat
chmod -R u+x /opt/tomcat/bin
```
{% endstep %}

{% step %}
### Configure Tomcat (Optional)

You can modify Tomcat's default port from 8080 to another port by editing the server configuration:

```
# Edit the server.xml file
nano /opt/tomcat/conf/server.xml
```

Look for the Connector element and modify the port attribute:

```
xml<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000" redirectPort="8443" />
```
{% endstep %}

{% step %}
### Test Tomcat Installation

Start Tomcat to verify the installation:

```
/opt/tomcat/bin/startup.sh
```

You should see startup messages indicating Tomcat is running. By default, Tomcat runs on port 8080.
{% endstep %}

{% step %}
### Exit the Container

Once you've completed the installation and configuration, exit the container:

```
exit
```
{% endstep %}

{% step %}
### Commit Changes to Create New Image

Now comes the key step - using `docker commit` to create a new image from your modified container:

```
docker commit -a "Your Name" -m "Ubuntu with Java and Tomcat installed" ubuntu-java-tomcat my-ubuntu-tomcat:v1.0
```

The commit command options are:

* `-a "Your Name"`: Sets the author of the image
* `-m "message"`: Adds a descriptive commit message
* `ubuntu-java-tomcat`: The container name to commit
* `my-ubuntu-tomcat:v1.0`: The new image name and tag
{% endstep %}

{% step %}
### Verify the New Image

Check that your new image was created successfully:

```
docker images
```

You should see `my-ubuntu-tomcat` listed among your local images.
{% endstep %}

{% step %}
### Run Container from New Image with Port Exposure

Now run a container from your newly created image, exposing Tomcat's port to the host:

```
docker run -d -p 8080:8080 --name tomcat-server my-ubuntu-tomcat:v1.0 /opt/tomcat/bin/catalina.sh run
```

The flags used are:

* `-d`: Runs the container in detached mode (background)
* `-p 8080:8080`: Maps host port 8080 to container port 8080
* `--name tomcat-server`: Assigns a name to the running container
* `/opt/tomcat/bin/catalina.sh run`: Starts Tomcat in the foreground
{% endstep %}

{% step %}
### Verify Tomcat is Running

Check if your container is running:

```
docker ps
```

Access Tomcat through your web browser by navigating to:

```
thttp://localhost:8080
```

You should see the Tomcat welcome page, confirming that your custom image is working correctly.
{% endstep %}
{% endstepper %}

### Additional Docker Commit Options

The `docker commit` command supports several useful options:

| Option          | Description                                        |
| --------------- | -------------------------------------------------- |
| `-a, --author`  | Set the author name for the image                  |
| `-m, --message` | Add a commit message describing the changes        |
| `-p, --pause`   | Pause the container during commit (default: true)  |
| `-c, --change`  | Apply Dockerfile instructions to the created image |

### Best Practices and Considerations

Security Considerations:

* Always run Tomcat with a dedicated user account, never as root
* Set appropriate file permissions for Tomcat directories
* Consider changing the default port for additional security

Docker Commit vs Dockerfile:

* Docker commit is useful for quick prototyping and debugging
* For production environments, prefer Dockerfiles for better reproducibility
* Committed images can become large; clean up unnecessary files before committing

Port Management:

* Exposed ports are metadata that document which ports the application uses
* Published ports actually map container ports to host ports
* Use `-p` flag to publish ports when running containers

### Troubleshooting (expandable)

<details>

<summary>Container Exits Immediately</summary>

* Ensure you provide a command that keeps the container running
* For Tomcat, use `catalina.sh run` instead of `startup.sh` in detached mode

</details>

<details>

<summary>Permission Denied Errors</summary>

* Run Docker commands with `sudo` if you encounter permission issues
* Ensure your user is in the docker group for non-root access

</details>

<details>

<summary>Port Already in Use</summary>

* Check if port 8080 is already occupied by another service
* Use a different host port mapping (e.g., `-p 8090:8080`)

</details>
