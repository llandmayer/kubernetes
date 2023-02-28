## Relevant Documentation
- https://kubernetes.io/docs/reference/kubectl/overview/
## Lesson Reference

 Create a pod.

```
vim pod.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: busybox
      image: radial/busyboxplus:curl
      command: ['sh', '-c', 'while true; do sleep 3600; done']
```
```
kubectl apply -f pod.yml
```

Get a list of pods.

```
kubectl get pods
```

Experiment with various output formats.

```
kubectl get pods -o wide
kubectl get pods -o json
kubectl get pods -o yaml
```

Sort results.

```
kubectl get pods -o wide --sort-by .spec.nodeName
```

Filter results by a label.

```
kubectl get pods -n kube-system --selector k8s-app=calico-node
```

Describe a pod.

```
kubectl describe pod my-pod
```

Test create/apply. Note that create will only work if the object does not already exist.

```
kubectl create -f pod.yml
kubectl apply -f pod.yml
```

Execute a command inside a pod.

```
kubectl exec my-pod -c busybox -- echo "Hello, world!"
```

Delete a pod.
```
kubectl delete pod my-pod
```