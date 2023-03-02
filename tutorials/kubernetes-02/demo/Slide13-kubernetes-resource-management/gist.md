# Inspecting Pod Resource Usage
## Relevant Documentation
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods
## Lesson Reference
Install Kubernetes Metrics Server.
```
kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml
```
Verify that the metrics server is responsive. Note that it may take a few minutes for the metrics server to become responsive to
requests.
```
kubectl get --raw /apis/metrics.k8s.io/
```
Create a pod to monitor.
```
vim pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: metrics-test
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
```
```
kubectl apply -f pod.yml
```

Use  kubectl top  to view resource usage by pod.

```
kubectl top pod
```

Sort output with  --sort-by .

```
kubectl top pod --sort-by cpu
```

Filter output by label with  --selector .

```
kubectl top pod --selector app=metrics-test
```
---
---

# Managing Container Resources
## Relevant Documentation
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container
## Lesson Reference
Create a pod with resource requests that exceed aviable node resources.
```
vim big-request-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: big-request-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    resources:
      requests:
        cpu: "10000m"
        memory: "128Mi"
```

```
kubectl create -f big-request-pod.yml
```

Check the pod status. It should never leave the Pending state since no worker nodes have enough resources to meet the
request.

```
kubectl get pod big-request-pod
````

Create a pod with resource requests and limits.
```
vim resource-pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    resources:
      requests:
        cpu: "250m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

```
kubectl create -f resource-pod.yml
```
Check the pod status.
```
kubectl get pods
```
---
---
# Monitoring Container Health with Probes
## Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
## Lesson Reference

Create a pod with a command-based liveness probe.
```
vim liveness-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 3600; done']
    livenessProbe:
      exec:
        command: ["echo", "Hello, world!"]
      initialDelaySeconds: 5
      periodSeconds: 5
```
```
kubectl create -f liveness-pod.yml
```
Check the pod status.
```
kubectl get pod liveness-pod
```
Create a pod with an http-based liveness probe.
```
vim liveness-pod-http.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod-http
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```
```
kubectl create -f liveness-pod-http.yml
```
Check the pod status.

```
kubectl get pod liveness-pod-http
```

Create a pod with a startup probe.
```
vim startup-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    startupProbe:
      httpGet:
        path: /
        port: 80
      failureThreshold: 30
      periodSeconds: 10
```
```
kubectl create -f startup-pod.yml
```
Check the pod status.
```
kubectl get pod startup-pod
```
Create a pod with a readiness probe.
```
vim readiness-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```
```
kubectl create -f readiness-pod.yml
```
Check the pod status.

```
kubectl get pod readiness-pod
```
---
---
# Building Self-Healing Pods with Restart Policies
## Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

## Lesson Reference
Create a pod with the Always restart policy that completes after a few seconds.
```
vim always-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: always-pod
spec:
  restartPolicy: Always
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 10']
```
```
kubectl create -f always-pod.yml
````

Check the pod status. You should see it restarting after every 10-second completion.
```
kubectl get pod always-pod
```
Create a pod with the OnFailure restart policy.
```
vim onfailure-pod.yml
````
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: onfailure-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 10']
```
```
kubectl create -f onfailure-pod.yml
```
Check the pod status. Note that the pod does not restart because it completed successfully.
```
kubectl get pod onfailure-pod
```
Delete, modify, and recreate the pod so that it fails.
```
vim onfailure-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: onfailure-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
```
```
kubectl create -f onfailure-pod.yml
```
Check the pod status. Note that the pod restarts because it exited with an error code.
```
kubectl get pod onfailure-pod
```
Create a pod with the Never restart policy that completes after a few seconds.
```
vim never-pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: never-pod
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'sleep 10; this is a bad command that will fail']
```
```
kubectl create -f never-pod.yml
```
Check the pod status. You should see that it does not restart after completing.

```
kubectl get pod never-pod
```