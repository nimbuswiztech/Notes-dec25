# ci cd pipeline setup with jenkins sonarqube docker and maven

## Complete CI/CD Pipeline Setup with Jenkins, SonarQube, Docker & Maven

### What is SonarQube?

**SonarQube** is a powerful **static code analysis tool** that continuously inspects code quality and security vulnerabilities in your source code. It performs comprehensive analysis without executing the code, detecting:

* **Bugs and potential issues** before they reach production
* **Security vulnerabilities** and hotspots
* **Code smells** and maintainability issues
* **Code duplications** and unused code
* **Coding standard violations**

SonarQube supports over 30 programming languages including Java, Python, JavaScript, C#, and many more. It integrates seamlessly with CI/CD pipelines, providing quality gates that can automatically fail builds if code doesn't meet defined quality standards.

### Pipeline Architecture Overview

CI/CD Pipeline Architecture with Jenkins, SonarQube, Docker and Maven

This comprehensive setup demonstrates a complete CI/CD pipeline using:

* **Jenkins** on EC2 as the orchestration server
* **Docker** as container runtime and for Docker agents
* **SonarQube** running as Docker container for code quality analysis
* **Maven** as Docker agent for building Java applications
* **Git** for source code management

### Prerequisites

* AWS EC2 instance (t2.medium or larger recommended)
* Ubuntu 22.04 LTS
* Security groups configured to allow:
  * Port 22 (SSH)
  * Port 8080 (Jenkins)
  * Port 9000 (SonarQube)
  * Port 8081 (Application)

### Step-by-Step Implementation

{% stepper %}
{% step %}
### Step 1: EC2 Instance Setup

**Launch EC2 Instance:**

```
bash# Connect to your EC2 instance
ssh -i your-key.pem ubuntu@your-ec2-ip

# Update system packages
sudo apt update
sudo apt upgrade -y
```
{% endstep %}

{% step %}
### Step 2: Jenkins Installation

**Install Java (Jenkins Requirement):**

```
bashsudo apt update
sudo apt install default-jdk -y
java -version
```

**Add Jenkins Repository and Install:**

```
bash# Add Jenkins repository key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# Add Jenkins repository
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt-get update
sudo apt-get install jenkins -y

# Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Access Jenkins:**

* Open browser: `http://your-ec2-ip:8080`
* Use the initial admin password
* Install suggested plugins
* Create admin user
{% endstep %}

{% step %}
### Step 3: Docker Installation

**Install Docker on EC2:**

```
bash# Install Docker dependencies
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

# Add Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt update
sudo apt install docker-ce -y

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add users to docker group
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins

# Restart Jenkins
sudo systemctl restart jenkins

# Verify installation
docker --version
```
{% endstep %}

{% step %}
### Step 4: SonarQube Docker Setup

**Deploy SonarQube Container:**

```
bash# Pull SonarQube image
docker pull sonarqube:latest

# Create persistent volumes
docker volume create sonarqube_data
docker volume create sonarqube_logs
docker volume create sonarqube_extensions

# Run SonarQube container
docker run -d --name sonarqube \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:latest

# Verify container is running
docker ps
```

**Access SonarQube:**

* URL: `http://your-ec2-ip:9000`
* Default credentials: `admin/admin`
* Change password when prompted
{% endstep %}

{% step %}
### Step 5: Jenkins Plugin Installation

**Install Required Plugins:**\
Navigate to **Manage Jenkins > Manage Plugins** and install:

1. **SonarQube Scanner Plugin** - Integrates SonarQube with Jenkins
2. **Docker Plugin** - Docker container management
3. **Docker Pipeline Plugin** - Docker support in pipelines
4. **Pipeline: Stage View Plugin** - Visual pipeline stages
5. **Maven Integration Plugin** - Maven build support
6. **Git Plugin** - Git repository integration
{% endstep %}

{% step %}
### Step 6: Jenkins Global Configuration

**Configure SonarQube Integration:**

1. Go to **Manage Jenkins > Configure System**
2. Find **SonarQube servers** section
3. Add server:
   * **Name**: `SonarQube-Server`
   * **Server URL**: `http://localhost:9000`
   * **Authentication token**: Create in SonarQube

**Generate SonarQube Token:**

1. In SonarQube: **My Account > Security**
2. Generate token: `jenkins-token`
3. Copy the generated token

**Add Jenkins Credentials:**

1. **Manage Jenkins > Credentials**
2. Add **Secret text**:
   * **Secret**: SonarQube token
   * **ID**: `sonar-token`
   * **Description**: `SonarQube Authentication Token`

**Configure Tools:**\
Go to **Manage Jenkins > Global Tool Configuration**:

* **Maven**: Name `Maven-3.9.4`, Install automatically
* **JDK**: Name `JDK-17`, Install automatically
* **SonarQube Scanner**: Name `SonarQube-Scanner`, Install automatically
{% endstep %}

{% step %}
### Step 7: Create Complete Pipeline

**Sample Jenkinsfile for Maven Webapp:**

