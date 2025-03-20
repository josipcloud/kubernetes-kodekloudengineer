# Lab 06 - Rollback a Deployment in Kubernetes 
## Task
There is a deployment named `nginx-deployment`. Roll it back to the previous revision.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=bwRQt02y6H0)

```
thor@jump_host ~$ kubectl rollout status deployment nginx-deployment 

thor@jump_host ~$ kubectl rollout history deployment nginx-deployment 

thor@jump_host ~$ kubectl rollout undo deployment nginx-deployment --to-revision=<revision-number>

```