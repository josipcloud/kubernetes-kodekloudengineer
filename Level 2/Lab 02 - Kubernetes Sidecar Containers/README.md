# Level 2 - Lab 02 - Kubernetes Sidecar Containers
## Task
1. Create a pod named `webserver`.

2. Create an `emptyDir` volume `shared-logs`.

3. Create two containers from `nginx` and `ubuntu` images with `latest` tag and remember to mention tag i.e `nginx:latest`. Nginx container name should be `nginx-container` and ubuntu container name should be `sidecar-container` on webserver pod.

4. Add command on sidecar-container `"sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"`.

5. Mount the volume `shared-logs` on both containers at location `/var/log/nginx`. All containers should be up and running.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=1y6ZDVDIWzM)

```
thor@jump_host ~$ kubectl run webserver --image=ubuntu:latest --dry-run=client -o yaml > webserver.yaml 
thor@jump_host ~$ vi webserver.yaml  
thor@jump_host ~$ cat webserver.yaml  

apiVersion: v1 
kind: Pod 
metadata: 
  name: webserver 
spec: 
  containers: 
  - image: nginx:latest 
    name: nginx-container 
    volumeMounts: 
    - name: shared-logs 
      mountPath: /var/log/nginx 
  - image: ubuntu:latest 
    name: sidecar-container 
    command:  
    - "sh" 
    - "-c" 
    - "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done" 
    volumeMounts: 
    - name: shared-logs 
      mountPath: /var/log/nginx 
  volumes: 
  - name: shared-logs 
    emptyDir: 
      sizeLimit: 100Mi 

thor@jump_host ~$ kubectl create -f webserver.yaml  
pod/webserver created 
```