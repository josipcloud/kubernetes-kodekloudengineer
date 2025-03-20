# Level 4 - Lab 01 - Deploy Redis Deployment on Kubernetes
## Task

Create a redis deployment with following parameters:

1. Create a `config map` called `my-redis-config` having `maxmemory 2mb` in `redis-config`.

2. Name of the `deployment` should be `redis-deployment`, it should use `redis:alpine` image and container name should be `redis-container`. Also make sure it has only `1` replica.

3. The container should request for `1` CPU.

4. Mount `2` volumes:

   a. An Empty directory volume called `data` at path `/redis-master-data`.

   b. A configmap volume called `redis-config` at path `/redis-master`.

   c. The container should expose the port `6379`.

5. Finally, `redis-deployment` should be in an up and running state.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=vKBbQLUnddo)

```
thor@jump_host ~$ vi cm.yaml 
thor@jump_host ~$ cat cm.yaml  

kind: ConfigMap 
apiVersion: v1 
metadata: 
  name: my-redis-config 
data: 
  maxmemory: 2mb 

thor@jump_host ~$ kubectl create -f cm.yaml 
configmap/my-redis-config created 

thor@jump_host ~$ kubectl describe cm my-redis-config  

Name:         my-redis-config 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 

Data 
==== 
maxmemory: 
---- 
2mb 
 
BinaryData 
==== 

Events:  <none> 

thor@jump_host ~$ kubectl create deploy redis-deployment --replicas=1 --image=redis:alpine --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    app: redis-deployment 
  name: redis-deployment 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: redis-deployment 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: redis-deployment 
    spec: 
      containers: 
      - image: redis:alpine 
        name: redis-container 
        ports: 
        - containerPort: 6379 
        resources: 
          requests: 
            cpu: "1" 
        volumeMounts: 
          - name: data 
            mountPath: /redis-master-data 
          - name: redis-config 
            mountPath: /redis-master 
      volumes: 
        - name: data 
          emptyDir: 
            sizeLimit: 100Mi 
        - name: redis-config 
          configMap: 
            name: my-redis-config 
```