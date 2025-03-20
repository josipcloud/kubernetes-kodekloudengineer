# Lab 01 - Create Pods in Kubernetes Cluster
## Task
1. Create a pod named `pod-nginx` using `nginx` image with `latest` tag only and remember to mention the tag i.e `nginx:latest`
2. Labels app should be set to `nginx_app`, also container should be named as `nginx-container`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=4hflN_fmyAs)

```
thor@jump_host ~$ kubectl run pod-nginx --image=nginx:latest --dry-run=client -o yaml > pod.yaml
thor@jump_host ~$ vi pod.yaml 
thor@jump_host ~$ cat pod.yaml
apiVersion: v1 
kind: Pod 
metadata: 
  creationTimestamp: null 
  labels: 
    app: nginx_app 
  name: pod-nginx 
spec: 
  containers: 
  - image: nginx:latest 
    name: nginx-container 
    resources: {} 
  dnsPolicy: ClusterFirst 
  restartPolicy: Always 
status: {} 

thor@jump_host ~$ kubectl create -f pod.yaml 
pod/pod-nginx created 

thor@jump_host ~$ kubectl get pods 

NAME        READY   STATUS              RESTARTS   AGE 
pod-nginx   0/1     ContainerCreating   0          5s 

thor@jump_host ~$ kubectl get pods 

NAME        READY   STATUS    RESTARTS   AGE 
pod-nginx   1/1     Running   0          12s 

thor@jump_host ~$ kubectl describe pod pod-nginx 

Name:             pod-nginx 
Namespace:        default 
Priority:         0 
Service Account:  default 
Node:             kodekloud-control-plane/172.17.0.2 
Start Time:       Wed, 15 Nov 2023 22:09:48 +0000 
Labels:           app=nginx_app 
Annotations:      <none> 
Status:           Running 
IP:               10.244.0.5 
IPs: 
  IP:  10.244.0.5 
Containers: 
  nginx-container: 
    Container ID:   containerd://5711c3b39ea389e234e7c272c05da82be9262970b175f3b9c452bd0253a2d49a 
    Image:          nginx:latest 
    Image ID:       docker.io/library/nginx@sha256:86e53c4c16a6a276b204b0fd3a8143d86547c967dc8258b3d47c3a21bb68d3c6 
    Port:           <none> 
    Host Port:      <none> 
    State:          Running 
      Started:      Wed, 15 Nov 2023 22:09:57 +0000 
    Ready:          True 
    Restart Count:  0 
    Environment:    <none> 
    Mounts: 
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-78rk7 (ro) 
Conditions: 
  Type              Status 
  Initialized       True  
  Ready             True  
  ContainersReady   True  
  PodScheduled      True  
Volumes: 
  kube-api-access-78rk7: 
    Type:                    Projected (a volume that contains injected data from multiple sources) 
    TokenExpirationSeconds:  3607 
    ConfigMapName:           kube-root-ca.crt 
    ConfigMapOptional:       <nil> 
    DownwardAPI:             true 
QoS Class:                   BestEffort 
Node-Selectors:              <none> 
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s 
                        node.kubernetes.io/unreachable:NoExecute op=Exists for 300s 
Events: 
  Type    Reason     Age   From               Message 
  ----    ------     ----  ----               ------- 
  Normal  Scheduled  23s   default-scheduler  Successfully assigned default/pod-nginx to kodekloud-control-plane 
  Normal  Pulling    22s   kubelet            Pulling image "nginx:latest" 
  Normal  Pulled     14s   kubelet            Successfully pulled image "nginx:latest" in 7.63905111s (7.639066712s including waiting) 
  Normal  Created    14s   kubelet            Created container nginx-container 
  Normal  Started    14s   kubelet            Started container nginx-container 