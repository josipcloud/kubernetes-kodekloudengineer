# Level 2 - Lab 09 - Deploy Node App on Kubernetes
## Task
1. Create a deployment using `gcr.io/kodekloud/centos-ssh-enabled:node` image, replica count must be `2`.

2. Create a service to expose this app, the service type must be `NodePort`, targetPort must be `8080` and nodePort should be `30012`.

3. Make sure all the pods are in `Running` state after the deployment.

4. You can check the application by clicking on `NodeApp` button on top bar.

`You can use any labels as per your choice.`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=ppRjI2pMheo)

```
thor@jump_host ~$ kubectl create deployment tomcat --image=gcr.io/kodekloud/centos-ssh-enabled:node --replicas=2
deployment.apps/tomcat created 

thor@jump_host ~$ kubectl expose deployment tomcat --type="NodePort" --port=800 
service/tomcat exposed 

thor@jump_host ~$ kubectl edit svc tomcat 
service/tomcat edited 

(edit "nodePort: 30012" , "port: 8080" , "targetPort: 8080")

(Check access via web button, it should work now)
```