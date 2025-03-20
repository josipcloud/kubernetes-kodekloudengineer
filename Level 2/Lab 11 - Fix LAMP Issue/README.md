# Level 2 - Lab 11 - Fix LAMP Issue
## Task

Fix the issues:


Deployment name is `lamp-wp` and its using a service named `lamp-service`. The Apache is using httpd default port and nodePort is `30008`. From the application logs it has been identified that application is facing some issue while connecting to the database in addition to other issues. Additionally, there are some environment variables associated with the pods like `MYSQL_ROOT_PASSWORD, MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD, MYSQL_HOST`.

Also do not try to delete/modify any other existing components like deployment name, service name, txpes, labels etc.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=X9MK_T4mc1w)

1. Edit `lamp-service` service:
```
thor@jump_host ~$ kubectl get deploy 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE 
lamp-wp   1/1     1            1           43s 

thor@jump_host ~$ kubectl get svc 
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE 
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP          63m 
lamp-service    NodePort    10.96.9.219    <none>        8080:30008/TCP   48s 
mysql-service   ClusterIP   10.96.117.97   <none>        3306/TCP         48s 

thor@jump_host ~$ kubectl edit svc lamp-service 
service/lamp-service edited

(edit "nodePort: 30008", "port: 80", "targetPort: 80") 
```

Check `index.php` configuration for errors:

```
thor@jump_host ~$ kubectl exec -it lamp-wp-56c7c454fc-ss6tm -- sh 
Defaulted container "httpd-php-container" out of: httpd-php-container, mysql-container 
/ # cd /app/ 
/app # ls 
index.php 
/app # cat index.php  
<?php 
$dbname = $_ENV['MYSQL_DATABASE']; 
$dbuser = $_ENV['MYSQL_USER']; 
$dbpass = $_ENV['MYSQL_PASSWORDS'];  <-------- error, "S" on the end. 
$dbhost = $_ENV['HOST_MYQSL'];       <-------- error, it should be MYSQL_HOST 

 
$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'"); 

$test_query = "SHOW TABLES FROM $dbname"; 
$result = mysqli_query($test_query); 

if ($result->connect_error) { 
   die("Connection failed: " . $conn->connect_error); 
} 
/app # exit 
```
There are two errors in the file as menitoned inline.
Copy the above file `index.php` to jump host and edit it:

```
thor@jump_host ~$ vi index.php 
thor@jump_host ~$ cat index.php  
<?php 
$dbname = $_ENV['MYSQL_DATABASE']; 
$dbuser = $_ENV['MYSQL_USER']; 
$dbpass = $_ENV['MYSQL_PASSWORD']; 
$dbhost = $_ENV['MYQSL_HOST']; 


$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'"); 

$test_query = "SHOW TABLES FROM $dbname"; 
$result = mysqli_query($test_query); 

if ($result->connect_error) { 
   die("Connection failed: " . $conn->connect_error); 
} 
  echo "Connected successfully"; 
```

Now we will create a configmap `fixed-php-config` from the `index.php` file:

```
thor@jump_host ~$ kubectl create configmap fixed-php-config --from-file index.php 
configmap/fixed-php-config created 

thor@jump_host ~$ kubectl get cm 
NAME               DATA   AGE 
fixed-php-config   1      2s 
kube-root-ca.crt   1      78m 
php-config         1      8m10s 

thor@jump_host ~$ kubectl describe cm fixed-php-config  
Name:         fixed-php-config 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 
 
Data 
==== 
index.php: 
---- 
<?php 
$dbname = $_ENV['MYSQL_DATABASE']; 
$dbuser = $_ENV['MYSQL_USER']; 
$dbpass = $_ENV['MYSQL_PASSWORD']; 
$dbhost = $_ENV['MYQSL_HOST']; 


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
```
Now we need to mount this newly created configmap `fixed-php-config` in the pod, in `php` container at location `/app/index.php`. So we need to edit deployment `lamp-wp`:

```
thor@jump_host ~$ kubectl edit deploy lamp-wp

...
volumeMounts:                 <---- under this
...
- mountPath: /app/index.php   <---- add this
  name: fixed-php-config      <---- add this
  subPath: index.php          <---- add this
...
```
Note: it is important to define `subPath: index.php`, otherwise, the index.php gets created as folder, instead of file.
Continuing, check `lamp-wp` deployment now, it should look somehow like this, the new configmap `fixed-php-config` needs to be mounted at `/app/index.php`:

