# Level 2 - Lab 06 - Deploy Jenkins to Kubernetes
## Task
1. Create a namespace `jenkins`.

2. Create a service for Jenkins deployment. Service name should be `jenkins-service` under `jenkins` namespace, type should be `NodePort`. NodePort should be `30008`.

3. Create a Jenkins deployment under `jenkins` namespace, it should be named `jenkins-deployment`, labels `app` should be `jenkins`, container name should be `jenkins-container`, use `jenkins/jenkins` image, containerPort should be `8080` and replicas count should be `1`.

Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins login screen in the browser before hitting the `Check` button.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=wwM69A8JP0s)

```
thor@jump_host ~$ kubectl create ns jenkins 
namespace/jenkins created 

thor@jump_host ~$ kubectl create deployment jenkins-deployment -n jenkins --replicas=1 --image=jenkins/jenkins --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml  
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: jenkins 
  name: jenkins-deployment 
  namespace: jenkins 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: jenkins 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: jenkins 
    spec: 
      containers: 
      - image: jenkins/jenkins 
        name: jenkins-container 
        ports: 
        - containerPort: 8080 

thor@jump_host ~$ kubectl expose deployment jenkins-deployment -n jenkins --name=jenkins-service --type="NodePort" --port=30008 
service/jenkins-service exposed 

thor@jump_host ~$ kubectl get svc -n jenkins 

NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE 
jenkins-service   NodePort   10.96.103.145   <none>        30008:31015/TCP   5s 

thor@jump_host ~$ kubectl edit svc jenkins-service -n jenkins 
service/jenkins-service edited 

(edit "nodePort: 30008" , "port: 8080" , "targetPort: 8080")

thor@jump_host ~$ kubectl get svc -n jenkins 

NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE 
jenkins-service   NodePort   10.96.103.145   <none>        8080:30008/TCP   41s 

(Check access via web button, it should work now)
```