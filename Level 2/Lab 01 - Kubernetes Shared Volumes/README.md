# Level 2 - Lab 01 - Kubernetes Shared Volumes
## Task
1. Create a pod named `volume-share-nautilus`.

2. For the first container, use image `fedora` with `latest` tag and remember to mention the tag i.e `fedora:latest`. Container should be named `volume-container-nautilus-1` and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/blog`.

3. For the second container, use image `fedora` with the `latest` tag and remember to mention the tag i.e `fedora:latest`. Container should be named `volume-container-nautilus-2` and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/demo`.

4. Volume name should be `volume-share` of type `emptyDir`.

5. After creating the pod, exec into the first container i.e `volume-container-nautilus-1`, and just for testing create a file `blog.txt` with any content under the mounted path of first container i.e `/tmp/blog`.

6. The file `blog.txt` should be present under the mounted path `/tmp/demo` on the second container `volume-container-nautilus-2` as well, since they are using a shared volume.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=gl7qDHrMDOI)

```
thor@jump_host ~$ vi nautilus.yaml  
thor@jump_host ~$ cat nautilus.yaml  

apiVersion: v1 
kind: Pod 
metadata: 
  labels: 
    run: volume-share-nautilus 
  name: volume-share-nautilus 
spec: 
  containers: 
  - image: fedora:latest 
    name: volume-container-nautilus-1 
    command:  
    - "sleep" 
    - "2000" 
    volumeMounts: 
    - name: volume-share 
      mountPath: /tmp/blog 
  - image: fedora:latest 
    name: volume-container-nautilus-2 
    command: 
    - "sleep" 
    - "2000" 
    volumeMounts: 
    - name: volume-share 
      mountPath: /tmp/demo 
  volumes: 
  - name: volume-share 
    emptyDir: 
      sizeLimit: 100Mi  

thor@jump_host ~$ kubectl get pods 

NAME                    READY   STATUS    RESTARTS   AGE 
volume-share-nautilus   2/2     Running   0          3s 

thor@jump_host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- sh 
sh-5.2# cd /tmp/blog 
sh-5.2# pwd 
/tmp/blog 
sh-5.2# vi blog.txt 
sh-5.2# exit 
exit 

thor@jump_host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- sh 
sh-5.2# cd /tmp/demo 
sh-5.2# ls 
blog.txt 
sh-5.2# cat blog.txt  
text 
sh-5.2# exit 
exit  
```