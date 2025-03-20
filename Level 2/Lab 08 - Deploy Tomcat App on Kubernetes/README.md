# Level 2 - Lab 08 - Deploy Tomcat App on Kubernetes
## Task
1. Create a namespace named `tomcat-namespace-nautilus`.

2. Create a `deployment` for tomcat app which should be named `tomcat-deployment-nautilus` under the same namespace you created. Replica count should be `1`, the container should be named `tomcat-container-nautilus`, its image should be `gcr.io/kodekloud/centos-ssh-enabled:tomcat` and its container port should be `8080`.

3. Create a `service` for tomcat app which should be named `tomcat-service-nautilus` under the same namespace you created. Service type should be `NodePort` and nodePort should be `32227`.

Before clicking on `Check` button please make sure the application is up and running.

`You can use any labels as per your choice.`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=KyTX34AjxOU)

```
thor@jump_host ~$ kubectl create ns tomcat-namespace-nautilus 
namespace/tomcat-namespace-nautilus created 

thor@jump_host ~$ kubectl create deploy tomcat-deployment-nautilus -n tomcat-namespace-nautilus --image=gcr.io/kodekloud/centos-ssh-enabled:tomcat --replicas=1 --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml  
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: tomcat-deployment-nautilus 
  name: tomcat-deployment-nautilus 
  namespace: tomcat-namespace-nautilus 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: tomcat-deployment-nautilus 
  strategy: {} 
  template: 
    metadata: 
      labels: 
        app: tomcat-deployment-nautilus 
    spec: 
      containers: 
      - image: gcr.io/kodekloud/centos-ssh-enabled:tomcat 
        name: tomcat-container-nautilus 
        ports: 
        - containerPort: 8080 

thor@jump_host ~$ kubectl create -f deploy.yaml  
deployment.apps/tomcat-deployment-nautilus created 

thor@jump_host ~$ kubectl expose deployment tomcat-deployment-nautilus -n tomcat-namespace-nautilus --name=tomcat-service-nautilus --type="NodePort" --port=32227 
service/tomcat-service-nautilus exposed 

thor@jump_host ~$ kubectl get svc -A 
NAMESPACE                   NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE 
default                     kubernetes                ClusterIP   10.96.0.1       <none>        443/TCP                  65m 
kube-system                 kube-dns                  ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   65m 
tomcat-namespace-nautilus   tomcat-service-nautilus   NodePort    10.96.192.205   <none>        32227:31451/TCP          5s 

thor@jump_host ~$ kubectl edit svc tomcat-service-nautilus -n tomcat-namespace-nautilus 
service/tomcat-service-nautilus edited

(edit "nodePort: 32227" , "port: 8080" , "targetPort: 8080")

(Check access via web button, it should work now)
```