```
groovypipeline {
    agent any
    
    tools {
        maven 'Maven-3.9.4'
        jdk 'JDK-17'
    }
    
    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/nimbuswiztech/maven_webapp.git'
            }
        }
        
        stage('Build with Maven Docker Agent') {
            agent {
                docker {
                    image 'maven:3.9.4-openjdk-17'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'maven:3.9.4-openjdk-17'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            agent {
                docker {
                    image 'maven:3.9.4-openjdk-17'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=maven-webapp \
                        -Dsonar.host.url=http://sonarqube:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('Package') {
            agent {
                docker {
                    image 'maven:3.9.4-openjdk-17'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.war', allowEmptyArchive: false
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("webapp:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    docker stop webapp-container || true
                    docker rm webapp-container || true
                    docker run -d --name webapp-container -p 8081:8080 webapp:${BUILD_NUMBER}
                '''
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```
{% endstep %}

{% step %}
### Step 8: Application Dockerfile

**Create Dockerfile in project root:**

```
textFROM tomcat:9.0-jdk17

# Remove default webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy WAR file to webapps directory  
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war

# Expose port
EXPOSE 8080

# Start Tomcat
CMD ["catalina.sh", "run"]
```
{% endstep %}

{% step %}
### Step 9: Configure SonarQube Project

**Create Project in SonarQube:**

1. Login to SonarQube (`http://your-ec2-ip:9000`)
2. **Create Project > Manually**
3. Project Key: `maven-webapp`
4. Display Name: `Maven Web Application`
5. Generate project token
{% endstep %}

{% step %}
### Step 10: Create Jenkins Pipeline Job

**Create Pipeline Job:**

1. Jenkins Dashboard > **New Item**
2. Name: `maven-webapp-pipeline`
3. Type: **Pipeline**
4. **Pipeline Definition**: Pipeline script from SCM
5. **SCM**: Git
6. **Repository URL**: `https://github.com/nimbuswiztech/maven_webapp.git`
7. **Script Path**: `Jenkinsfile`
{% endstep %}

{% step %}
### Step 11: Execute Pipeline

1. Go to pipeline job
2. Click **"Build Now"**
3. Monitor progress in **Stage View**
{% endstep %}
{% endstepper %}

### Pipeline Flow Explanation

#### Stage 1: Checkout

* Retrieves source code from Git repository
* Jenkins automatically clones the specified branch

#### Stage 2: Build with Maven Docker Agent

* Spins up Maven Docker container as build agent
* Persists Maven dependencies using volume mapping
* Compiles Java source code

#### Stage 3: Test

* Executes unit tests in Maven Docker agent
* Publishes JUnit test results
* Fails pipeline if tests fail

#### Stage 4: SonarQube Analysis

* Performs static code analysis using SonarQube
* Scans for bugs, vulnerabilities, code smells
* Sends analysis results to SonarQube server

#### Stage 5: Quality Gate

* Waits for SonarQube quality gate evaluation
* **Pipeline fails** if code doesn't meet quality standards
* Ensures only quality code proceeds to deployment

#### Stage 6: Package

* Creates deployable WAR file using Maven
* Archives build artifacts for later use

#### Stage 7: Build Docker Image

* Builds Docker image containing the application
* Tags image with build number for versioning

#### Stage 8: Deploy

* Stops existing application container
* Deploys new version using latest Docker image
* Application becomes available on port 8081

### Key Benefits of This Setup

#### Automated Quality Assurance

* **Static code analysis** prevents bugs reaching production
* **Quality gates** enforce coding standards automatically
* **Continuous monitoring** of technical debt

#### Docker Agent Architecture

* **Ephemeral build environments** ensure clean builds
* **Resource efficiency** - agents created on-demand
* **Consistent build environments** across different projects

#### Complete Automation

* **Triggered by code commits** via webhooks
* **Automated testing** and quality checks
* **Zero-downtime deployments** using Docker containers

### Troubleshooting Common Issues

<details>

<summary>Docker Permission Issues</summary>

```
bashsudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl restart docker
```

</details>

<details>

<summary>SonarQube Connection Problems</summary>

* Verify container is running: `docker ps | grep sonarqube`
* Check security group allows port 9000
* Validate authentication token

</details>

<details>

<summary>Pipeline Failures</summary>

* Review **Console Output** in Jenkins build
* Check tool configurations in Global Tools
* Verify all required plugins are installed

</details>

<details>

<summary>Maven Dependencies</summary>

* Ensure volume mapping preserves `.m2` repository
* Use correct Docker agent configuration

</details>

### Security Best Practices

1. **Use dedicated service accounts** for Jenkins and Docker
2. **Store all credentials securely** in Jenkins credential store
3. **Regular security updates** for all components
4. **Network segmentation** using security groups
5. **Regular backup** of Jenkins configuration and SonarQube data

### Complete Documentation

For the complete setup guide with detailed commands, troubleshooting, and advanced configurations:

jenkins-pipeline.md

This comprehensive CI/CD pipeline provides automated code quality assurance, containerized builds, and seamless deployment for Java web applications, following modern DevOps best practices with industry-standard tools.
