# Level 4 - Lab 04 - Deploy Drupal App on Kubernetes
## Task
1. Configure a persistent volume `drupal-mysql-pv` with `hostPath = /drupal-mysql-data` (`/drupal-mysql-data` directory already exists on the worker Node i.e jump host). `5Gi` of storage and `ReadWriteOnce` access mode.

2. Configure one PersistentVolumeClaim named `drupal-mysql-pvc` with storage request of `3Gi` and `ReadWriteOnce` access mode.

3. Create a deployment `drupal-mysql` with `1` replica, use `mysql:5.7` image. Mount the claimed PVC at `/var/lib/mysql`.

4. Create a deployment `drupal` with `1` replica and use `drupal:8.6` image.

5. Create a `NodePort` type service which should be named `drupal-service` and nodePort should be `30095`.

6. Create a service `drupal-mysql-service` to expose mysql deployment on port `3306`.

7. Set rest of the configuration for deployments, services, secrets etc as per your preferences. Ath the end you should be able to access the Drupal installation page by clicking on `App` button.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=nmtqlZF-Gqk)


```
  
apiVersion: v1 
kind: PersistentVolume 
metadata: 
  name: drupal-mysql-pv 
spec: 
  capacity: 
    storage: 5Gi 
  volumeMode: Filesystem 
  accessModes: 
    - ReadWriteOnce 
  persistentVolumeReclaimPolicy: Recycle 
  storageClassName: slow 
  hostPath: 
    path: /drupal-mysql-data 
--- 
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: drupal-mysql-pvc 
spec: 
  accessModes: 
    - ReadWriteOnce 
  volumeMode: Filesystem 
  resources: 
    requests: 
      storage: 3Gi 
  storageClassName: slow 
--- 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: drupal-mysql 
  labels: 
    app: drupal-mysql 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: drupal-mysql 
  template: 
    metadata: 
      name: drupal-mysql 
      labels: 
        app: drupal-mysql 
    spec: 
      containers: 
        - name: myslq-drupal-container 
          image: mysql:5.7 
          env: 
          - name: MYSQL_DATABASE 
            value: "drupal_databse" 
          - name: MYSQL_ROOT_PASSWORD 
            value: "R00t" 
          - name: MYSQL_PASSWORD 
            value: "MysqlPassword123@" 
          - name: MYSQL_USER  
            value: admin    
          volumeMounts: 
            - mountPath: /var/lib/mysql 
              name: drupal-mysql-pvc-volume 
      volumes: 
        - name: drupal-mysql-pvc-volume 
          persistentVolumeClaim: 
            claimName: drupal-mysql-pvc 
--- 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: drupal 
  labels: 
    app: drupal 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: drupal 
  template: 
    metadata: 
      name: drupal 
      labels: 
        app: drupal 
    spec: 
      containers: 
        - name: drupal-container 
          image: drupal:8.6 
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: drupal-service 
spec: 
  selector: 
    app: drupal 
  ports: 
    - protocol: TCP 
      port: 80 
      targetPort: 80 
      nodePort: 30095 
  type: NodePort 
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: drupal-mysql-service 
spec: 
  selector: 
    app: drupal-mysql 
  ports: 
    - protocol: TCP 
      port: 3306 
      targetPort: 3306 
  type: ClusterIP 

thor@jump_host ~$ kubectl create -f depl.yaml 

```


```

thor@jump_host ~$ kubectl get svc 
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
drupal-mysql-service   ClusterIP   10.96.174.183   <none>        3306/TCP       2m3s
drupal-service         NodePort    10.96.193.203   <none>        80:30095/TCP   2m3s 
kubernetes             ClusterIP   10.96.0.1       <none>        443/TCP        40m 

thor@jump_host ~$ kubectl describe deployments.apps  

Name:                   drupal 
Namespace:              default 
CreationTimestamp:      Tue, 05 Dec 2023 13:12:20 +0000 
Labels:                 app=drupal 
Annotations:            deployment.kubernetes.io/revision: 1 
Selector:               app=drupal 
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate 
MinReadySeconds:        0 
RollingUpdateStrategy:  25% max unavailable, 25% max surge 
Pod Template: 
  Labels:  app=drupal 
  Containers: 
   drupal-container: 
    Image:        drupal:8.6 
    Port:         <none> 
    Host Port:    <none> 
    Environment:  <none> 
    Mounts:       <none> 
  Volumes:        <none> 
Conditions: 
  Type           Status  Reason 
  ----           ------  ------ 
  Available      True    MinimumReplicasAvailable 
  Progressing    True    NewReplicaSetAvailable 
OldReplicaSets:  <none> 
NewReplicaSet:   drupal-864ddb8f8c (1/1 replicas created) 
Events: 
  Type    Reason             Age    From                   Message 
  ----    ------             ----   ----                   ------- 
  Normal  ScalingReplicaSet  2m21s  deployment-controller  Scaled up replica set drupal-864ddb8f8c to 1 
  
Name:                   drupal-mysql 
Namespace:              default 
CreationTimestamp:      Tue, 05 Dec 2023 13:12:19 +0000 
Labels:                 app=drupal-mysql 
Annotations:            deployment.kubernetes.io/revision: 1 
Selector:               app=drupal-mysql 
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable 
StrategyType:           RollingUpdate 
MinReadySeconds:        0 
RollingUpdateStrategy:  25% max unavailable, 25% max surge 
Pod Template: 
  Labels:  app=drupal-mysql 
  Containers: 
   myslq-drupal-container: 
    Image:      mysql:5.7 
    Port:       <none> 
    Host Port:  <none> 
    Environment: 
      MYSQL_DATABASE:       drupal_databse 
      MYSQL_ROOT_PASSWORD:  R00t 
      MYSQL_PASSWORD:       MysqlPassword123@ 
      MYSQL_USER:           admin 
    Mounts: 
      /var/lib/mysql from drupal-mysql-pvc-volume (rw) 
  Volumes: 
   drupal-mysql-pvc-volume: 
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace) 
    ClaimName:  drupal-mysql-pvc 
    ReadOnly:   false 
Conditions: 
  Type           Status  Reason 
  ----           ------  ------ 
  Available      True    MinimumReplicasAvailable 
  Progressing    True    NewReplicaSetAvailable 
OldReplicaSets:  <none> 
NewReplicaSet:   drupal-mysql-6657bc7cb4 (1/1 replicas created) 
Events: 
  Type    Reason             Age    From                   Message 
  ----    ------             ----   ----                   ------- 
  Normal  ScalingReplicaSet  2m21s  deployment-controller  Scaled up replica set drupal-mysql-6657bc7cb4 to 1 
```