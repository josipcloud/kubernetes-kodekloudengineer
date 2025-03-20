# Lab 09 - Countdown job in Kubernetes
## Task
1. Create a job `countdown-xfusion`.

2. The spec template should be named `countdown-xfusion` (under metadata), and the container should be named `container-countdown-xfusion`.

3. Use image `fedora` with `latest` tag and remember to mention tag i.e `fedora-latest`. Restart policy should be `Never`.

4. Use command `sleep 5`.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=DAi3BhX1MIQ)

```
thor@jump_host ~$ vi job.yaml 
thor@jump_host ~$ cat job.yaml  

apiVersion: batch/v1 
kind: Job 
metadata: 
  name: countdown-xfusion 
spec: 
  template: 
    metadata: 
      name: countdown-xfusion 
    spec: 
      containers: 
      - name: container-countdown-xfusion 
        image: fedora:latest 
        command: ["sleep",  "5"] 
      restartPolicy: Never 
  backoffLimit: 4 

thor@jump_host ~$ kubectl create -f job.yaml 
```