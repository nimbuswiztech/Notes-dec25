# shell script complete guide

Shell Script complete guide

Shell scripting is an essential skill for DevOps engineers, as it empowers you to automate tasks, streamline processes, and manage infrastructure more efficiently. In this comprehensive guide, we’ll take you on a journey from the basics of shell scripting to advanced techniques, complete with real-time examples and code snippets.

{% stepper %}
{% step %}
### Introduction to Shell Scripting

Shell scripting is the art of writing scripts that run in a command-line shell. In the DevOps world, this usually means using Bash (Bourne Again Shell), which is the default shell on most Linux systems.

Shell scripts are used for various purposes, such as automating repetitive tasks, configuring servers, and managing deployments.
{% endstep %}

{% step %}
### Getting Started with Bash

Before diving into scripting, make sure you have a basic understanding of Bash. You can start by opening a linux terminal and trying out simple commands like `ls`, `pwd`, and `echo`.
{% endstep %}

{% step %}
### Basic Script Structure

A Bash script typically starts with a shebang line that specifies the interpreter to use. Here’s a simple script:

{% code title="hello.sh" %}
```bash
#!/bin/bash

# This is a comment
echo "Hello, World!"
```
{% endcode %}

* The `#!/bin/bash` line tells the system to use the Bash interpreter.
* Comments start with `#` and are ignored by the shell.
* `echo` is used to print text to the console.
{% endstep %}

{% step %}
### Variables and Data Types

In Bash, you can declare variables like this:

{% code title="variables.sh" %}
```bash
name="DevOps"
```
{% endcode %}

Bash has no explicit data types. Variables are treated as strings by default, but you can perform arithmetic operations using `(( ))`:

{% code title="arithmetic.sh" %}
```bash
count=5
((count++))
echo "Count: $count"
```
{% endcode %}
{% endstep %}

{% step %}
### Control Structures

Control structures help you make decisions and control the flow of your scripts. Here are some common ones:

* If statements:

{% code title="if.sh" %}
```bash
if [ "$var" == "value" ]; then
    echo "Variable is equal to 'value'"
fi
```
{% endcode %}

* For loops:

{% code title="for.sh" %}
```bash
for fruit in apple banana cherry; do
    echo "I like $fruit"
done
```
{% endcode %}

* While loops:

{% code title="while.sh" %}
```bash
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```
{% endcode %}
{% endstep %}

{% step %}
### Functions

Functions allow you to modularize your code. Here’s how to define and call a function:

{% code title="functions.sh" %}
```bash
say_hello() {
    echo "Hello, $1!"
}
say_hello "Alice"
```
{% endcode %}
{% endstep %}

{% step %}
### File Handling

Dealing with files is common in DevOps tasks. You can read, write, and manipulate files in Bash.

* Reading a file:

{% code title="read_file.sh" %}
```bash
while read line; do
    echo "Line: $line"
done < file.txt
```
{% endcode %}

* Writing to a file:

{% code title="write_file.sh" %}
```bash
echo "Hello, World!" > output.txt
```
{% endcode %}
{% endstep %}

{% step %}
### Advanced Techniques

To become a proficient DevOps scripter, you should explore advanced techniques:

* Command-line arguments: Parse and use command-line arguments in your scripts.
* Error handling: Implement error-checking and logging in your scripts.
* Regular expressions: Use regex for pattern matching and text manipulation.
* Piping and redirection: Combine commands using pipes (`|`) and redirect input/output.
{% endstep %}

{% step %}
### Best Practices

Follow these best practices for writing maintainable and efficient shell scripts:

* Use meaningful variable and function names.
* Comment your code to explain complex logic.
* Modularize your code with functions.
* Test your scripts thoroughly before deploying them.
* Use version control to track changes.
{% endstep %}

{% step %}
### Real-world Examples

Here are some real-world scenarios where shell scripting is invaluable:

