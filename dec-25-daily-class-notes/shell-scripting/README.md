# Shell scripting

## Shell Script complete guide

Shell scripting is an essential skill for DevOps engineers, as it empowers you to automate tasks, streamline processes, and manage infrastructure more efficiently. In this comprehensive guide, we’ll take you on a journey from the basics of shell scripting to advanced techniques, complete with real-time examples and code snippets.

#### Table of Contents <a href="#id-90bf" id="id-90bf"></a>

1. Introduction to Shell Scripting
2. Getting Started with Bash
3. Basic Script Structure¯
4. Variables and Data Types
5. Control Structures
6. Functions
7. File Handling
8. Advanced Techniques
9. Best Practices
10. Real-world Examples

#### 1. Introduction to Shell Scripting <a href="#id-6610" id="id-6610"></a>

Shell scripting is the art of writing scripts that run in a command-line shell. In the DevOps world, this usually means using Bash (Bourne Again Shell), which is the default shell on most Linux systems.

Shell scripts are used for various purposes, such as automating repetitive tasks, configuring servers, and managing deployments.

#### 2. Getting Started with Bash <a href="#id-6b11" id="id-6b11"></a>

Before diving into scripting, make sure you have a basic understanding of Bash. You can start by opening a linux terminal and trying out simple commands like `ls`, `pwd`, and `echo`.

#### 3. Basic Script Structure <a href="#e8f6" id="e8f6"></a>

A Bash script typically starts with a shebang line that specifies the interpreter to use. Here’s a simple script:

```
#!/bin/bash
# This is a comment
echo "Hello, World!"
```

* The `#!/bin/bash` line tells the system to use the Bash interpreter.
* Comments start with `#` and are ignored by the shell.
* `echo` is used to print text to the console.

#### 4. Variables and Data Types <a href="#id-81a8" id="id-81a8"></a>

In Bash, you can declare variables like this:

```
name="DevOps"
```

Bash has no explicit data types. Variables are treated as strings by default, but you can perform arithmetic operations using `(( ))`:

```
count=5
((count++))
echo "Count: $count"
```

#### 5. Control Structures <a href="#b32c" id="b32c"></a>

Control structures help you make decisions and control the flow of your scripts. Here are some common ones:

* If statements:

```
if [ "$var" == "value" ]; then
    echo "Variable is equal to 'value'"
fi
```

* For loops:

```
for fruit in apple banana cherry; do
    echo "I like $fruit"
done
```

* While loops:

```
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

#### 6. Functions <a href="#e525" id="e525"></a>

Functions allow you to modularize your code. Here’s how to define and call a function:

```
say_hello() {
    echo "Hello, $1!"
}say_hello "Alice"
```

#### 7. File Handling <a href="#id-00ec" id="id-00ec"></a>

Dealing with files is common in DevOps tasks. You can read, write, and manipulate files in Bash:

* Reading a file:

```
while read line; do
    echo "Line: $line"
done < file.txt
```

* Writing to a file:

```
echo "Hello, World!" > output.txt
```

#### 8. Advanced Techniques <a href="#c959" id="c959"></a>

To become a proficient DevOps scripter, you should explore advanced techniques:

* Command-line arguments: Parse and use command-line arguments in your scripts.
* Error handling: Implement error-checking and logging in your scripts.
* Regular expressions: Use regex for pattern matching and text manipulation.
* Piping and redirection: Combine commands using pipes (`|`) and redirect input/output.

#### 9. Best Practices <a href="#id-33f2" id="id-33f2"></a>

Follow these best practices for writing maintainable and efficient shell scripts:

* Use meaningful variable and function names.
* Comment your code to explain complex logic.
* Modularize your code with functions.
* Test your scripts thoroughly before deploying them.
* Use version control to track changes.

#### 10. Real-world Examples <a href="#id-82fb" id="id-82fb"></a>

Here are some real-world scenarios where shell scripting is invaluable:

* Automating deployments: Write scripts to deploy applications and configurations.
* Server provisioning: Automate server setup and configuration.
* Backup and cleanup: Schedule backups and perform routine system maintenance.
* Monitoring and alerts: Use scripts to monitor system metrics and send alerts.
* Log analysis: Analyze log files for errors and trends.

#### File Handling and Text Processing <a href="#id-7e44" id="id-7e44"></a>

#### a. Searching for Keywords in Log Files <a href="#id-360c" id="id-360c"></a>

Suppose you need to search for specific keywords in log files for troubleshooting. You can use `grep` for this:

```
#!/bin/bash
search_term="error"
log_file="application.log"
if grep -q "$search_term" "$log_file"; then
    echo "Found '$search_term' in $log_file"
else
    echo "No '$search_term' found in $log_file"
fi
```

#### b. Parsing CSV Files <a href="#d0d2" id="d0d2"></a>

You often need to work with CSV files in DevOps tasks. Here’s a script that reads a CSV file and extracts data:

```
#!/bin/bash
csv_file="data.csv"
while IFS=',' read -r col1 col2 col3; do
    echo "Column 1: $col1, Column 2: $col2, Column 3: $col3"
