# Level 3 - Lab 04 - Persistent Volumes in Kubernetes
## Task
1. Create a `PersistentVolume` named `pv-datacenter`. Configure the `spec` as storage class should be `manual`, set capacity to `4Gi`, set access mode to `ReadWriteOnce`, volume type should be `hostPath` and set path to `/mnt/dba` (this directory is already created, you might not be able to access it directly, so you need not to worry about it).

2. Create a `PersistentVolumeClaim` named `pvc-datacenter`. Configure the `spec` as storage class should be `manual`, request `2Gi` of the storage, set access mode to `ReadWriteOnce`.

3. Create a `pod` named `pod-datacenter`, mount the persistent volume you created with claim name `pvc-datacenter` at document root of the web server, the container within the pod should be named `container-datacenter` using image `httpd` with `latest` tag (remember to mention the tag i.e `httpd:latest`).

4. Create a node port type service named `web-datacenter` using node port `30008` to expose the web server running within the pod.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=EPP9dbt_BsQ)

```
thor@jump_host ~$ vi pv.yaml 
thor@jump_host ~$ cat pv.yaml 

apiVersion: v1 
kind: PersistentVolume 
metadata: 
  name: pv-datacenter 
spec: 
  capacity: 
    storage: 4Gi 
  volumeMode: Filesystem 
  accessModes: 
    - ReadWriteOnce 
  storageClassName: manual 
  hostPath: 
    path: /mnt/dba 

thor@jump_host ~$ vi pvc.yaml 
thor@jump_host ~$ cat pvc.yaml 

apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: pvc-datacenter 
spec: 
  storageClassName: manual 
  accessModes: 
    - ReadWriteOnce 
  resources: 
    requests: 
      storage: 2Gi 
```

Useful link: [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/).

```
thor@jump_host ~$ kubectl create -f pvc.yaml 
persistentvolumeclaim/pvc-datacenter created 

thor@jump_host ~$ kubectl get pvc 
NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE 
pvc-datacenter   Bound    pv-datacenter   4Gi        RWO            manual         2s 

thor@jump_host ~$ kubectl create -f pv.yaml 
thor@jump_host ~$ kubectl get pv 
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE 
pv-datacenter   4Gi        RWO            Retain           Bound    default/pvc-datacenter   manual                  23m 

thor@jump_host ~$ vi pod.yaml 
thor@jump_host ~$ cat pod.yaml 

apiVersion: v1 
kind: Pod 
metadata: 
  labels: 
    run: pod-datacenter 
  name: pod-datacenter 
spec: 
  containers: 
  - image: httpd:latest 
    name: container-datacenter 
    volumeMounts: 
      - mountPath: /var/www/html 
        name: pvc-datacenter 
  volumes: 
    - name: pvc-datacenter 
      persistentVolumeClaim: 
        claimName: pvc-datacenter  
```
**Note**: In the context of a web server, the "document root" refers to the directory where the web server looks for files to serve. For example, in the case of Apache HTTP Server (httpd), the default document root is typically set to `/var/www/html/`. 


Useful link: [Service](https://kubernetes.io/docs/concepts/services-networking/service/ ).


```
thor@jump_host ~$ vi svc.yaml 
thor@jump_host ~$ cat svc.yaml 

apiVersion: v1 
kind: Service 
metadata: 
  name: web-datacenter 
spec: 
  type: NodePort 
  selector: 
    run: pod-datacenter 
  ports: 
    - port: 80 
      targetPort: 80 
      nodePort: 30008 
```