* Automating deployments: Write scripts to deploy applications and configurations.
* Server provisioning: Automate server setup and configuration.
* Backup and cleanup: Schedule backups and perform routine system maintenance.
* Monitoring and alerts: Use scripts to monitor system metrics and send alerts.
* Log analysis: Analyze log files for errors and trends.
{% endstep %}
{% endstepper %}

***

File Handling and Text Processing

a. Searching for Keywords in Log Files

Suppose you need to search for specific keywords in log files for troubleshooting. You can use `grep` for this:

{% code title="search_logs.sh" %}
```bash
#!/bin/bash
search_term="error"
log_file="application.log"
if grep -q "$search_term" "$log_file"; then
    echo "Found '$search_term' in $log_file"
else
    echo "No '$search_term' found in $log_file"
fi
```
{% endcode %}

b. Parsing CSV Files

You often need to work with CSV files in DevOps tasks. Here’s a script that reads a CSV file and extracts data:

{% code title="parse_csv.sh" %}
```bash
#!/bin/bash
csv_file="data.csv"
while IFS=',' read -r col1 col2 col3; do
    echo "Column 1: $col1, Column 2: $col2, Column 3: $col3"
done < "$csv_file"
```
{% endcode %}

Automation and Server Management

a. Automating Software Updates

Automation is crucial in DevOps. You can create a script to update your system and installed packages:

{% code title="update_system.sh" %}
```bash
#!/bin/bash

# Update system packages
sudo apt update
sudo apt upgrade -y

# Update Docker containers (if applicable)
docker-compose -f /path/to/docker-compose.yaml pull
docker-compose -f /path/to/docker-compose.yaml up -d
```
{% endcode %}

b. Server Backup Script

Creating regular backups of your servers is essential. Here’s a simple backup script using `rsync`:

{% code title="backup.sh" %}
```bash
#!/bin/bash
backup_dir="/backup"
source_dir="/var/www/html"

# Create a backup directory
mkdir -p "$backup_dir"

# Perform the backup
rsync -av "$source_dir" "$backup_dir"
```
{% endcode %}

Error Handling and Logging

a. Logging Script Output

Logging helps you keep track of script execution and errors:

{% code title="logging.sh" %}
```bash
#!/bin/bash
log_file="/var/log/my_script.log"

# Redirect stdout and stderr to a log file
exec > "$log_file" 2>&1
echo "Script started at $(date)"

# Your script logic here
echo "Script finished at $(date)"
```
{% endcode %}

b. Error Handling

You can add error handling to your scripts using `set -e` to exit on error:

{% code title="error_handling.sh" %}
```bash
#!/bin/bash
set -e

# Your script logic here

# If an error occurs, the script will exit here
```
{% endcode %}

Automation with Cron Jobs

Cron jobs are scheduled tasks in Unix-like systems. You can use them for regular DevOps tasks:

{% code title="crontab_example" %}
```cron
# Edit the crontab using 'crontab -e'

# This script will run every day at midnight
0 0 * * * /path/to/your/script.sh
```
{% endcode %}

Managing Environment Variables

Managing environment variables is crucial for configuration in DevOps:

{% code title="env_vars.sh" %}
```bash
#!/bin/bash

# Define environment variables
export DATABASE_URL="mysql://username:password@localhost/database"

# Use environment variables in your scripts
echo "Database URL: $DATABASE_URL"
```
{% endcode %}

Check web status

Let’s create a script to automate a common DevOps task — checking the status of a web server. Create a file named `check_web_status.sh` and add the following code:

{% code title="check_web_status.sh" %}
```bash
#!/bin/bash

# Define a function to check the status of a website
check_website() {
    local url="$1"
    local response=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    if [ "$response" == "200" ]; then
        echo "Website $url is up and running!"
    else
        echo "Website $url is down!"
    fi
}

# Call the function with a sample website
check_website "https://www.example.com"
```
{% endcode %}

