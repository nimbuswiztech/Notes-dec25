# scripts 1

#### Bash Script (memorycheck.sh)

{% code title="memorycheck.sh" %}
```bash
#!/bin/bash

# Fetch disk usage percentage for the current directory
space=$(df -h . | tail -1 | awk '{print $(NF-1)}' | tr -d '%')

echo "Consumed disk space is ${space}%"

# Check if disk usage is greater than or equal to 90%
if [ "$space" -ge 90 ]; then
    echo "Disk space is more than 90%. Please take appropriate action." \
    | mail -s "Alert: Disk Space Exceeded 90%" -c "abc@gmail.com" "abc@gmail.com"
fi
```
{% endcode %}

#### Notes

* `tr -d '%'` is preferred over `sed 's/%/ /g'` for removing `%`.
* Wrapped variables in double quotes for safety.
* Added a proper subject for the email alert.
* Improved message readability.

***

#### ✅ Cron Job Setup

To run this script every hour at the beginning of the hour, add the following entry to the crontab:

```
```

#### 📘 Cron Syntax Breakdown

```
*     *     *     *     *     command_to_execute
|     |     |     |     |
|     |     |     |     +----- Day of the week (0 - 6) (Sunday=0)
|     |     |     +----------- Month (1 - 12)
|     |     +----------------- Day of the month (1 - 31)
|     +----------------------- Hour (0 - 23)
+----------------------------- Minute (0 - 59)
```

#### 📌 Cron Examples

| Cron Expression | Description                           |
| --------------- | ------------------------------------- |
| `0 * * * *`     | Every hour at 0 minutes               |
| `0 7 22 * *`    | At 7:00 AM on the 22nd of every month |
| `0 5 * * 1`     | At 5:00 AM every Monday               |
| `5 * * * *`     | At 5 minutes past every hour          |
| `30 14 10 11 *` | At 2:30 PM on November 10th           |
| `0 */1 * * *`   | Every hour (same as `0 * * * *`)      |
