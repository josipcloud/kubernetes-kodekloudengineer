# Level 2 - Lab 03 - Deploy Nginx Web Server on Kubernetes Cluster
## Task
1. Create a deployment using `nginx` image with `latest` tag and remember to mention the tag i.e `nginx:latest`. Name it `nginx-deployment`. The container should be named `nginx-container`. Also make sure replica count is `3`.

2. Create a `NodePort` type service named `nginx-service`. The nodePort should be `30011`.



## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=sxIc0uNqg4s)

```
thor@jump_host ~$ kubectl create deployment nginx-deployment --image=nginx:latest --replicas=3 --dry-run=client -o yaml > deployment.yaml 
thor@jump_host ~$ vi deployment.yaml 
thor@jump_host ~$ cat deployment.yaml

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: nginx-deployment 
  name: nginx-deployment 
spec: 
  replicas: 3 
  selector: 
    matchLabels: 
      app: nginx-deployment 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: nginx-deployment 
    spec: 
      containers: 
      - image: nginx:latest 
        name: nginx-container 
        resources: {} 
status: {}

thor@jump_host ~$ kubectl expose deployment nginx-deployment --name=nginx-service --type="NodePort" --port 30011 
service/nginx-service exposed 

thor@jump_host ~$ kubectl get svc 

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE 
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP           45m 
nginx-service   NodePort    10.96.208.125   <none>        30011:30851/TCP   3s 

thor@jump_host ~$ kubectl edit svc nginx-service 
service/nginx-service edited 

(change nodePort to 30011, port to 80, targetPort to 80)

thor@jump_host ~$ kubectl get svc 

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        46m 
nginx-service   NodePort    10.96.208.125   <none>        80:30011/TCP   94s 
```