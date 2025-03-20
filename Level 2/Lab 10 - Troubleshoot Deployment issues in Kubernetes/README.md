# Level 2 - Lab 10 - Troubleshoot Deployment issues in Kubernetes 
## Task
The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix it.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=4fpFkpvIwwo)

```
thor@jump_host ~$ kubectl describe deploy redis-deployment  

There were two typos in the deployment:
...
 Image:      redis:alpin -----> instead of "redis:alpine"  
...
   config: 
    Type:      ConfigMap (a volume populated by a ConfigMap) 
    Name:      redis-cofig -----> instead of "redis-config" 
...

thor@jump_host ~$ kubectl edit deploy redis-deployment  

(fix errors above)

```