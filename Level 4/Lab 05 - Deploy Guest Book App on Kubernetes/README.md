# Level 4 - Lab 05 - Deploy Guest Book App on Kubernetes 
## Task
**BACK-END TIER**

1. Create a deployment named `redis-master` for Redis master.


    a) Replicas count should be `1`.

    b) Container name should be `master-redis-xfusion` and it should use image `redis`.

    c) Request resources as `CPU` should be `100m` and `Memory` should be `100Mi`.

    d) Container port should be redis default port i.e `6379`.

2. Create a service named `redis-master` for Redis master, Port and targetPort should be Redis default port i.e `6379`.

3. Create another deployment named `redis-slave` for Redis slave.

    a) Replicas count should be `2`.

    b) Container name should be `slave-redis-xfusion` and it should use `gcr.io/google_samples/gb-redisslave:v3` image.

    c) Requests resources as `CPU` should be `100m` and `Memory` should be `100Mi`.

    d) Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`.

    e) Container port should be Redis default port i.e `6379`.

4. Create another service named `redis-slave`. It should use Redis default port i.e `6379`.

**FRONT END TIER**

1. Create a deployment named `frontend`.

    a) Replicas count should be `3`.

    b) Container name should be `php-redis-xfusion` and it should use `gcr.io/google-samples/gb-frontend@sha256:cbc8ef4b0a2d0b95965e0e7dc8938c270ea98e34ec9d60ea64b2d5f2df2dfbbf` image.

    c) Request resouces as `CPU` should be `100m` and `Memory` should be `100Mi`.

    d) Define an environment. variable named as `GET_HOSTS_FROM` and its value should be `dns`

    e) Container port should be `80`.

2. Create a service named `frontend`. Its `type` should be `NodePort`, port should be `80` and its `nodePort` should be `30009`.

Finally, you can check the `guestbook app` by clicking on `App` button.

`You can use any labels as per your choice.`

## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=CuuXFn1jjeA)


```
thor@jump_host ~$ vi guestbookapp.yaml
thor@jump_host ~$ cat guestbookapp.yaml

apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: redis-master 
  labels: 
    app: redis-master 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: redis-master 
  template: 
    metadata: 
      name: redis-master 
      labels: 
        app: redis-master 
    spec: 
      containers: 
        - name: master-redis-devops  
          image: redis 
          imagePullPolicy: IfNotPresent 
          resources: 
            requests: 
              cpu: "100m" 
              memory: "100Mi" 
          ports: 
            - containerPort: 6379 
      restartPolicy: Always 
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: redis-master 
spec: 
  selector: 
    app: redis-master 
  ports: 
    - protocol: TCP 
      port: 6379 
      targetPort: 6379 
  type: ClusterIP 
--- 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: redis-slave 
  labels: 
    app: redis-slave 
spec: 
  replicas: 2 
  selector: 
    matchLabels: 
      app: redis-slave 
  template: 
    metadata: 
      name: redis-slave 
      labels: 
        app: redis-slave 
    spec: 
      containers: 
        - name: slave-redis-devops 
          image: gcr.io/google_samples/gb-redisslave:v3 
          imagePullPolicy: IfNotPresent 
          resources: 
            requests: 
              cpu: "100m" 
              memory: "100Mi" 
          env: 
            - name: GET_HOSTS_FROM 
              value: dns 
          ports: 
            - containerPort: 6379 
      restartPolicy: Always 
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: redis-slave 
spec: 
  selector: 
    app: redis-slave 
  ports: 
    - protocol: TCP 
      port: 6389 
      targetPort: 6379 
  type: ClusterIP 
--- 
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: frontend 
  labels: 
    app: frontend 
spec: 
  replicas: 3 
  selector: 
    matchLabels: 
      app: frontend 
  template: 
    metadata: 
      name: frontend 
      labels: 
        app: frontend 
    spec: 
      containers: 
        - name: php-redis-devops 
          image: gcr.io/google-samples/gb-frontend:v4 
          imagePullPolicy: IfNotPresent 
          resources: 
            requests: 
              cpu: "100m" 
              memory: "100Mi" 
          env: 
            - name: GET_HOSTS_FROM 
              value: dns 
          ports: 
            - containerPort: 80 
      restartPolicy: Always
--- 
apiVersion: v1 
kind: Service 
metadata: 
  name: frontend 
spec: 
  selector: 
    app: frontend 
  ports: 
    - protocol: TCP 
      port: 80 
      targetPort: 80 
      nodePort: 30009 
  type: NodePort
```