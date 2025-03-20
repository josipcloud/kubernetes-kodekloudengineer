# Lab 07 - Create Replicaset in Kubernetes Cluster
## Task
1. Create a ReplicaSet using `nginx` image with `latest` tag and remember to mention tag i.e `nginx:latest` and name it `nginx-replicaset`.

2. Labels `app` should be `nginx_app`, labels `type` should be `front-end`.

3. The container should be named `nginx-container` and replicas count should be `4`. 


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=R-ruawscwDg)

```
thor@jump_host ~$ kubectl create deployment nginx-replicaset --image=nginx:latest --replicas=4 --dry-run=client -o yaml > replicaset.yaml 

thor@jump_host ~$ vi replicaset.yaml 

thor@jump_host ~$ cat replicaset.yaml 

apiVersion: apps/v1 
kind: ReplicaSet 
metadata: 
  creationTimestamp: null 
  labels: 
    app: nginx_app 
    type: front-end 
  name: nginx-replicaset 
spec: 
  replicas: 4 
  selector: 
    matchLabels: 
      app: nginx_app 
      type: front-end 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: nginx_app 
        type: front-end 
    spec: 
      containers: 
      - image: nginx:latest 
        name: nginx-container 
        resources: {} 
status: {} 

```