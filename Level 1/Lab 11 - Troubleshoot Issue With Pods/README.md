# Lab 11 - Troubleshoot Issue with Pods
## Task
1. There is a pod named `webserver` and the container under it is named `httpd-container`. It is using image `httpd:latest`.

2. There is a sidecar container as well named `sidecar-container` which is using `ubuntu:latest` image.

Look into the issue and fix it, make sure pod is in `running` state and you are able to access the app.


## Solution
Youtube video solution can be found [here.](https://www.youtube.com/watch?v=s56qzCEmSzg)

```
thor@jump_host ~$ kubectl describe pod webserver

(issue was with the image name, there was an error, image was set to "httpd:latests", instead of "httpd:latest".

thor@jump_host ~$ kubectl edit pod webserver

(edit image section)
```