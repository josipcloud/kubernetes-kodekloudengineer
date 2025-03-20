# Lab 14 - Fix Issue with Volume Mounts in Kubernetes
## Task
The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Figure out the issue and fix it.

Once issue is fixed, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` under nginx document root and you should be able to access the website using `Website` button on top bar.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=8vOOFPIK0YI)

```
thor@jump_host ~$ kubectl get pods 

NAME           READY   STATUS    RESTARTS   AGE 
nginx-phpfpm   2/2     Running   0          30m 

thor@jump_host ~$ kubectl describe pod nginx-phpfpm 

Name:             nginx-phpfpm 
Namespace:        default 
Priority:         0 
Service Account:  default 
Node:             kodekloud-control-plane/172.17.0.2 
Start Time:       Tue, 31 Oct 2023 11:11:46 +0000 
Labels:           app=php-app 
Annotations:      <none> 
Status:           Running 
IP:               10.244.0.5 
IPs: 
  IP:  10.244.0.5 
Containers: 
  php-fpm-container: 
    Container ID:   containerd://525af9bca40a7c27b69a44f5b62202c1970960e4abb34bede7180b7899755936 
    Image:          php:7.2-fpm-alpine 
    Image ID:       docker.io/library/php@sha256:2e2d92415f3fc552e9a62548d1235f852c864fcdc94bcf2905805d92baefc87f 
    Port:           <none> 
    Host Port:      <none> 
    State:          Running 
      Started:      Tue, 31 Oct 2023 11:11:52 +0000 
    Ready:          True 
    Restart Count:  0 
    Environment:    <none> 
    Mounts: 
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xp5gm (ro) 
      /var/www/html from shared-files (rw) 
  nginx-container: 
    Container ID:   containerd://24f8f0a0e1ac3b524e7a9e6f3ef9259eb91839a12c742ef841c21f35e38a89ec 
    Image:          nginx:latest 
    Image ID:       docker.io/library/nginx@sha256:add4792d930c25dd2abf2ef9ea79de578097a1c175a16ab25814332fe33622de 
    Port:           <none> 
    Host Port:      <none> 
    State:          Running 
      Started:      Tue, 31 Oct 2023 11:12:02 +0000 
    Ready:          True 
    Restart Count:  0 
    Environment:    <none> 
    Mounts: 
      /etc/nginx/nginx.conf from nginx-config-volume (rw,path="nginx.conf") 
      /usr/share/nginx/html from shared-files (rw)    < - - - - - - THIS IS AN ERROR, WE NEED TO EDIT IT TO /var/www/html from shared-files
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xp5gm (ro) 
Conditions: 
  Type              Status 
  Initialized       True  
  Ready             True  
  ContainersReady   True  
  PodScheduled      True  
Volumes: 
  shared-files: 
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime) 
    Medium:      
    SizeLimit:  <unset> 
  nginx-config-volume: 
    Type:      ConfigMap (a volume populated by a ConfigMap) 
    Name:      nginx-config 
    Optional:  false 
  kube-api-access-xp5gm: 
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
  Normal  Scheduled  30m   default-scheduler  Successfully assigned default/nginx-phpfpm to kodekloud-control-plane 
  Normal  Pulling    30m   kubelet            Pulling image "php:7.2-fpm-alpine" 
  Normal  Pulled     30m   kubelet            Successfully pulled image "php:7.2-fpm-alpine" in 4.790315545s (4.790328622s including waiting) 
  Normal  Created    30m   kubelet            Created container php-fpm-container 
  Normal  Started    30m   kubelet            Started container php-fpm-container 
  Normal  Pulling    30m   kubelet            Pulling image "nginx:latest" 
  Normal  Pulled     30m   kubelet            Successfully pulled image "nginx:latest" in 9.466225231s (9.466234683s including waiting) 
  Normal  Created    30m   kubelet            Created container nginx-container 
  Normal  Started    30m   kubelet            Started container nginx-container 


thor@jump_host ~$ kubectl edit pod nginx-phpfpm

(edit "/usr/share/nginx/html" from shared-files (rw) to "/var/www/html from shared-files")

thor@jump_host ~$ kubectl get cm 

NAME               DATA   AGE 
kube-root-ca.crt   1      51m 
nginx-config       1      30m 

thor@jump_host ~$ kubectl describe cm nginx-config 

Name:         nginx-config 
Namespace:    default 
Labels:       <none> 
Annotations:  <none> 

Data 
==== 
nginx.conf: 
---- 
events { 
} 
http { 
  server { 
    listen 8099 default_server; 
    listen [::]:8099 default_server; 
 

    # Set nginx to serve files from the shared volume! 
    root /var/www/html; 
    index  index.html index.htm index.php; 
    server_name _; 
    location / { 
      try_files $uri $uri/ =404; 
    } 
    location ~ \.php$ { 
      include fastcgi_params; 
      fastcgi_param REQUEST_METHOD $request_method; 
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
      fastcgi_pass 127.0.0.1:9000; 
    } 
  } 
} 


