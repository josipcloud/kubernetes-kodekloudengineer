# Level 3 - Lab 10 - Fix Python App Deployed on Kubernetes Cluster
## Task

Fix the issues:
1. The deployment name is `python-deployment-xfusion`, it is using `poroko/flask-demo-app` image. The deployment and service of this app is already deployed.

2. `nodePort` should be `32345` and targetPort should be python flask app's default port.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=VXAF9ebKMwA)


```
thor@jump_host ~$ kubectl get pods
python-deployment-xfusion   0/1     1            0           48s 

thor@jump_host ~$ kubectl describe deploy python-deployment-xfusion  
Name:                   python-deployment-xfusion 
Namespace:              default 
CreationTimestamp:      Sat, 30 Dec 2023 22:37:31 +0000 
Labels:                 <none> 
Annotations:            deployment.kubernetes.io/revision: 1 
Selector:               app=python_app 
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable 
StrategyType:           RollingUpdate 
MinReadySeconds:        0 
RollingUpdateStrategy:  25% max unavailable, 25% max surge 
Pod Template: 
  Labels:  app=python_app 
  Containers: 
   python-container-xfusion: 
    Image:        poroko/flask-app-demo
    Port:         5000/TCP 
    Host Port:    0/TCP 
    Environment:  <none> 
    Mounts:       <none> 
  Volumes:        <none> 
Conditions: 
  Type           Status  Reason 
  ----           ------  ------ 
  Available      False   MinimumReplicasUnavailable 
  Progressing    True    ReplicaSetUpdated 
OldReplicaSets:  <none> 
NewReplicaSet:   python-deployment-xfusion-5dcd4fff84 (1/1 replicas created) 
Events: 
  Type    Reason             Age   From                   Message 
  ----    ------             ----  ----                   ------- 
  Normal  ScalingReplicaSet  58s   deployment-controller  Scaled up replica set python-deployment-xfusion-5dcd4fff84 to 1 
```
The image is wrong, it is set to `poroko/flask-app-demo` instead to `poroko/flask-demo-app`. Fix it:
```
thor@jump_host ~$ kubectl edit deploy python-deployment-xfusion  

(change image to poroko/flask-demo-app)
```
Next, troubleshoot service:

```
thor@jump_host ~$ kubectl get svc 
NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE 
kubernetes               ClusterIP   10.96.0.1     <none>        443/TCP          123m 
python-service-xfusion   NodePort    10.96.65.92   <none>        8080:32345/TCP   2m54s 
```
There is an error here, it the port should be changed from port `8080` to port `5000`, because that is the flak app default port.
```
thor@jump_host ~$ kubectl edit svc python-service-xfusion
thor@jump_host ~$ kubectl get svc 
NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE 
kubernetes               ClusterIP   10.96.0.1     <none>        443/TCP          123m 
python-service-xfusion   NodePort    10.96.65.92   <none>        5000:32345/TCP   10s 
```
Now check access via Web Button, it should work now.