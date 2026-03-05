# docker files

To configure a Jenkins pipeline to use a Docker agent for Git operations (such as cloning a repository), you can specify the Docker agent using the `agent` directive in your Jenkinsfile. This will run your pipeline (or stages) inside a specified Docker container, giving you a consistent environment for your steps. Below is a simple example of how to structure your Jenkinsfile:

{% code title="Jenkinsfile" %}
```groovy
pipeline {
    agent none

    environment {
        M2_HOME = "/usr/share/maven"
        PATH = "${M2_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout and Build') {
            agent {
                docker {
                    image 'maven:3.8.1-adoptopenjdk-11'
                }
            }
            steps {
                // Checkout the Git repository
                git branch: 'main', url: 'https://github.com/nimbuswiztech/maven_webapp.git'

                // Build with Maven
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            agent {
                docker {
                    image 'maven:3.8.1-adoptopenjdk-11'
                }
            }
            steps {
                sh 'mvn test'
            }
        }
    }
}
```
{% endcode %}

{% hint style="info" %}
Key Points:

* The `docker` agent runs the defined stages within the specified Docker image.
* Use an image that includes Git (e.g., `alpine/git`, `ubuntu:latest` with Git installed, or your own Dockerfile).
* The pipeline automatically checks out your code if you use the `git` step inside your pipeline.
* You can further customize the Docker container by passing arguments (like volume mounts, environment variables, etc.), depending on your requirements.
{% endhint %}

{% hint style="info" %}
Tips:

* For projects that require multiple environments, you can use different Docker images for each stage by specifying agent in the stage block.
* If you require SSH keys or credentials inside your Docker container for private repositories, you can mount them using the `args` option.
{% endhint %}

This setup will ensure your Jenkins pipeline checks out code and runs all build steps inside a Dockerized environment, simplifying dependency management and environment consistency across builds.

{% stepper %}
{% step %}
### Use `sudo` to run Docker commands

Try prefixing your command with `sudo`:

```
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube
```
{% endstep %}

{% step %}
### Add your user to the `docker` group

This lets you run Docker without `sudo`.

Run these commands:

```
sudo usermod -aG docker $USER
```

Then **log out and log back in** (or restart the session) for the changes to take effect.

After that, test running Docker again without sudo:

```
docker run -d --name sonarqube -p 9000:9000 sonarqube
```
{% endstep %}
{% endstepper %}

{% hint style="warning" %}
Summary:

* The permission issue is because your user is not in the `docker` group which controls access to the Docker daemon.
* Adding your user to the `docker` group or using `sudo` resolves this.

Try the `sudo` approach first if you want a quick fix, or add your user to the group for a long-term solution.
{% endhint %}
