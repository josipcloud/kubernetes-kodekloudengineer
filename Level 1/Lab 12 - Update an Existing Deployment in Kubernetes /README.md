# Lab 12 - Update an Existing Deployment in Kubernetes 
## Task
We already have a deployment named `nginx-deployment` and service named `nginx-service`. Some changes need to be made in this deployment and service, make sure not to delete the deployment and service.

1. Change the service nodeport from `30008` to `32165`.

2. Change the replicas count from `1` to `5`.

3. Change the image from `nginx:1.17` to `nginx-latest`.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=GLhuuhRF2lc)

```
thor@jump_host ~$ kubectl edit deployment nginx-deployment
thor@jump_host ~$ kubectl edit svc nginx-service
```