BinaryData 

==== 

Events:  <none> 

 

--------------------------------------------------------------

 
thor@jump_host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- /bin/bash 
root@nginx-phpfpm:/# ls 
bin   dev                  docker-entrypoint.sh  home  lib32  libx32  mnt  proc  run   srv  tmp  var 
boot  docker-entrypoint.d  etc                   lib   lib64  media   opt  root  sbin  sys  usr 
root@nginx-phpfpm:/# ls -al 
total 76 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 . 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 .. 
lrwxrwxrwx    1 root   root       7 Oct  9 00:00 bin -> usr/bin 
drwxr-xr-x    2 root   root    4096 Sep 29 20:04 boot 
drwxr-xr-x    5 root   root     360 Oct 31 11:12 dev 
drwxr-xr-x    1 root   root    4096 Oct 25 01:21 docker-entrypoint.d 
-rwxrwxr-x    1 root   root    1620 Oct 25 01:21 docker-entrypoint.sh 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 etc 
drwxr-xr-x    2 root   root    4096 Sep 29 20:04 home 
lrwxrwxrwx    1 root   root       7 Oct  9 00:00 lib -> usr/lib 
lrwxrwxrwx    1 root   root       9 Oct  9 00:00 lib32 -> usr/lib32 
lrwxrwxrwx    1 root   root       9 Oct  9 00:00 lib64 -> usr/lib64 
lrwxrwxrwx    1 root   root      10 Oct  9 00:00 libx32 -> usr/libx32 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 media 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 mnt 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 opt 
dr-xr-xr-x 3044 root   root       0 Oct 31 11:12 proc 
drwx------    1 root   root    4096 Oct 31 11:32 root 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 run 
lrwxrwxrwx    1 root   root       8 Oct  9 00:00 sbin -> usr/sbin 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 srv 
dr-xr-xr-x   13 nobody nogroup    0 Oct 31 11:11 sys 
drwxrwxrwt    1 root   root    4096 Oct 25 01:21 tmp 
drwxr-xr-x    1 root   root    4096 Oct  9 00:00 usr 
drwxr-xr-x    1 root   root    4096 Oct  9 00:00 var 


(To copy local file within a container:)

thor@jump_host ~$ kubectl cp index.php nginx-phpfpm:/index.php -c nginx-container 

root@nginx-phpfpm:/# ls -al 
total 80 
drwxr-xr-x    1 root   root    4096 Oct 31 11:39 . 
drwxr-xr-x    1 root   root    4096 Oct 31 11:39 .. 
lrwxrwxrwx    1 root   root       7 Oct  9 00:00 bin -> usr/bin 
drwxr-xr-x    2 root   root    4096 Sep 29 20:04 boot 
drwxr-xr-x    5 root   root     360 Oct 31 11:12 dev 
drwxr-xr-x    1 root   root    4096 Oct 25 01:21 docker-entrypoint.d 
-rwxrwxr-x    1 root   root    1620 Oct 25 01:21 docker-entrypoint.sh 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 etc 
drwxr-xr-x    2 root   root    4096 Sep 29 20:04 home 
-rw-r--r--    1 root   root      19 Oct 31 11:39 index.php 
lrwxrwxrwx    1 root   root       7 Oct  9 00:00 lib -> usr/lib 
lrwxrwxrwx    1 root   root       9 Oct  9 00:00 lib32 -> usr/lib32 
lrwxrwxrwx    1 root   root       9 Oct  9 00:00 lib64 -> usr/lib64 
lrwxrwxrwx    1 root   root      10 Oct  9 00:00 libx32 -> usr/libx32 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 media 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 mnt 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 opt 
dr-xr-xr-x 3101 root   root       0 Oct 31 11:12 proc 
drwx------    1 root   root    4096 Oct 31 11:32 root 
drwxr-xr-x    1 root   root    4096 Oct 31 11:12 run 
lrwxrwxrwx    1 root   root       8 Oct  9 00:00 sbin -> usr/sbin 
drwxr-xr-x    2 root   root    4096 Oct  9 00:00 srv 
dr-xr-xr-x   13 nobody nogroup    0 Oct 31 11:11 sys 
drwxrwxrwt    1 root   root    4096 Oct 25 01:21 tmp 
drwxr-xr-x    1 root   root    4096 Oct  9 00:00 usr 
drwxr-xr-x    1 root   root    4096 Oct  9 00:00 var 

root@nginx-phpfpm:/# cp index.php /var/www/html 
root@nginx-phpfpm:/# cd /var/www/html 
root@nginx-phpfpm:/var/www/html# ls 
index.php 
root@nginx-phpfpm:/var/www/html#   
```