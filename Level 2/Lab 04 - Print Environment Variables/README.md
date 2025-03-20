# Level 2 - Lab 04 - Print Environment Variables
## Task
1. Create a `pod` named `print-envars-greeting`.

2. Configure spec like this: Container name should be `print-env-container` and use `bash` image.

3. Create three environment variables:

     a. `GREETING` with value `Welcome to`.
     
     b. `COMPANY` with value `DevOps`.

     c. `GROUP` with value `Datacenter`.

4. Use command to echo `["$(GREETING) $(COMPANY) $(GROUP)"]` message.

5. You can check the output using `kubectl logs -f print-envars-greeting` command.

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=wyZcW7a7pFs)

```
thor@jump_host ~$ vi pod.yaml  
thor@jump_host ~$ cat pod.yaml  

apiVersion: v1 
kind: Pod 
metadata: 
  labels: 
    run: print-envars-greeting 
  name: print-envars-greeting 
spec: 
  containers: 
  - image: bash 
    name: print-env-container 
    env: 
    - name: GREETING 
      value: "Welcome to" 
    - name: COMPANY 
      value: "DevOps" 
    - name: GROUP 
      value: "Datacenter" 
    command: 
    - "sh" 
    - "-c" 
    - "echo $GREETING $COMPANY $GROUP" 

thor@jump_host ~$ kubectl get pods 
NAME                    READY   STATUS              RESTARTS   AGE 
print-envars-greeting   0/1     ContainerCreating   0          2s 

thor@jump_host ~$ kubectl get pods 
NAME                    READY   STATUS      RESTARTS     AGE 
print-envars-greeting   0/1     Completed   1 (2s ago)   3s 

thor@jump_host ~$ kubectl get pods 
NAME                    READY   STATUS             RESTARTS     AGE 
print-envars-greeting   0/1     CrashLoopBackOff   1 (3s ago)   5s 

thor@jump_host ~$ kubectl logs -f print-envars-greeting  
Welcome to DevOps Datacenter 
```