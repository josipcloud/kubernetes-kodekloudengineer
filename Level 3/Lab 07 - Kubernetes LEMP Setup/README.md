# Level 3 - Lab 07 - Kubernetes LEMP Setup
## Task
1. Create some secrets for MySQL.
- Create a secret named `mysql-root-pass` with key/value pairs as below:
```
name: password
value: R00t
```
- Create a secret named `mysql-user-pass` with key/value pairs as below:
```
name: username
value: kodekloud_joy

name: password:
value: GyQkFRVNr3
```
- Create a secret named `mysql-db-url` with key/value pairs as below:
```
name: database
value: kodekloud_db5
```
- Create a secret named `mysql-host` with key/value pairs as below:
```
name: host
value: mysql-service
```

2. Create a config map `php-config` for `php.ini` with `variables_order = "EGPCS"` data.

3. Create a deployment named `lemp-wp`.

4. Create two containers under it. First container must be `nginx-php-container` using image `webdevops/php-nginx:alpine-3-php7` and second container must be `mysql-container` from image `mysql:5.6`. Mount `php-config` configmap in nginx container at `/opt/docker/etc/php/php.ini` location.

5. Add some environment variables for both containers:

- `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_HOST`. Take their values from the secrets you created. Please make sure to use env field (do not use envFrom) to define the name-value pair of environment variables.

6. Create a node port type service `lemp-service` to expose the web application, nodePort must be `30008`.

7. Create a service for mysql named `mysql-service` and its port must be `3306`.

8. We already have a `/tmp/index.php` file on `jump_host` server.

- Copy this file into the `nginx` container under document root i.e `/app` and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the environment variables to fetch those values.

- Once done, you must be able to access this website using `Website` button on the top bar, please note that you should see `Connected successfully` message while accessing this page.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=66Ow7ZyRKwE)


Task 1 - Create secrets:

```
thor@jump_host ~$ kubectl create secret generic mysql-root-pass --from-literal=password=R00t 
secret/mysql-root-pass created 

thor@jump_host ~$ kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_joy --from-literal=password=GyQkFRVNr3 
secret/mysql-user-pass created 

thor@jump_host ~$ kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db5 
secret/mysql-db-url created 

thor@jump_host ~$ kubectl create secret generic mysql-host --from-literal=host=mysql-service 
secret/mysql-host created 
```

Task 2 - Create a ConfigMap:

```
thor@jump_host ~$ vi php.ini 
thor@jump_host ~$ cat php.ini 
variables_order = "EGPCS" 

thor@jump_host ~$ kubectl create cm php-config --from-file=php.ini 
configmap/php-config created 

thor@jump_host ~$ kubectl get cm 
NAME               DATA   AGE 
kube-root-ca.crt   1      57m 
php-config         1      2s
 
thor@jump_host ~$ kubectl describe cm php-config 
Name:         php-config 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 

Data 
==== 
php.ini: 
---- 
variables_order = "EGPCS" 

 
BinaryData 
==== 

Events:  <none> 
```

Task 3,4,5 - Create a deployment `lemp-wp`:

```
thor@jump_host ~$ vi deploy.yaml 
thor@jump_host ~$ cat deploy.yaml 

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    app: lemp-wp 
  name: lemp-wp 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: lemp-wp 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: lemp-wp 
    spec: 
      containers: 
      - image: webdevops/php-nginx:alpine-3-php7 
        name: nginx-php-container 
        volumeMounts: 
          - mountPath: /opt/docker/etc/php/php.ini 
            name: php-ini 
            subPath: php.ini 
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
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: mysql-host 
              key: host 
      - image: mysql:5.6 
        name: mysql-container 
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
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: mysql-host 
              key: host 
      volumes: 
        - name: php-ini 
          configMap: 
            name: php-config 
```

Task 6: Create a service `lemp-service`:

```
thor@jump_host ~$ kubectl expose deploy lemp-wp --name=lemp-service --type=NodePort --port=80 
service/lemp-service exposed 

thor@jump_host ~$ kubectl edit svc lemp-service

(edit "nodePort: 30008", "port: 80", "targetPort: 80")

thor@jump_host ~$ kubectl get svc 
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE 
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP        28m 
lemp-service   NodePort    10.96.64.145   <none>        80:30008/TCP   4m38s   
```

Task 7: Create a service `mysql-service`:

```
thor@jump_host ~$ kubectl expose deploy lemp-wp --name=mysql-service --port=3306 
service/mysql-service exposed 

thor@jump_host ~$ kubectl get svc 
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        29m 
lemp-service    NodePort    10.96.64.145    <none>        80:30008/TCP   5m20s 
mysql-service   ClusterIP   10.96.158.186   <none>        3306/TCP       2s 
```

Task 8: Copy files:

```
thor@jump_host ~$ kubectl cp /tmp/index.php lemp-wp-5cc5566d69-dpgxm:/app -c nginx-php-container 
thor@jump_host ~$ kubectl exec -it lemp-wp-5cc5566d69-dpgxm -c nginx-php-container -- sh 
/ # cat /app/index.php 
<?php 
$dbname = 'dbname'; 
$dbuser = 'dbuser'; 
$dbpass = 'dbpass'; 
$dbhost = 'dbhost'; 
 
$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'"); 

$test_query = "SHOW TABLES FROM $dbname"; 
$result = mysqli_query($test_query); 

if ($result->connect_error) { 
   die("Connection failed: " . $conn->connect_error); 
} 
  echo "Connected successfully";/ #  
/ # vi /app/index.php 
/ # cat /app/index.php 
<?php 
$dbname = $_ENV['MYSQL_DATABASE']; 
$dbuser = $_ENV['MYSQL_USER']; 
$dbpass = $_ENV['MYSQL_PASSWORD']; 
$dbhost = $_ENV['MYSQL_HOST']; 

$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'"); 

$test_query = "SHOW TABLES FROM $dbname"; 
$result = mysqli_query($test_query); 
 
if ($result->connect_error) { 
   die("Connection failed: " . $conn->connect_error); 
} 
  echo "Connected successfully";/ #  
```
(Click on the `Website` button to access the app, you should get **Connected successfully**.)