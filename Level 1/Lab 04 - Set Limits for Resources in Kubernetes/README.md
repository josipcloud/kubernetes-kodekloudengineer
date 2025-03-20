# Lab 04 - Set Limits for Resources in Kubernetes
## Task
Create a pod named `httpd-pod` and a container under it named `httpd-container`, use `httpd` image with `latest` tag and remember to mention tag i.e `httpd:latest` and set the following limits:

Requests: Memory: `15Mi`, CPU: `100m`

Limits: Memory: `20Mi`, CPU: `100m`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=gxrZaF1Uutg)

```
thor@jump_host ~$ kubectl run httpd-pod --image=httpd:latest --dry-run=client -o yaml > pod.yaml 
thor@jump_host ~$ vi pod.yaml 
thor@jump_host ~$ cat pod.yaml 
apiVersion: v1 
kind: Pod 
metadata: 
  creationTimestamp: null 
  labels: 
    run: httpd-pod 
  name: httpd-pod 
spec: 
  containers: 
  - image: httpd:latest 
    name: httpd-container 
    resources: 
      requests: 
        memory: "15Mi" 
        cpu: "100m" 
      limits: 
        memory: "20Mi" 
        cpu: "100m" 
  dnsPolicy: ClusterFirst 
  restartPolicy: Always 
status: {} 

thor@jump_host ~$ kubectl create -f pod.yaml 
pod/httpd-pod created 

thor@jump_host ~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE 
httpd-pod   1/1     Running   0          12s 

thor@jump_host ~$ kubectl describe pod httpd-pod 
Name:             httpd-pod 
Namespace:        default 
Priority:         0 
Service Account:  default 
Node:             kodekloud-control-plane/172.17.0.2 
Start Time:       Fri, 17 Nov 2023 15:36:23 +0000 
Labels:           run=httpd-pod 
Annotations:      <none> 
Status:           Running 
IP:               10.244.0.5 
IPs: 
  IP:  10.244.0.5 
Containers: 
  httpd-container: 
    Container ID:   containerd://3b781ac4dc0b519ce5e0722131e698449d64cb5d40bfc7ffc4588aca9ca1779a 
    Image:          httpd:latest 
    Image ID:       docker.io/library/httpd@sha256:4e24356b4b0aa7a961e7dfb9e1e5025ca3874c532fa5d999f13f8fc33c09d1b7 
    Port:           <none> 
    Host Port:      <none> 
    State:          Running 
      Started:      Fri, 17 Nov 2023 15:36:33 +0000 
    Ready:          True 
    Restart Count:  0 
    Limits: 
      cpu:     100m 
      memory:  20Mi 
    Requests: 
      cpu:        100m 
      memory:     15Mi 
    Environment:  <none> 
    Mounts: 
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7p2dh (ro) 
Conditions: 
  Type              Status 
  Initialized       True  
  Ready             True  
  ContainersReady   True  
  PodScheduled      True  
Volumes: 
  kube-api-access-7p2dh: 
    Type:                    Projected (a volume that contains injected data from multiple sources) 
    TokenExpirationSeconds:  3607 
    ConfigMapName:           kube-root-ca.crt 
    ConfigMapOptional:       <nil> 
    DownwardAPI:             true 
QoS Class:                   Burstable 
Node-Selectors:              <none> 
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s 
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s 
Events: 
  Type    Reason     Age   From               Message 
  ----    ------     ----  ----               ------- 
  Normal  Scheduled  18s   default-scheduler  Successfully assigned default/httpd-pod to kodekloud-control-plane 
  Normal  Pulling    16s   kubelet            Pulling image "httpd:latest" 
  Normal  Pulled     9s    kubelet            Successfully pulled image "httpd:latest" in 7.442453722s (7.442469762s including waiting) 
  Normal  Created    9s    kubelet            Created container httpd-container 
  Normal  Started    8s    kubelet            Started container httpd-container 

```