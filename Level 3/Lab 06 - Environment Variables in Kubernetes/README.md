# Level 3 - Lab 06 - Environment Variables in Kubernetes
## Task
1. Create a `pod` named `envars`.

2. Container name should be `fieldref-container`, use image `busybox` preferable `latest` tag, use command `'sh', '-c'` and args should be:

   `'while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;'`

**(Note: please take care of indentations)**

3. Define `Four` environment variables as mentioned below:

   a) The first `env` should be named `NODE_NAME`, set valueFrom fieldref and fieldPath should be `spec.nodeName`.

   b) The second `env` should be named `POD_NAME`, set valueFrom fieldref and fieldPath should be `metadata.name`.

   c) The third `env` should be named `POD_IP`, set valueFrom fieldref and fieldPath should be `status.podIP`.

   d) The fourth `env` should be named `POD_SERVICE_ACCOUNT`, set valueFrom fieldref and fieldPath should be `spec.serviceAccountName`.

4. Set restart policy to `Never`.

5. To check the output, exec into the pod and use `printenv` command.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=qn9hdb9396Y)

---

Useful link 1: [Expose Pod Information to Containers Through Environment Variables](https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/).

Useful link 2: [Pod Manifest example](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/pods/inject/dapi-envars-pod.yaml).

```
thor@jump_host ~$ vi pod.yaml  
thor@jump_host ~$ cat pod.yaml  

apiVersion: v1 
kind: Pod 
metadata: 
  labels: 
    run: envars 
  name: envars 
spec: 
  containers: 
  - image: busybox:latest 
    name: fieldref-container 
    command: 
    - "sh" 
    - "-c" 
    - "while true; do echo -en '/n'; printenv NODE_NAME POD_NAME; printenv POD_IP POD_SERVICE_ACCOUNT; sleep 10; done;" 
    env: 
      - name: NODE_NAME 
        valueFrom: 
          fieldRef: 
            fieldPath: spec.nodeName 
      - name: POD_NAME 
        valueFrom: 
          fieldRef: 
            fieldPath: metadata.name 
      - name: POD_IP 
        valueFrom: 
          fieldRef: 
            fieldPath: status.podIP 
      - name: POD_SERVICE_ACCOUNT 
        valueFrom: 
          fieldRef: 
            fieldPath: spec.serviceAccountName 
  restartPolicy: Never 

thor@jump_host ~$ kubectl logs envars  
/nkodekloud-control-plane 
envars 
10.244.0.5 
default 
/nkodekloud-control-plane 
envars 
10.244.0.5 
default 
/nkodekloud-control-plane 
envars 
10.244.0.5 
default 

thor@jump_host ~$ kubectl exec -it envars -- sh 
/ #  
/ # printenv 
POD_IP=10.244.0.5 
KUBERNETES_SERVICE_PORT=443 
KUBERNETES_PORT=tcp://10.96.0.1:443 
HOSTNAME=envars 
SHLVL=1 
HOME=/root 
NODE_NAME=kodekloud-control-plane 
TERM=xterm 
POD_NAME=envars 
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1 
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
POD_SERVICE_ACCOUNT=default 
KUBERNETES_PORT_443_TCP_PORT=443 
KUBERNETES_PORT_443_TCP_PROTO=tcp 
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443 
KUBERNETES_SERVICE_PORT_HTTPS=443 
KUBERNETES_SERVICE_HOST=10.96.0.1 
PWD=/ 
```