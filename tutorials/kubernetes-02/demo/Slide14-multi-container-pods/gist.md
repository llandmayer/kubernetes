## Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/#using-pods
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/
## Lesson Reference
Create a simple multi-container Pod.
```
vim multi-container-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: redis
    image: redis
  - name: couchbase
    image: couchbase
```
```
kubectl create -f multi-container-pod.yml
```
View your pod's status.
```
kubectl get pod multi-container-pod
```
Create a multi-container Pod that uses shared storage.
```
vim sidecar-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo logs data > /output/output.log; sleep 5; done']
    volumeMounts:
    - name: sharedvol
      mountPath: /output
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'tail -f /input/output.log']
    volumeMounts:
    - name: sharedvol
      mountPath: /input
  volumes:
  - name: sharedvol
    emptyDir: {}
```
```
kubectl create -f sidecar-pod.yml
```
View the logs for the sidecar container.
```
kubectl logs sidecar-pod -c sidecar
```

---
---
# Init Containers
## Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
## Lesson Reference
Create a pod with an init container that delays startup by 30 seconds.
```
vim init-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
  initContainers:
  - name: delay
    image: busybox
    command: ['sleep', '30']
```
```
kubectl create -f init-pod.yml
```
Check the pod status. You should see it remain in an Init state until the 30-second delay passes, then it should fully start up.
```
kubectl get pod init-pod
```