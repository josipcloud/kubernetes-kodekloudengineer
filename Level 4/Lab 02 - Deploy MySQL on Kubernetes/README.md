# Level 4 - Lab 02 - Deploy My SQL on Kubernetes
## Task
1. Create a PersistentVolume `mysql-pv`, its capacity should be `250Mi`, set other parameters as per your preference.

2. Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as `mysql-pv-claim` and request a `250Mi` of storage. Set other parameters as per your preference.

3. Create a deployment named `mysql-deployment`, use any mysql image as per your preference. Mount the PersistentVolume at mount path `/var/lib/mysql`.

4. Create a `NodePort` type service named `mysql` and set nodePort to `30007`.

5. Create a secret named `mysql-root-pass` having a key pair value, where key is `password` and its value is `YUIidhb667`, create another secret named `mysql-user-pass` having some key pair values, where first key is `username` and its value is `kodekloud_pop`, second key is `password` and value is `B4zNgHA7Ya`, create one more secret named `mysql-db-url`, key name is `database` and value is `kodekloud_db5`.

6. Define some Environment variables within the container:

   a) `name: MYSQL_ROOT_PASSWORD`, should pick value from secretKeyRef `name: mysql-root-pass` and `key: password`.

   b) `name: MYSQL_DATABASE`, should pick value from secretKeyRef `name: mysql-db-url` and `key:database`.

   c) `name: MYSQL_USER`, should pick value from secretKeyRef `name: mysql-user-pass` and `key: username`.

   d) `name: MYSQL_PASSWORD`, should pick value from secretKeyRef `name: mysql-user-pass` and `key: password`. 

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=6T9NQB7z96o)

Task 1:

```
thor@jump_host ~$ kubectl get sc 
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE 
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  39m 

thor@jump_host ~$ vi pv.yaml 
thor@jump_host ~$ cat pv.yaml 

apiVersion: v1 
kind: PersistentVolume 
metadata: 
  name: mysql-pv 
spec: 
  capacity: 
    storage: 250Mi 
  volumeMode: Filesystem 
  accessModes: 
    - ReadWriteOnce 
  storageClassName: standard 
  hostPath: 
    path: /var/lib/mysql 

thor@jump_host ~$ kubectl create -f pv.yaml 
persistentvolume/mysql-pv created 
```

Task 2:

```
thor@jump_host ~$ vi pvc.yaml 
thor@jump_host ~$ cat pvc.yaml 

apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:                           
  name: mysql-pv-claim 
spec:                               
  storageClassName: standard        
  accessModes: 
    - ReadWriteOnce                 
  resources: 
    requests:  
      storage: 250Mi 

thor@jump_host ~$ kubectl create -f pvc.yaml 
```

Task 3 - but without secrets and variables so far:

```
thor@jump_host ~$ kubectl create deploy mysql-deployment --image=mysql:latest --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml 
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    app: mysql-deployment 
  name: mysql-deployment 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: mysql-deployment 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: mysql-deployment 
    spec: 
      containers: 
      - image: mysql:latest 
        name: mysql 
        volumeMounts: 
        - name: mysql-pv-claim 
          mountPath: /var/lib/mysql 
      volumes:                        
      - name: mysql-pv-claim 
        persistentVolumeClaim: 
          claimName: mysql-pv-claim 

thor@jump_host ~$ kubectl create -f deploy.yaml  
deployment.apps/mysql-deployment created 

 
(credentials are missing still so there will be errors you can see with "kubectl logs <pod-name>

```

Task 4:

```
thor@jump_host ~$ kubectl expose deploy mysql-deployment --type=NodePort --name=mysql --port=8080 
service/mysql exposed 
thor@jump_host ~$ kubectl edit svc mysql 

(edit "nodePort: 30007", "port: 8080", "targetPort: 8080")
service/mysql edited 

thor@jump_host ~$ kubectl get svc 
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE 
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          46m 
mysql        NodePort    10.96.142.87   <none>        8080:30007/TCP   56s 
```

Task 5:

```
thor@jump_host ~$ kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667 
secret/mysql-root-pass created 

thor@jump_host ~$ kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_top --from-literal=password=YchZHRcLkL 
secret/mysql-user-pass created 

thor@jump_host ~$ kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db8 
secret/mysql-db-url created 
```

Task 6:

```
thor@jump_host ~$ kubectl delete -f deploy.yaml
thor@jump_host ~$ vi deploy.yaml
thor@jump_host ~$ cat deploy.yaml

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    app: mysql-deployment 
  name: mysql-deployment 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: mysql-deployment 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: mysql-deployment 
    spec: 
      containers: 
      - image: mysql:latest 
        name: mysql 
        env: 
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: mysql-root-pass 
              key: password 
        - name: MYSQL_DATABASE 
          valueFrom: 
            secretKeyRef: 
              name: mysql-db-url
              key: database 
        - name: MYSQL_USER 
          valueFrom: 
            secretKeyRef: 
              name: mysql-user-pass 
              key: username 
        - name: MYSQL_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: mysql-user-pass 
              key: password 
        volumeMounts: 
        - name: mysql-pv-claim 
          mountPath: /var/lib/mysql 
      volumes: 
      - name: mysql-pv-claim 
        persistentVolumeClaim: 
          claimName: mysql-pv-claim 


thor@jump_host ~$ kubectl create -f deploy.yaml
thor@jump_host ~$ kubectl get pods 
NAME                                READY   STATUS    RESTARTS   AGE 
mysql-deployment-678665c57f-f5m69   1/1     Running   0          26s 
```