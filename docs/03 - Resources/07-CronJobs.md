# CronJobs

One CronJob object is like one line of a crontab (cron table) file on a Unix system. It runs a Job periodically on a given schedule, written in Cron format, performing regular scheduled actions such as backups, report generation, and so on. 

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"  # minute (0-60) hour (0-60) day-of-month (1-31) month (1-12) day0f-the-week (0-6)
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
  concurrencyPolicy: Allow
  startingDeadlineSeconds: 60
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```
# Concurrency Policy

- **Allow** (default): The CronJob allows concurrently running Jobs 
- **Forbid** : The CronJob does not allow concurrent runs; if it is time for a new Job run and the previous Job run hasn't finished yet, the CronJob skips the new Job run.
- **Replace**: If it is time for a new Job run and the previous Job run hasn't finished yet, the CronJob replaces the currently running Job run with a new Job run


> [!NOTE]
> Specifying a timezone using CRON_TZ or TZ variables inside .spec.schedule is not officially supported (and never has been).
> CronJob contains a template for new Jobs. If you modify an existing CronJob, the changes you make will apply to new Jobs that start to run after your modification is complete.
