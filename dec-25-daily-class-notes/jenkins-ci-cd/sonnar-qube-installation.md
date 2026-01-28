# sonnar qube installation

{% stepper %}
{% step %}
### Launch EC2 Instance

* Log in to AWS Management Console.
* Launch an EC2 instance:
  * Choose Amazon Linux 2 or Ubuntu Server.
  * Select instance type (e.g., t2.medium or better recommended for SonarQube).
  * Configure security groups to allow inbound ports:
    * SSH (port 22) for access
    * HTTP (port 9000) for SonarQube UI
{% endstep %}

{% step %}
### Connect to EC2 Instance

Use SSH to connect to your instance from your terminal:

```bash
ssh -i /path/to/your-key.pem ec2-user@<EC2-PUBLIC-IP>
```
{% endstep %}

{% step %}
### Update System Packages

{% tabs %}
{% tab title="Amazon Linux 2" %}
```bash
sudo yum update -y
```
{% endtab %}

{% tab title="Ubuntu" %}
```bash
sudo apt update && sudo apt upgrade -y
```
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Install Java (SonarQube requires Java 11 or higher)

{% tabs %}
{% tab title="Amazon Linux 2" %}
```bash
sudo amazon-linux-extras install java-openjdk11 -y
```
{% endtab %}

{% tab title="Ubuntu" %}
```bash
sudo apt install openjdk-11-jdk -y
```
{% endtab %}
{% endtabs %}

Verify Java installation:

```bash
java -version
```
{% endstep %}

{% step %}
### Create SonarQube User

Create a dedicated user to run SonarQube:

```bash
sudo useradd sonar
sudo passwd sonar
```
{% endstep %}

{% step %}
### Download SonarQube

Navigate to /opt (or choose a directory of your preference) and download the SonarQube Community Edition:

```bash
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.0.0.60804.zip
```

Note: Replace the link with the current version from the SonarQube download page if needed.
{% endstep %}

{% step %}
### Unzip SonarQube

{% tabs %}
{% tab title="Amazon Linux 2" %}
```bash
sudo yum install unzip -y
```
{% endtab %}

{% tab title="Ubuntu" %}
```bash
sudo apt install unzip -y
```
{% endtab %}
{% endtabs %}

Then unzip and set ownership:

```bash
sudo unzip sonarqube-10.0.0.60804.zip
sudo mv sonarqube-10.0.0.60804 sonarqube
sudo chown -R sonar:sonar sonarqube
```
{% endstep %}

{% step %}
### Configure SonarQube

Edit the SonarQube configuration file:

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

* Set database connection if you plan to use an external database (by default it uses embedded H2 for testing).
* (Optional) Edit `sonar.web.host=0.0.0.0` to make it accessible on all interfaces.

Save and exit.
{% endstep %}

{% step %}
### (Optional) Install and Configure PostgreSQL (for production)

{% hint style="info" %}
SonarQube recommends PostgreSQL or MySQL for production. The embedded H2 DB is suitable for testing only.
{% endhint %}

If you want PostgreSQL installed (example for Amazon Linux 2):

```bash
sudo amazon-linux-extras enable postgresql13
sudo yum install postgresql postgresql-server postgresql-devel postgresql-contrib postgresql-docs -y
sudo service postgresql initdb
sudo service postgresql start
sudo -i -u postgres psql
```

Then create the database and user for SonarQube inside the psql prompt.
{% endstep %}

{% step %}
### Start SonarQube

Switch to the sonar user and start SonarQube:

```bash
sudo su - sonar
cd /opt/sonarqube/bin/linux-x86-64
./sonar.sh start
```
{% endstep %}

{% step %}
### Access SonarQube Web Interface

*   Open your browser and navigate to:

    ```
    http://<EC2-PUBLIC-IP>:9000
    ```
*   Login with default credentials:

    ```
    Username: admin
    Password: admin
    ```
{% endstep %}

{% step %}
### Optional — Configure SonarQube as a systemd Service

Create the systemd unit file `/etc/systemd/system/sonarqube.service` with the following content:

```ini
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
User=sonar
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload systemd and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
```
{% endstep %}
{% endstepper %}

If you want this adjusted for a different OS or to cover production settings with an external database, tell me which OS and database you'd like and I can adapt the steps.
