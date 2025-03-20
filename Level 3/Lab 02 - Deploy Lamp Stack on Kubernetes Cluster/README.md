# Level 3 - Lab 02 - Deploy Lamp Stack on Kubernetes Cluster
## Task
1. Create a config map `php-config` for `php.ini` with `variables_order = "EGPCS"` data.

2. Create a deployment named `lamp-wp`.

3. Create two containers under it. First container must be `httpd-php-container` using image `webdevops/php-apache:alpine-3-php7` and second container must be `mysql-container` from image `mysql:5.6`. Mount `php-config` configmap in httpd container at `/opt/docker/etc/php/php.ini` location.

4. Create kubernetes generic secrets for mysql related values like mysql root password, mysql user, mysql password, mysql host and mysql database. Set any values of your choice.

5. Add some environment variables for both containers: 

   a) `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD` and `MYSQL_HOST`. Take their values from the secrets you created. Please make sure to use `env` field (do not use `envFrom`) to define the name-value pair of environment variables.

6. Create a node port type service `lamp-service` to expose the web application, nodePort must be `30008`.

7. Create a service for mysql named `mysql-service` and its port must be `3306`.

8. We already have `/tmp/index.php` file on `jump_host` server.

   a) Copy this file into httpd container under Apache document root i.e `/app` and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the enrivornment variables to fetch those values.

   b) You must be able to access this `index.php` on node port `30008` at the end, please note that you should see `Connected successfully` message while accessing this page.
## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=-gH3mlk3OQw)

Task 1. Create Configmap:
```
thor@jump_host ~$ vi php.ini 
thor@jump_host ~$ cat php.ini 
variables_order = "EGPCS" 
 
thor@jump_host ~$ kubectl create configmap php-config --from-file=php.ini 
configmap/php-config created 

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

Task 4. Create secret:
```
thor@jump_host ~$ kubectl create secret --help 
thor@jump_host ~$ kubectl create secret generic my-secret --from-literal=MYSQL_ROOT_PASSWORD=password --from-literal=MYSQL_DATABASE=database --from-literal=MYSQL_USER=user --from-literal=MYSQL_PASSWORD=mysqlpwd --from-literal=MYSQL_HOST=mysql-service 
secret/my-secret created 

thor@jump_host ~$ kubectl get secret 
NAME        TYPE     DATA   AGE 
my-secret   Opaque   5      6s 

thor@jump_host ~$ kubectl describe secret my-secret 
Name:         my-secret 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 
 
Type:  Opaque 

Data 
==== 
MYSQL_HOST:           13 bytes 
MYSQL_PASSWORD:       8 bytes 
MYSQL_ROOT_PASSWORD:  8 bytes 
MYSQL_USER:           4 bytes 
MYSQL_DATABASE:       8 bytes 
```

Task 2. and 3. Create Deployment:
```
thor@jump_host ~$ kubectl create deployment lamp-wp --image=webdevops/php-apache:alpine-3-php7 --replicas=1 --dry-run=client -o yaml 

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  creationTimestamp: null 
  labels: 
    app: lamp-wp 
  name: lamp-wp 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: lamp-wp 
  strategy: {} 
  template: 
    metadata: 
      creationTimestamp: null 
      labels: 
        app: lamp-wp 
    spec: 
      containers: 
      - image: webdevops/php-apache:alpine-3-php7 
        name: php-apache 
        resources: {} 
status: {} 

thor@jump_host ~$ kubectl create deployment lamp-wp --image=webdevops/php-apache:alpine-3-php7 --replicas=1 --dry-run=client -o yaml > deployment.yaml 
thor@jump_host ~$ vi deployment.yaml 
thor@jump_host ~$ cat deployment.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: lamp-wp 
  name: lamp-wp 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: lamp-wp 
  strategy: {} 
  template: 
    metadata: 
      labels: 
        app: lamp-wp 
    spec: 
      containers: 
      - image: webdevops/php-apache:alpine-3-php7 
        name: httpd-php-container 
        env: 
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_ROOT_PASSWORD 
        - name: MYSQL_DATABASE 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_DATABASE 
        - name: MYSQL_USER 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_USER 
        - name: MYSQL_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_PASSWORD 
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_HOST 
        volumeMounts: 
        - mountPath: /opt/docker/etc/php/php.ini 
          name: php-config 
          subPath: php.ini 
      - image: mysql:5.6 
        name: mysql-container 
        env: 
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_ROOT_PASSWORD 
        - name: MYSQL_DATABASE 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_DATABASE 
        - name: MYSQL_USER 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_USER 
        - name: MYSQL_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_PASSWORD 
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_HOST 
      volumes: 
      - configMap: 
          name: php-config 
        name: php-config 
