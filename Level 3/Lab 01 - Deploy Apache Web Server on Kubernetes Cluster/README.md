# Level 3 - Lab 01 - Deploy Apache Web Server on Kubernetes Cluster
## Task
1. Create a `namespace` named `httpd-namespace-datacenter`.

2. Create a `deployment` named `httpd-deployment-datacenter` under newly created namespace. For the deployment use `httpd` image with `latest` tag and remember to mention the tag i.e `httpd:latest`, and make sure replica count is `2`.

3. Create a `service` named `httpd-service-datacenter` under same namespace to expose the deployment, nodePort should be `30004`.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=ibo6E-FCc1E)

```
thor@jump_host ~$ kubectl create ns httpd-namespace-datacenter 
namespace/httpd-namespace-datacenter created 

thor@jump_host ~$ kubectl create deployment httpd-deployment-datacenter -n httpd-namespace-datacenter --image=httpd:latest --replicas=2 
deployment.apps/httpd-deployment-datacenter created 

thor@jump_host ~$ kubectl get deploy -n httpd-namespace-datacenter 
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE 
httpd-deployment-datacenter   2/2     2            2           19s 

thor@jump_host ~$ kubectl describe deploy httpd-deployment-datacenter -n httpd-namespace-datacenter 
Name:                   httpd-deployment-datacenter 
Namespace:              httpd-namespace-datacenter 
CreationTimestamp:      Wed, 22 Nov 2023 21:51:39 +0000 
Labels:                 app=httpd-deployment-datacenter 
Annotations:            deployment.kubernetes.io/revision: 1 
Selector:               app=httpd-deployment-datacenter 
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable 
StrategyType:           RollingUpdate 
MinReadySeconds:        0 
RollingUpdateStrategy:  25% max unavailable, 25% max surge 
Pod Template: 
  Labels:  app=httpd-deployment-datacenter 
  Containers: 
   httpd: 
    Image:        httpd:latest 
    Port:         <none> 
    Host Port:    <none> 
    Environment:  <none> 
    Mounts:       <none> 
  Volumes:        <none> 
Conditions: 
  Type           Status  Reason 
  ----           ------  ------ 
  Available      True    MinimumReplicasAvailable 
  Progressing    True    NewReplicaSetAvailable 
OldReplicaSets:  <none> 
NewReplicaSet:   httpd-deployment-datacenter-fcbb7f74 (2/2 replicas created) 
Events: 
  Type    Reason             Age   From                   Message 
  ----    ------             ----  ----                   ------- 
  Normal  ScalingReplicaSet  34s   deployment-controller  Scaled up replica set httpd-deployment-datacenter-fcbb7f74 to 2 

thor@jump_host ~$ kubectl expose deployment httpd-deployment-datacenter -n httpd-namespace-datacenter --name=httpd-service-datacenter --type="NodePort" --port=30004 
service/httpd-service-datacenter exposed 

thor@jump_host ~$ kubectl get svc -n httpd-namespace-datacenter 
NAME                       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)           AGE 
httpd-service-datacenter   NodePort   10.96.98.185   <none>        30004:31819/TCP   9s 

thor@jump_host ~$ kubectl edit svc httpd-service-datacenter -n httpd-namespace-datacenter 
service/httpd-service-datacenter edited 

(edit "nodePort: 30004" , "port: 80" , "targetPort: 80")
```