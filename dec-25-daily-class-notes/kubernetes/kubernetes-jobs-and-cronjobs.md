# Kubernetes Jobs and CronJobs

### Jobs

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

```yaml
apiVersion: batch/v1          ## The version of the Kubernetes API
kind: Job                     ## The type of object for jobs
metadata:
 name: job-test
spec:                        ## What state you desire for the object
 template:
   metadata:
     name: job-test
   spec:
     containers:
     - name: job
       image: busybox                  ##  Image used
       command: ["echo", "job-test"]   ##  Command used to create logs for verification later
     restartPolicy: OnFailure          ##  Restart Policy in case container failed
```

### CronJob

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```

| Entry                  | Description                                                | Equivalent to |
| ---------------------- | ---------------------------------------------------------- | ------------- |
| @yearly (or @annually) | Run once a year at midnight of 1 January                   | 0 0 1 1 \*    |
| @monthly               | Run once a month at midnight of the first day of the month | 0 0 1 \* \*   |
| @weekly                | Run once a week at midnight on Sunday morning              | 0 0 \* \* 0   |
| @daily (or @midnight)  | Run once a day at midnight                                 | 0 0 \* \* \*  |
| @hourly                | Run once an hour at the beginning of the hour              | 0 \* \* \* \* |
