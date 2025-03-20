# Level 3 - Lab 08 - Kubernetes Troubleshooting
## Task
Fix errors in kubernetes manifest `/home/thor/mysql_deployment.yml`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=qiymaD6G0m0)

```
thor@jump_host ~$ cat /home/thor/mysql_deployment.yml

apiVersion: apps/v1  
kind: PersistentVolume             
metadata: 
  name: mysql-pv 
  labels:  
  type: local  
spec: 
  storageClassName: standard       
  capacity: 
    storage: 250Mi 
  accessModes:  
    - ReadWriteOnce 
  hostPath:                        
  path: "/mnt/data"  
  persistentVolumeReclaimPolicy:   
  -  Retain   
---     
apiVersion: apps/v1  
kind: PersistentVolumeClaim  
metadata:                           
  name: mysql-pv-claim 
  labels: 
  app: mysql-app  
spec:                               
  storageClassName: standard        
  accessModes: 
    - ReadWriteOnce                 
  resources: 
    requests:  
      storage: 250MB  
--- 
apiVersion: v1                     
kind: Service                       
metadata: 
  name: mysql          
  labels:              
    app: mysql-app 
spec: 
  type: NodePort 
  ports: 
    - targetPort: 3306 
      port: 3306 
      nodePort: 30011 
  selector:     
    app: mysql-app 
  tier: mysql 
--- 
apiVersion: v1  
kind: Deployment             
metadata: 
  name: mysql-deployment        
  labels:                        
    app: mysql-app  
spec: 
  selector: 
    matchLabels: 
      app: mysql-app 
    tier: mysql  
  strategy: 
    type: Recreate 
  template:                     
    metadata: 
      labels:                   
        app: mysql-app 
      tier: mysql  
    spec:                        
      containers:  
      - images: mysql:5.6  
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
        ports: 
        - containerPort: 3306         
          name: mysql 
        volumeMounts: 
        - name: mysql-persistent-storage  
          mountPath: /var/lib/mysql 
      volumes:                        
      - name: mysql-persistent-storage 
          persistentVolumeClaim: 
          claimName: mysql-pv-claim

thor@jump_host ~$ vi /home/thor/mysql_deployment.yml

(edit problems, see fixed version below)
```
Fixed manifest:

```
thor@jump_host ~$ cat /home/thor/mysql_deployment.yml

apiVersion: v1  
kind: PersistentVolume             
metadata: 
  name: mysql-pv 
  labels:  
    type: local  
spec: 
  storageClassName: standard       
  capacity: 
    storage: 250Mi 
  accessModes:  
    - ReadWriteOnce 
  hostPath:                        
    path: "/mnt/data"  
  persistentVolumeReclaimPolicy: Retain   
---     
apiVersion: v1  
kind: PersistentVolumeClaim  
metadata:                           
  name: mysql-pv-claim 
  labels: 
    app: mysql-app  
spec:                               
  storageClassName: standard        
  accessModes: 
    - ReadWriteOnce                 
  resources: 
    requests:  
      storage: 250Mi 
--- 
apiVersion: v1                     
kind: Service                       
metadata: 
  name: mysql          
  labels:              
    app: mysql-app 
spec: 
  type: NodePort 
  ports: 
    - targetPort: 3306 
      port: 3306 
      nodePort: 30011 
  selector:     
    app: mysql-app 
    tier: mysql 
--- 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: mysql-deployment 
  labels: 
    app: mysql-app 
spec: 
  selector: 
    matchLabels: 
      app: mysql-app 
      tier: mysql 
  strategy: 
    type: Recreate 
  template: 
    metadata: 
      labels: 
        app: mysql-app 
        tier: mysql 
    spec: 
      containers: 
      - image: mysql:5.6  # Fixed 'images' to 'image' 
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
        ports: 
        - containerPort: 3306 
          name: mysql 
        volumeMounts: 
        - name: mysql-persistent-storage 
          mountPath: /var/lib/mysql 
      volumes: 
      - name: mysql-persistent-storage 
        persistentVolumeClaim: 
          claimName: mysql-pv-claim 
```