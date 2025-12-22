# scripts 2

#### ✅ 1. Clean-Up Script (cleanup.sh)

{% hint style="info" %}
Script to keep only the latest 20 files in the current directory.
{% endhint %}

{% code title="cleanup.sh" %}
```bash
#!/bin/bash

# Script to keep only the latest 20 files in the current directory

total_files=$(ls -1 | wc -l)
files_to_delete=$((total_files - 20))

if [ "$files_to_delete" -gt 0 ]; then
    echo "Deleting $files_to_delete oldest files..."
    ls -1t | tail -n "$files_to_delete" | xargs -d '\n' rm -rf
else
    echo "Less than or equal to 20 files, nothing to delete."
fi
```
{% endcode %}

***

#### ✅ 2. Service Health Check & Restart (check\_services.sh)

{% hint style="info" %}
Monitor and restart stopped services, and send email notification if needed.
{% endhint %}

{% code title="check_services.sh" %}
```bash
#!/bin/bash

# Monitor and restart stopped services, send email notification if needed

services="serv1 serv2 serv3"
log_file="/tmp/stopped_services.log"

> "$log_file"  # Clear log file

for service in $services; do
    if ! pgrep -f "$service" > /dev/null; then
        echo "Service $service is not running." >> "$log_file"
        echo "Attempting to restart $service..." >> "$log_file"
        
        sudo service "$service" restart >> "$log_file" 2>&1

        echo "Service $service was down and has been restarted. Please verify." \
        | mail -s "Alert: $service service was stopped" -c abc@gmail.com group@gmail.com
    fi
done
```
{% endcode %}

***

#### ✅ 3. Disk Space Monitor (memorycheck.sh)

{% hint style="info" %}
Check if current disk usage exceeds threshold and send alert.
{% endhint %}

{% code title="memorycheck.sh" %}
```bash
#!/bin/bash

# Check if current disk usage exceeds threshold and send alert

usage=$(df -h . | tail -1 | awk '{print $(NF-1)}' | tr -d '%')

echo "Consumed disk space: ${usage}%"

if [ "$usage" -ge 90 ]; then
    echo "Disk usage is above 90%. Please take action." \
    | mail -s "Alert: Disk Usage Exceeded 90%" -c abc@gmail.com abc@gmail.com
fi
```
{% endcode %}
