# Level 3 - Lab 05 - Manage Secrets in Kubernetes
## Task
1. We already have a secret key file `news.txt` under `/opt` location on `jump host`. Create a `generic secret` named `news`, it should contain the password/license-number present in `news.txt` file.

2. Also create a `pod` named `secret-xfusion`.

3. Configure pod's `spec` as container name should be `secret-container-xfusion`, image should be `ubuntu` preferably with `latest` tag (remember to mention the tag with image). Use `sleep` command for container so that it remains in running state. Consume the created secret and mount it under `/opt/cluster` within the container.

4. To verify you can exec into the container `secret-container-xfusion`, to check the secret key under the mounted path `/opt/cluster`. Before hitting the `Check` button please make sure pod/pods are in running state, also validation can take some time to complete so keep patience.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=4Tu-jrL713Q)

```
thor@jump_host ~$ cat /opt/news.txt  
5ecur3 

thor@jump_host ~$ kubectl create secret --help 
thor@jump_host ~$ kubectl create secret generic news --from-file=/opt/news.txt 

thor@jump_host ~$ vi pod.yaml 
thor@jump_host ~$ cat pod.yaml 

apiVersion: v1 
kind: Pod 
metadata: 
  creationTimestamp: null 
  labels: 
    run: secret-xfusion 
  name: secret-xfusion 
spec: 
  containers: 
  - image: ubuntu:latest 
    name: secret-container-xfusion 
    command: ["sleep", "2000"] 
    volumeMounts: 
      - name: secret-volume 
        mountPath: "/opt/cluster" 
  volumes: 
    - name: secret-volume 
      secret: 
        secretName: news 
```