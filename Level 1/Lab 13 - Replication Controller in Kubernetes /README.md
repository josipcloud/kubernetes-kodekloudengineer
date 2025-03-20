# Lab 13 - Replication Controller in Kubernetes
## Task
1. Create a `ReplicationController` using `httpd` image, preferably with `latest` tag, and name it `httpd-replicationcontroller`.

2. Labels `app` should be `httpd_app` and labels `type` should be `front-end`. The container should be named `httpd-container` and also make sure replica count is set to `3`. 

All pods should be in running state after deployment.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=P4xKA1K56AE)

```
thor@jump_host ~$ vi replicationcontroller.yaml
thor@jump_host ~$ cat replicationcontroller.yaml

apiVersion: v1 
kind: ReplicationController 
metadata: 
  name: httpd-replicationcontroller 
spec: 
  replicas: 3 
  selector: 
    app: httpd_app 
    type: front-end 
  template: 
    metadata: 
      name: httpd 
      labels: 
        app: httpd_app 
        type: front-end 
    spec: 
      containers: 
      - name: httpd-container 
        image: httpd:latest 

thor@jump_host ~$ kubectl create -f replicationcontroller.yaml
thor@jump_host ~$ kubectl get rc
```