# Level 2 - Lab 07 - Deploy Grafana on Kubernetes Cluster 
## Task
1. Create a deployment named `grafana-deployment-nautilus` using any grafana image for Grafana app. Set other parameters as per your choice.

2. Create `NodePort` type service with nodePort `32000` to expose the app.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=9JOfAa5dQDM)

```
thor@jump_host ~$ kubectl create deployment grafana-deployment-nautilus --image=grafana/grafana --replicas=3 
deployment.apps/grafana-deployment-nautilus created 

thor@jump_host ~$ kubectl get deploy 
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE 
grafana-deployment-nautilus   3/3     3            3           24s 

thor@jump_host ~$ kubectl expose deployment grafana-deployment-nautilus --type="NodePort" --port=32000 
service/grafana-deployment-nautilus exposed 

thor@jump_host ~$ kubectl edit svc grafana-deployment-nautilus  
service/grafana-deployment-nautilus edited 

(edit "nodePort: 32000" , "port: 3000" , "targetPort: 3000", you can see here that grafana server on port 3000: https://hub.docker.com/r/grafana/grafana)

thor@jump_host ~$ kubectl get svc 
NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE 
grafana-deployment-nautilus   NodePort    10.96.158.153   <none>        3000:32000/TCP   2m35s 
kubernetes                    ClusterIP   10.96.0.1       <none>        443/TCP          48m 
```