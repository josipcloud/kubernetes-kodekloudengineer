# Level 2 - Lab 05 - Rolling Updates And Rolling Back Deployments in Kubernetes
## Task
1. Create a namespace `devops`. Create a deployment called `httpd-deploy` under this new namespace. It should have one container called `httpd`, use `httpd:2.4.28` image and `3` replicas. The deployment should use `RollingUpdate` strategy with `maxSurge=1` and `maxUnavailable=2`. Also create a `NodePort` type service named `httpd-service` and expose the deployment on `nodePort: 30008`.

2. Now upgrade the deployment to version `httpd:2.4.43` using a rolling update.

3. Finally, once all pods are updated undo the recent update and roll back to the previous/original version.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=Sg6P2e2tv3U)

```
thor@jump_host ~$ kubectl create ns devops 
namespace/devops created 

thor@jump_host ~$ kubectl create deployment httpd-deploy -n devops --image=httpd:2.4.28 --replicas=3 --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml  
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: httpd-deploy 
  name: httpd-deploy 
  namespace: devops 
spec: 
  replicas: 3 
  selector: 
    matchLabels: 
      app: httpd-deploy 
  strategy: 
    type: RollingUpdate 
    rollingUpdate: 
      maxUnavailable: 2 
      maxSurge: 1 
  template: 
    metadata: 
      labels: 
        app: httpd-deploy 
    spec: 
      containers: 
      - image: httpd:2.4.28 
        name: httpd 

thor@jump_host ~$ kubectl create -f deploy.yaml  
deployment.apps/httpd-deploy created 

thor@jump_host ~$ kubectl expose deployment httpd-deploy -n devops --name=httpd-service --type="NodePort" --port=30008 
service/httpd-service exposed 

thor@jump_host ~$ kubectl get svc -n devops 

NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)           AGE 
httpd-service   NodePort   10.96.29.32   <none>        30008:32460/TCP   4s 

thor@jump_host ~$ kubectl edit svc httpd-service -n devops 
service/httpd-service edited 

(edit "nodePort: 30008" , "port: 80" , "targetPort: 80")

thor@jump_host ~$ kubectl get svc -n devops 

NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE 
httpd-service   NodePort   10.96.29.32   <none>        80:30008/TCP   102s 

(Check access via web button, it should work now)

(Now perform Rolling update:)

thor@jump_host ~$ kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 --namespace=devops 
deployment.apps/httpd-deploy image updated 

(Now Rollback:)

thor@jump_host ~$ kubectl rollout history deployment/httpd-deploy --namespace=devops 
deployment.apps/httpd-deploy  

REVISION  CHANGE-CAUSE 
1         <none> 
2         <none> 

thor@jump_host ~$ kubectl rollout undo deployment/httpd-deploy --to-revision=1 --namespace=devops 
deployment.apps/httpd-deploy rolled back 
```