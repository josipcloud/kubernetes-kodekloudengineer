# Level 4 - Lab 03 - Kubernetes Nginx and Php FPM Setup
## Task
1. Create a service to expose this app, the service type must be `NodePort`, nodePort should be `30012`.

2. Create a config map named `nginx-config` for `nginx.conf` as we want to add some custom settings in `nginx.conf`.

    a) Change the default port `80` to `8095` in `nginx.conf`.

    b) Change the default document root `/usr/share/nginx` to `/var/www/html` in `nginx.conf`.

    c) Update the directory index to `index index.html index.htm index.php` in `nginx.conf`.

3. Create a pod named `nginx-phpfpm`.

    a) Create a shared oclume named `shared-files` that will be used by both containers (nginx and phpfpm) also it should be a `emptyDir` volume.

    b) Map the ConfigMap we declared above as a volume for nginx container. Name the volume as `nginx-config-volume`, mount path should be `/etc/nginx/nginx.conf` and `subPath` should be `nginx.conf`.

    c) Nginx container should be named as `nginx-container` and it should use `nginx:latest` image. PhpFPM contaienr should be named `php-fpm-container` and it should use `php:8.2-fpm-alpine` image.

    d) The shared volume `shared-files` should be mounted at `/var/www/html` location in both containers. Copy `/opt/index.php` from `jump host` to the nginx document root inside the `nginx` container, oce done you can access the app using `App` button on the top bar.

    Before clicking on `finish` button always make sure to check if all pods are in `running` state.

    `You can use any labels as per your choice.`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=Jyc46AWRSyA)


```
apiVersion: v1 
kind: Service 
metadata: 
  name: httpd-service 
spec: 
  selector: 
    app: nginx 
  ports: 
    - protocol: TCP 
      port: 80 
      targetPort: 8091 
      nodePort: 30012 
  type: NodePort 
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: php-fpm-service 
spec: 
  selector: 
    app: nginx 
  ports: 
    - protocol: TCP 
      port: 9000 
      targetPort: 9000 
  type: ClusterIP 
--- 
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: nginx-config 
data: 
  nginx.conf: | 
    events {} 
    http { 
      server { 
        listen 8091; 
        index index.html index.htm index.php; 
        root  /var/www/html; 
        location ~ \.php$ { 
          include fastcgi_params; 
          fastcgi_param REQUEST_METHOD $request_method; 
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
          fastcgi_pass php-fpm-service:9000; 
        } 
      } 
    } 
--- 
apiVersion: v1 
kind: Pod 
metadata: 
  name: nginx-phpfpm 
  labels: 
    app: nginx 
spec: 
  containers: 
    - name: nginx-container 
      image: nginx:latest 
      imagePullPolicy: IfNotPresent 
      volumeMounts: 
        - name: nginx-config-volume 
          mountPath: /etc/nginx/nginx.conf 
          subPath: nginx.conf 
        - name: shared-files 
          mountPath: /var/www/html 
    - name: php-fpm-container 
      image: php:8.2-fpm-alpine 
      volumeMounts: 
        - name: shared-files 
          mountPath: /var/www/html 
  restartPolicy: Always 
  volumes: 
    - name: shared-files 
      emptyDir: {} 
    - name: nginx-config-volume 
      configMap: 
        name: nginx-config 
  
 
thor@jump_host ~$ kubectl create -f depl.yaml 
thor@jump_host ~$ kubectl cp /opt/index.php nginx-phpfpm:/var/www/html -c nginx-container 
thor@jump_host ~$ kubectl exec -it  nginx-phpfpm -c nginx-container -- sh 
# cd /var/www/html 
# cat index.php 
It works!#  
```