# Lab 02 - Create Deployments in Kubernetes Cluster
## Task
Create a deployment named `httpd` to deploy the application `httpd` using the image `httpd:latest` (remember to mention the tag as well)

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=QfBcebWqG3Q)

```
thor@jump_host ~$ kubectl create deployment httpd --image=httpd:latest 
```