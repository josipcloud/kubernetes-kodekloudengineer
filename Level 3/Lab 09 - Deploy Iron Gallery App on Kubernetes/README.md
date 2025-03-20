# Level 3 - Lab 09 - Deploy Iron Gallery App on Kubernetes
## Task
1. Create a namespace `iron-namespace-xfusion`.

2. Create a deployment `iron-gallery-deployment-xfusion` for `iron-gallery` under the same namespace you created.

- Labels `run` should be `iron-gallery`.
- Replicas count should be `1`.
- Selector's matchLabels `run` should be `iron-gallery`.
- Template labels `run` should be `iron-gallery` under metadata.
- The container should be named `iron-gallery-container-xfusion`, use `kodekloud/irongallery:2.0` image (use exact image name / tag).
- Resources limits for memory should be `100Mi` and for CPU should be `50m`.
- First volumeMount name should be `config`, its mountPath should be `/usr/share/nginx/html/data`.
- Second volumeMount name should be `images`, its mountPath should be `/usr/share/nginx/html/uploads`.
- First volume name should be `config` and give it `emptyDir` and second volume name should be `images`, also give it `emptyDir`.

3. Create a deployment `iron-db-deployment-xfusion` for `iron db` under the same namespace.
- Labels `db` should be `mariadb`.
- Replicas count should be `1`.
- Selector's matchLabels `db` should be `mariadb`.
- Template labels `db` should be `mariadb` under metadata.
- The container name should be `iron-db-container-xfusion`, use `kodekloud/irondb:2.0` image (use exact image name / tag).
- Define environment, set `MYSQL_DATABASE`, its value should be `database_apache`, set `MYSQL_ROOT_PASSWORD` and `MYSQL_PASSWORD` value should be with some complex passwords for DB connections, and `MYSQL_USER` value should be any custom user (except root).
- Volume mount name should be `db` and its mountPath should be `/var/lib/mysql`. Volume name should be `db` and give it an `emptyDir`.

4. Create a service for `iron db` which should be named `iron-db-service-xfusion` under the smae namespace. Configure spec as selector's db should be `mariadb`. Protocol should be `TCP`, port and targetPort should be `3306` and its type should be `ClusterIP`.

5. Create a service for `iron gallery` which should be named `iron-gallery-service-xfusion` under the same namespace. Configure spec as selector's run should be `iron-gallery`. Protocol should be `TCP`, port and targetPort should be `80`, nodePort should be `32678` and its type should be `NodePort`.

Note: We don't need to make connection between database and front-end for now, if the installation page is coming up it should be enough for now. 

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=OABi4e48mUw)

----
Task 1:
```
thor@jump_host ~$ kubectl create ns iron-namespace-xfusion 
namespace/iron-namespace-xfusion created 
```
Task 2:
```
thor@jump_host ~$ vi deploy.yaml
thor@jump_host ~$ cat deploy.yaml

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    run: iron-gallery 
  name: iron-gallery-deployment-xfusion 
  namespace: iron-namespace-xfusion 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      run: iron-gallery 
  strategy: {} 
  template: 
    metadata: 
      labels: 
        run: iron-gallery 
    spec: 
      containers: 
      - image: kodekloud/irongallery:2.0 
        name: iron-gallery-container-xfusion 
        resources: 
          limits: 
            memory: "100Mi" 
            cpu: "50m" 
        volumeMounts: 
        - mountPath: /usr/share/nginx/html/data 
          name: config 
        - mountPath: /usr/share/nginx/html/uploads 
          name: images 
      volumes: 
      - name: config 
        emptyDir: 
          sizeLimit: 100Mi 
      - name: images 
        emptyDir: 
          sizeLimit: 100Mi 
```
Task 3:

```
thor@jump_host ~$ vi deploy2.yaml  
thor@jump_host ~$ cat deploy2.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    db: mariadb 
  name: iron-db-deployment-xfusion 
  namespace: iron-namespace-xfusion 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      db: mariadb 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        db: mariadb 
    spec: 
      containers: 
      - image: kodekloud/irondb:2.0 
        name: iron-db-container-xfusion 
        env: 
        - name: MYSQL_DATABASE 
          value: database_apache 
        - name: MYSQL_ROOT_PASSWORD 
          value: brsdcrr2 
        - name: MYSQL_PASSWORD 
          value: fewfwdfwf442 
        - name: MYSQL_USER 
          value: john 
        volumeMounts: 
        - mountPath: /var/lib/mysql 
          name: db 
      volumes: 
      - name: db 
        emptyDir: 
          sizeLimit: 100Mi 
```
Task 4:
```
thor@jump_host ~$ kubectl expose deploy iron-db-deployment-xfusion -n iron-namespace-xfusion --name=iron-db-service-xfusion --port=3306 
service/iron-db-service-xfusion exposed 
```
Task 5:
```
thor@jump_host ~$ kubectl expose deploy iron-gallery-deployment-xfusion -n iron-namespace-xfusion  --type=NodePort --port=80 --name=iron-gallery-service-xfusion 
service/iron-gallery-service-xfusion exposed 

thor@jump_host ~$ kubectl edit svc iron-gallery-service-xfusion -n iron-namespace-xfusion 

(edit "nodePort: 32678", "port: 80", "targetport: 80")
```