```

Task 6. Create service `lamp-service`:
```
thor@jump_host ~$ kubectl expose deployment lamp-wp --name=lamp-service --type=NodePort --port=30008 
service/lamp-service exposed

thor@jump_host ~$ kubectl edit svc lamp-service  
(edit "nodePort: 30008" , "port: 80" , "targetPort: 80")

thor@jump_host ~$ kubectl get svc 
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        28m 
lamp-service   NodePort    10.96.194.106   <none>        80:30008/TCP   74s 
```

Task 7. Create service `mysql-service`:
```
thor@jump_host ~$ kubectl expose deployment lamp-wp --name=mysql-service --port=3306 
service/mysql-service exposed 

thor@jump_host ~$ kubectl get svc 
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        35m 
lamp-service    NodePort    10.96.194.106   <none>        80:30008/TCP   7m56s 
mysql-service   ClusterIP   10.96.223.142   <none>        3306/TCP       4s 
```

Task 8.:
```
thor@jump_host ~$ cat /tmp/index.php 
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
  echo "Connected successfully"; 

thor@jump_host ~$ kubectl cp /tmp/index.php lamp-wp-7f67984579-qdftd:/app -c httpd-php-container 
thor@jump_host ~$ kubectl exec -it lamp-wp-7f67984579-qdftd -c httpd-php-container -- sh 
/ # cd /app 
/app # ls 
index.php 
/app # cat index.php  
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
  echo "Connected successfully"; 

/app # printenv | grep MYSQL 
MYSQL_ROOT_PASSWORD=password 
MYSQL_PASSWORD=mysqlpwd 
MYSQL_HOST=mysql-service 
MYSQL_USER=user 
MYSQL_DATABASE=database 
```

Task 8.a):
```
thor@jump_host ~$ cp /tmp/index.php fixed-index.php 
thor@jump_host ~$ vi fixed-index.php   
thor@jump_host ~$ cat fixed-index.php  

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
  echo "Connected successfully"; 

thor@jump_host ~$ kubectl create cm php-index-cm --from-file=fixed-index.php 
configmap/php-index-cm created 

thor@jump_host ~$ kubectl describe cm php-index-cm  

Name:         php-index-cm 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 

Data 
==== 
fixed-index.php: 
---- 
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
  echo "Connected successfully"; 

 
BinaryData 
==== 

Events:  <none> 
 
thor@jump_host ~$ kubectl delete deploy lamp-wp  
deployment.apps "lamp-wp" deleted 

thor@jump_host ~$ cp deployment.yaml deployment-2.yaml 
thor@jump_host ~$ vi deployment-2.yaml  
thor@jump_host ~$ cat deployment-2.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: lamp-wp 
  name: lamp-wp 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: lamp-wp 
  strategy: {} 
  template: 
    metadata: 
      labels: 
        app: lamp-wp 
    spec: 
      containers: 
      - image: webdevops/php-apache:alpine-3-php7 
        name: httpd-php-container 
        env: 
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_ROOT_PASSWORD 
        - name: MYSQL_DATABASE 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_DATABASE 
        - name: MYSQL_USER 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_USER 
        - name: MYSQL_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_PASSWORD 
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_HOST 
        volumeMounts: 
        - mountPath: /opt/docker/etc/php/php.ini 
          name: php-config 
          subPath: php.ini 
        - mountPath: /app/index.php 
          subPath: index.php 
          name: php-index-cm 
      - image: mysql:5.6 
        name: mysql-container 
        env: 
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_ROOT_PASSWORD 
        - name: MYSQL_DATABASE 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_DATABASE 
        - name: MYSQL_USER 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_USER 
        - name: MYSQL_PASSWORD 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_PASSWORD 
        - name: MYSQL_HOST 
          valueFrom: 
            secretKeyRef: 
              name: my-secret 
              key: MYSQL_HOST 
      volumes: 
      - configMap: 
          name: php-config 
        name: php-config 
      - configMap: 
          name: php-index-cm 
        name: php-index-cm 
```