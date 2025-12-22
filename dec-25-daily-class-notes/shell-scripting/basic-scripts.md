# basic scripts

### ✅ Shell Basics

```bash
echo $SHELL    # Check the default shell for the user (e.g., /bin/bash, /bin/zsh)
```

***

### ✅ Shebang

```bash
#!/bin/bash    # Shebang line to specify the script should run using the bash shell
```

***

### ✅ Script: `exp1.sh`

```bash
#!/bin/bash

# Script to print static welcome message

echo -e "Hi\nThis is DevOps class"
echo "Started on Nov"
```

#### 🔧 Execution Methods

```bash
./exp1.sh        # Executes script directly (make sure it's executable)
bash exp1.sh     # Executes using bash explicitly
sh exp1.sh       # Executes using default shell interpreter
```

***

### ✅ Script: `exp2.sh`

```bash
#!/bin/bash

# Script to demonstrate passing arguments to a shell script

echo "This is $1 class"
echo "Started on $2"
```

#### 🔧 Execution Example

```bash
bashCopyEdit./exp2.sh DevOps Nov

# Output:

# This is DevOps class

# Started on Nov
```

***

### ✅ Input Arguments in Shell Script

* `$0` → Script name
* `$1 to $9` → 1st to 9th arguments
* `${10}, ${11}` → 10th argument onward

#### 🧠 Example:

```bash
#!/bin/bash
echo "Script Name: $0"
echo "First Arg : $1"
echo "Tenth Arg: ${10}"
```

***

### ✅ Script: `exp3.sh`

```bash
#!/bin/bash

# Script to print predefined variables

name="Test"
month="Nov"
num="2022"

echo "This is $name platform and month is $month"
echo "The number will be $num"
```

### ✅ Conditional Statement Syntax

```bash
if [ condition ]; then
    # commands if true
else
    # commands if false
fi
```

***

### ✅ Script: `exp4.sh` – Check if Number is 5

```bash
#!/bin/bash

if [ "$1" -eq 5 ]; then
    echo "$1 is Five"
else
    echo "$1 is not Five"
fi
```

***

### ✅ Numeric Comparison Operators in Bash

| Operator | Meaning                  |
| -------- | ------------------------ |
| `-eq`    | Equal to                 |
| `-ne`    | Not equal to             |
| `-lt`    | Less than                |
| `-le`    | Less than or equal to    |
| `-gt`    | Greater than             |
| `-ge`    | Greater than or equal to |

***

### ✅ Script: `exp5.sh` – Biggest of Two Numbers

```bash
#!/bin/bash

if [ "$1" -gt "$2" ]; then
    echo "$1 is Big"
else
    echo "$2 is Big"
fi
```

***

### ✅ Script: `exp6.sh` – Biggest of Two Numbers with Argument Check

```bash
#!/bin/bash

if [ "$#" -ne 2 ]; then
    echo "Enter only two arguments"
    exit 1
fi

if [ "$1" -gt "$2" ]; then
    echo "$1 is Big"
else
    echo "$2 is Big"
fi
```

***

### ✅ Special Shell Variables

| Variable | Meaning                                                       |
| -------- | ------------------------------------------------------------- |
| `$?`     | Exit status of last command (0 = success, non-zero = failure) |
| `$$`     | Process ID of the current shell script                        |
| `$!`     | PID of the last background command                            |
| `$*`     | All arguments as a single string                              |
| `$@`     | All arguments as individual strings (useful in loops)         |
| `$#`     | Total number of input arguments                               |

***

### ✅ Loop Syntax

#### 🔁 `for` Loop

```bash
for i in list; do
    # commands
done
```

#### 🔁 `while` Loop

```bash
while [ condition ]; do
    # commands
done
```

***

### ✅ Script: Check File, Directory, Link, or Not Exist

```bash
#!/bin/bash

echo "Enter the name to check:"
read name

if [ -f "$name" ]; then
    echo "$name is a file"
elif [ -d "$name" ]; then
    echo "$name is a directory"
elif [ -L "$name" ]; then
    echo "$name is a symbolic link"
else
    echo "$name does not exist"
fi
```

***

### ✅ Script : Disk Usage Alert if More Than 90%

```bash
bashCopyEdit#!/bin/bash

# Check disk usage and send alert if usage exceeds 90%

usage=$(df -h . | tail -1 | awk '{print $5}' | tr -d '%')

if [ "$usage" -ge 90 ]; then
    echo "Disk is full. Please take appropriate action." \
    | mail -s "Alert: Disk Usage Full" -c "abc@gmail.com" "devops@gmail.com"
fi
```

***

#### ✅ Cron Job Syntax Reference

| Field        | Range         | Description        |
| ------------ | ------------- | ------------------ |
| Minute       | 0–59          | Minute of the hour |
| Hour         | 0–23          | Hour of the day    |
| Day of Month | 1–31          | Day of the month   |
| Month        | 1–12          | Month of the year  |
| Day of Week  | 0–6 (Sun–Sat) | Day of the week    |

***

#### ✅ Useful Cron Commands

```bash
crontab -l    # List current user's cron jobs
crontab -e    # Edit cron jobs
```

***

#### ✅ Cron Job Examples

```bash
* * * * * /home/ubuntu/shell.sh       # Every minute
0 * * * * /home/ubuntu/shell.sh       # Every hour at 0 minutes

# Run on 28th December 2022 at 1:30 PM on a Wednesday
30 13 28 12 3 /home/ubuntu/shell.sh
```

***

### ✅ Display Employees Over Age 40

#### 📄 input File: `employees.txt`

```
Name    Emp-ID    Age
ABC     202201    35
EFG     202202    40
HIJ     202203    38
LMN     202204    45
XYZ     202205    43
GHY     202206    42
```

#### ✅ Script: `filter_employees.sh`

```bash
#!/bin/bash

# Display employees older than 40

awk 'NR==1 || $3 > 40' employees.txt
```
