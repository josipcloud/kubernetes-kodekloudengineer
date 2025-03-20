# Lab 05 - Rolling Updates in Kubernetes
## Task
Perform a rolling update for the application and incorporate `nginx:1.18` image. The deployment name is `nginx-deployment`. Make sure all pods are up and running after the update.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=XBONTnUQs9E)

```
thor@jump_host ~$ kubectl edit deployment nginx-deployment  
deployment.apps/nginx-deployment edited 

(change image section to nginx:1.18)
```