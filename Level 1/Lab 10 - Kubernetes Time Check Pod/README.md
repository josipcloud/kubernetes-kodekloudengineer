# Lab 10 - Kubernetes Time Check Pod
## Task
1. Create a pod called `time-check` in the `datacenter` namespace. This pod should run a container called `time-check`. Container should use `busybox` image with `latest` tag, and remember to mention tag i.e `busybox:latest`.

2. Create a config map called `time-config` with the data `TIME_FREQ=11` in the same namespace.

3. The time-check container should run the command `while true; do date; sleep $TIME_FREQ;done` and should write the result to the location `/opt/security/time/time-check.log`. Remember you will also need to add an environmental variable `TIME_FREQ` in the container, which should pick value from the config map `TIME_FREQ` key.

4. Create a volume `log-volume` and mount the same on `/opt/security/time` within the container.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=KQUF7z0oZw8)

```
(Useful link: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) 

thor@jump_host ~$ vi configmap.yaml 
thor@jump_host ~$ cat configmap.yaml 

apiVersion: v1 
kind: ConfigMap 
metadata: 
  namespace: datacenter 
  name: time-config 
data: 
  TIME_FREQ: "11"  

thor@jump_host ~$ vi pod.yaml
thor@jump_host ~$ cat pod.yaml 

apiVersion: v1 
kind: Pod 
metadata: 
  name: time-check 
  namespace: xfusion 
spec: 
  containers: 
    - name: time-check 
      image: busybox:latest 
      command: 
        - "sh" 
        - "-c" 
        - "while true; do date; sleep $TIME_FREQ;done >> /opt/sysops/time/time-check.log" 
      env: 
        - name: TIME_FREQ 
          valueFrom: 
            configMapKeyRef: 
              name: time-config 
              key: TIME_FREQ 
      volumeMounts: 
      - mountPath: /opt/sysops/time 
        name: log-volume 
  volumes: 
  - name: log-volume 
    emptyDir: 
      sizeLimit: 100Mi 

thor@jump_host ~$ kubeclt create -f configmap.yaml
thor@jump_host ~$ kubeclt create -f pod.yaml

thor@jump_host ~$ kubeclt exec -it time-check -n datacenter -- sh 
/ #  
/ # cd /opt/security/time/ 
/opt/security/time # ls 
time-check.log 
/opt/security/time # cat time-check.log  
Fri Oct 27 21:48:34 UTC 2023 
Fri Oct 27 21:48:45 UTC 2023 
Fri Oct 27 21:48:56 UTC 2023 


```