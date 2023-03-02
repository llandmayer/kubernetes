## Relevant Documentation
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
## Lesson Reference

List namespaces in the cluster.
```
kubectl get namespaces
```
Specify a namespace when listing other objects such as pods.
```
kubectl get pods -n kube-system
```
Create a namespace.
```
kubectl create namespace <my-namespace>
```
Get all pods in all namespaces
```
kubetctl get pods --all-namespaces
```

## Bonus
We can also prevent access between specific namespaces using `NetworkPolicy`
for more on it check https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource

And also, we'll hear more about it in the Networking Lesson.