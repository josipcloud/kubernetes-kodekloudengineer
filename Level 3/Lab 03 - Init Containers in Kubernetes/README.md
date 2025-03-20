# Level 3 - Lab 03 - Init Containers in Kubernetes
## Task
1. Create a `Deployment` named `ic-deploy-devops`.

2. Configure `spec` as replicas should be `1`, labels `app` should be `ic-devops`, template's metadata labels `app` should be the same `ic-devops`.

3. The `initContainers` should be named `ic-msg-devops`m use image `ubuntu`, preferably with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'echo Init Done - Welcome to xFusionCorp Industries > /ic/media'`. The volume mount should be named `ic-volume-devops` and mount path should be `/ic`.

4. Main container should be named `ic-main-devops`m use image `ubuntu`, preferably with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'while true; do cat /ic/media; sleep 5; done'`. The volume mount should be named `ic-volume-devops` and mount path should be `/ic`.

5. Volume to be named `ic-volume-devops` and it should be an emptyDir type.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=BkImNsD-vgk)

```
thor@jump_host ~$ kubectl create deploy ic-deploy-devops --image=ubuntu:latest --dry-run=client -o yaml > deploy.yaml 
thor@jump_host ~$ vi deploy.yaml 
thor@jump_host ~$ cat deploy.yaml  

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  labels: 
    app: ic-devops 
  name: ic-deploy-devops 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: ic-devops 
  strategy: {} 
  template: 
    metadata: 
      labels: 
        app: ic-devops 
    spec: 
      initContainers: 
      - image: ubuntu:latest 
        name: ic-msg-devops 
        command: ['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/media'] 
        volumeMounts: 
        - mountPath: /ic 
          name: ic-volume-devops  
      containers: 
      - image: ubuntu:latest 
        name: ic-main-devops 
        command: ['/bin/bash', '-c', 'while true; do cat /ic/media; sleep 5; done'] 
        volumeMounts: 
        - mountPath: /ic 
          name: ic-volume-devops 
      volumes: 
      - name: ic-volume-devops 
        emptyDir: 
          sizeLimit: 100Mi 
```