```
thor@jump_host ~$ kubectl describe deploy lamp-wp  
Name:               lamp-wp 
Namespace:          default 
CreationTimestamp:  Mon, 20 Nov 2023 21:32:34 +0000 
Labels:             app=lamp 
Annotations:        deployment.kubernetes.io/revision: 2 
Selector:           app=lamp,tier=frontend 
Replicas:           1 desired | 1 updated | 1 total | 0 available | 1 unavailable 
StrategyType:       Recreate 
MinReadySeconds:    0 
Pod Template: 
  Labels:  app=lamp 
           tier=frontend 
  Containers: 
   httpd-php-container: 
    Image:      webdevops/php-apache:alpine-3-php7 
    Port:       80/TCP 
    Host Port:  0/TCP 
    Environment: 
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-root-pass'>  Optional: false 
      MYSQL_DATABASE:       <set to the key 'database' in secret 'mysql-db-url'>     Optional: false 
      MYSQL_USER:           <set to the key 'username' in secret 'mysql-user-pass'>  Optional: false 
      MYSQL_PASSWORD:       <set to the key 'password' in secret 'mysql-user-pass'>  Optional: false 
      MYSQL_HOST:           <set to the key 'host' in secret 'mysql-host'>           Optional: false 
    Mounts: 
      /app/index.php from fixed-php-config (rw,path="index.php")   <--------------- THIS IS NEW
      /opt/docker/etc/php/php.ini from php-config-volume (rw,path="php.ini") 
   mysql-container: 
    Image:      mysql:5.6 
    Port:       3306/TCP 
    Host Port:  0/TCP 
    Environment: 
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-root-pass'>  Optional: false 
      MYSQL_DATABASE:       <set to the key 'database' in secret 'mysql-db-url'>     Optional: false 
      MYSQL_USER:           <set to the key 'username' in secret 'mysql-user-pass'>  Optional: false 
      MYSQL_PASSWORD:       <set to the key 'password' in secret 'mysql-user-pass'>  Optional: false 
      MYSQL_HOST:           <set to the key 'host' in secret 'mysql-host'>           Optional: false 
    Mounts:                 <none> 
  Volumes: 
   php-config-volume: 
    Type:      ConfigMap (a volume populated by a ConfigMap) 
    Name:      php-config 
    Optional:  false 
   fixed-php-config:                                         <--------- THIS IS NEW
    Type:      ConfigMap (a volume populated by a ConfigMap) <--------- THIS IS NEW
    Name:      fixed-php-config                              <--------- THIS IS NEW
    Optional:  false                                         <--------- THIS IS NEW
Conditions: 
  Type           Status  Reason 
  ----           ------  ------ 
  Available      False   MinimumReplicasUnavailable 
  Progressing    True    ReplicaSetUpdated 
OldReplicaSets:  lamp-wp-56c7c454fc (0/0 replicas created) 
NewReplicaSet:   lamp-wp-748b69d7f6 (1/1 replicas created) 
Events: 
  Type    Reason             Age   From                   Message 
  ----    ------             ----  ----                   ------- 
  Normal  ScalingReplicaSet  12m   deployment-controller  Scaled up replica set lamp-wp-56c7c454fc to 1 
  Normal  ScalingReplicaSet  5s    deployment-controller  Scaled down replica set lamp-wp-56c7c454fc to 0 from 1 
  Normal  ScalingReplicaSet  2s    deployment-controller  Scaled up replica set lamp-wp-748b69d7f6 to 1 
```

Now check inside a container if the file is mounted correctly:

```
thor@jump_host ~$ kubectl exec -it lamp-wp-748b69d7f6-xszd2 -c httpd-php-container -- sh 
/ # cd app/ 
/app # ls 
index.php 
/app # cat index.php  
<?php 
$dbname = $_ENV['MYSQL_DATABASE']; 
$dbuser = $_ENV['MYSQL_USER']; 
$dbpass = $_ENV['MYSQL_PASSWORD']; 
$dbhost = $_ENV['MYQSL_HOST']; 
 
 
$connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'"); 

$test_query = "SHOW TABLES FROM $dbname"; 
$result = mysqli_query($test_query); 

if ($result->connect_error) { 
   die("Connection failed: " . $conn->connect_error); 
} 
  echo "Connected successfully"; 
```

Now check the Web Button, the access should work, or you can do curl within the container:

```
thor@jump_host ~$ kubectl exec -it lamp-wp-748b69d7f6-xszd2 -c httpd-php-container -- sh 
/ # curl http://localhost:80
Connected successfully
```