In this script:

* We use the `curl` command to send an HTTP request to the website.
* The `-s` flag makes `curl` operate in silent mode, suppressing progress and error messages.
* `-o /dev/null` discards the response body.
* `-w "%{http_code}"` instructs `curl` to print only the HTTP response code.
* We compare the response code to determine if the website is up or down.

Run the script with `./check_web_status.sh`, and it will check the status of "https://www.example.com" and provide the result.

Automate server monitoring

Let’s create a script to automate server monitoring by checking CPU usage. Create a file named `monitor_cpu.sh` and add the following code:

{% code title="monitor_cpu.sh" %}
```bash
#!/bin/bash

# Get CPU usage percentage
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
echo "CPU Usage: $cpu_usage%"
```
{% endcode %}

In this script:

* We use the `top` command to get CPU usage information.
* `top -bn1` runs `top` in batch mode for a single iteration.
* `grep "Cpu(s)"` extracts the line with CPU usage details.
* `awk '{print $2 + $4}'` calculates the sum of user and system CPU usage percentages.

Run the script with `./monitor_cpu.sh`, and it will display the current CPU usage percentage.

Monitor disk space

Let’s create a script to automate server monitoring by checking disk space. Create a file named `monitor_disk_space.sh` and add the following code:

{% code title="monitor_disk_space.sh" %}
```bash
#!/bin/bash

# Set the threshold for disk usage (in percentage)
threshold=90

# Get the disk usage percentage
disk_usage=$(df -h / | tail -n 1 | awk '{print $5}' | sed 's/%//')
if [ "$disk_usage" -ge "$threshold" ]; then
    echo "Disk space is running low! Disk Usage: $disk_usage%"
else
    echo "Disk space is within acceptable limits. Disk Usage: $disk_usage%"
fi
```
{% endcode %}

In this script:

* We set a threshold for disk usage (in this case, 90%).
* We use the `df` command to get disk usage information for the root filesystem (`/`).
* `tail -n 1` extracts the last line of the `df` output.
* `awk '{print $5}'` extracts the fifth column, which contains the usage percentage.
* We compare the usage percentage to the threshold and provide a warning if it exceeds the limit.

Run the script with `./monitor_disk_space.sh`, and it will check the disk space usage and issue a warning if it's above the threshold.

Automate package installations using Functions

Let’s create a script to automate the installation of packages using a function. Create a file named `install_packages.sh` and add the following code:

{% code title="install_packages.sh" %}
```bash
#!/bin/bash

# Define a function to install packages
install_packages() {
    local package_manager=""        # Check which package manager is available
    if [ -x "$(command -v apt-get)" ]; then
        package_manager="apt-get"
    elif [ -x "$(command -v yum)" ]; then
        package_manager="yum"
    else
        echo "Error: No supported package manager found."
        exit 1
    fi
    echo "Updating package lists..."
    sudo $package_manager update -y
    echo "Installing packages..."
    sudo $package_manager install -y package1 package2 package3
}

# Call the function to install packages
install_packages
```
{% endcode %}

In this script:

* We define a function `install_packages` that checks for available package managers (`apt-get` or `yum`) and installs specified packages.
* We use `command -v` to check if a command is available.
* The `-y` flag is used to automatically answer yes to prompts during package installation.

Run the script with `./install_packages.sh`, and it will update the package lists and install the specified packages based on the available package manager.

These are just a few examples of how shell scripting can be applied in real-world DevOps scenarios. As you gain experience, you’ll encounter more complex tasks that require custom scripts tailored to your infrastructure and requirements. Remember to follow best practices, document your scripts, and continually refine your skills to become a more proficient DevOps engineer.

In conclusion, mastering shell scripting is a critical skill for DevOps engineers. This guide provides you with a solid foundation and real-world examples to help you become proficient in shell scripting and streamline your DevOps tasks. Happy scripting!
