# Lab 08 - Create Cronjobs in Kubernetes
## Task
1. Create a cronjob named `nautilus`.

2. Set Its schedule to something like `* /5 * * * *`.

3. Container name should be `cron-nautilus`.

4. Use `httpd` image with `latest` tag and remember to mention the tag i.e `httpd:latest`.

5. Run a dummy command `echo Welcome to xfusioncorp!`

6. Ensure restart policy is `OnFailure`.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=H3Wn2-tfhac)

```
thor@jump_host ~$ vi cronjob.yaml 
thor@jump_host ~$ cat cronjob.yaml  

apiVersion: batch/v1 
kind: CronJob 
metadata: 
  name: nautilus 
spec: 
  schedule: "*/5 * * * *" 
  jobTemplate: 
    spec: 
      template: 
        spec: 
          containers: 
          - name: cron-nautilus 
            image: httpd:latest 
            imagePullPolicy: IfNotPresent 
            command: 
            - /bin/sh 
            - -c 
            - echo Welcome to xfusioncorp! 
          restartPolicy: OnFailure 

thor@jump_host ~$ kubrectl create -f cronjob.yaml 
```