done < "$csv_file"
```

#### Automation and Server Management <a href="#id-35d8" id="id-35d8"></a>

#### a. Automating Software Updates <a href="#id-48c8" id="id-48c8"></a>

Automation is crucial in DevOps. You can create a script to update your system and installed packages:

```
#!/bin/bash
# Update system packages
sudo apt update
sudo apt upgrade -y
# Update Docker containers (if applicable)
docker-compose -f /path/to/docker-compose.yaml pull
docker-compose -f /path/to/docker-compose.yaml up -d
```

#### b. Server Backup Script <a href="#id-3e68" id="id-3e68"></a>

Creating regular backups of your servers is essential. Here’s a simple backup script using `rsync`:

```
#!/bin/bash
backup_dir="/backup"
source_dir="/var/www/html"
# Create a backup directory
mkdir -p "$backup_dir"
# Perform the backup
rsync -av "$source_dir" "$backup_dir"
```

#### Error Handling and Logging <a href="#e621" id="e621"></a>

#### a. Logging Script Output <a href="#id-22af" id="id-22af"></a>

Logging helps you keep track of script execution and errors:

```
#!/bin/bash
log_file="/var/log/my_script.log"
# Redirect stdout and stderr to a log file
exec > "$log_file" 2>&1
echo "Script started at $(date)"
# Your script logic here
echo "Script finished at $(date)"
```

#### b. Error Handling <a href="#id-0c3c" id="id-0c3c"></a>

You can add error handling to your scripts using `set -e` to exit on error:

```
#!/bin/bash
set -e
# Your script logic here
# If an error occurs, the script will exit here
```

#### Automation with Cron Jobs <a href="#d48b" id="d48b"></a>

Cron jobs are scheduled tasks in Unix-like systems. You can use them for regular DevOps tasks:

```
# Edit the crontab using 'crontab -e'
# This script will run every day at midnight
0 0 * * * /path/to/your/script.sh
```

#### Managing Environment Variables <a href="#f52e" id="f52e"></a>

Managing environment variables is crucial for configuration in DevOps:

```
#!/bin/bash
# Define environment variables
export DATABASE_URL="mysql://username:password@localhost/database"
# Use environment variables in your scripts
echo "Database URL: $DATABASE_URL"
```

#### Check web status <a href="#f858" id="f858"></a>

Let’s create a script to automate a common DevOps task — checking the status of a web server. Create a file named `check_web_status.sh` and add the following code:

```
#!/bin/bash

# Define a function to check the status of a website
check_website() {
    local url="$1"
    local response=$(curl -s -o /dev/null -w "%{http_code}" "$url")        if [ "$response" == "200" ]; then
        echo "Website $url is up and running!"
    else
        echo "Website $url is down!"
    fi
}
# Call the function with a sample website
check_website "https://www.example.com"
```

In this script:

* We use the `curl` command to send an HTTP request to the website.
* The `-s` flag makes `curl` operate in silent mode, suppressing progress and error messages.
* `-o /dev/null` discards the response body.
* `-w "%{http_code}"` instructs `curl` to print only the HTTP response code.
* We compare the response code to determine if the website is up or down.

Run the script with `./check_web_status.sh`, and it will check the status of "[https://www.example.com](https://www.example.com/)" and provide the result.

#### Automate server monitoring <a href="#c40b" id="c40b"></a>

Let’s create a script to automate server monitoring by checking CPU usage. Create a file named `monitor_cpu.sh` and add the following code:

```
#!/bin/bash
# Get CPU usage percentage
cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
echo "CPU Usage: $cpu_usage%"
```

In this script:

* We use the `top` command to get CPU usage information.
* `top -bn1` runs `top` in batch mode for a single iteration.
* `grep "Cpu(s)"` extracts the line with CPU usage details.
* `awk '{print $2 + $4}'` calculates the sum of user and system CPU usage percentages.

Run the script with `./monitor_cpu.sh`, and it will display the current CPU usage percentage.

#### Monitor disk space <a href="#id-34fd" id="id-34fd"></a>

Let’s create a script to automate server monitoring by checking disk space. Create a file named `monitor_disk_space.sh` and add the following code:

```
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

In this script:

* We set a threshold for disk usage (in this case, 90%).
* We use the `df` command to get disk usage information for the root filesystem (`/`).
* `tail -n 1` extracts the last line of the `df` output.
* `awk '{print $5}'` extracts the fifth column, which contains the usage percentage.
* We compare the usage percentage to the threshold and provide a warning if it exceeds the limit.

Run the script with `./monitor_disk_space.sh`, and it will check the disk space usage and issue a warning if it's above the threshold.

#### Automate package installations using Functions <a href="#a574" id="a574"></a>

Let’s create a script to automate the installation of packages using a function. Create a file named `install_packages.sh` and add the following code:

```
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

In this script:

* We define a function `install_packages` that checks for available package managers (`apt-get` or `yum`) and installs specified packages.
* We use `command -v` to check if a command is available.
* The `-y` flag is used to automatically answer yes to prompts during package installation.

Run the script with `./install_packages.sh`, and it will update the package lists and install the specified packages based on the available package manager.

These are just a few examples of how shell scripting can be applied in real-world DevOps scenarios. As you gain experience, you’ll encounter more complex tasks that require custom scripts tailored to your infrastructure and requirements. Remember to follow best practices, document your scripts, and continually refine your skills to become a more proficient DevOps engineer.

In conclusion, mastering shell scripting is a critical skill for DevOps engineers. This guide provides you with a solid foundation and real-world examples to help you become proficient in shell scripting and streamline your DevOps tasks. Happy scripting!
