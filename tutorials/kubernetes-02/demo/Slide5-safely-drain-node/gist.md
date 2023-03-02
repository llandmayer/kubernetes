## Relevant Documentation
- https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

## Requirements
    - minikube
    - kubectl
## Lesson Reference
Begin by making sure minikube is running
```bash
╭─leandro.landmeyer in ~/Repos/tmp/kubernetes on main✘✘✘
╰─± minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

or starting minikube.

```
minikube start
```

In this lesson we'll need to have some extra worker nodes, for this reason make sure you have as much available memory as possible.

Let's add some extra nodes to our minikube cluster.
```bash
minikube node add && minikube node add
```

Now let's create some objects. We will examine how these objects are affected by the drain process.

First, create a pod.
```bash
vim pod.yml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-pod
spec:
    containers:
    - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
    restartPolicy: OnFailure
```

```bash
kubectl apply -f pod.yml
```

Create a deployment with two replicas.
```bash
vim deployment.yml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: my-deployment
    labels:
        app: my-deployment
spec:
    replicas: 2
    selector:
        matchLabels:
            app: my-deployment
    template:
        metadata:
        labels:
            app: my-deployment
        spec:
            containers:
            - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
```

```bash
kubectl apply -f deployment.yml
```

Get a list of pods. You should see the pods you just created (including the two replicas from the deployment). Take node of
which node these pods are running on.
```
kubectl get pods -o wide
```
Drain the node which the my-pod pod is running.
```
kubectl drain <node name> --ignore-daemonsets --force
```

Check your list of pods again. You should see the deployment replica pods being moved to the remaining node. The regular
pod will be deleted.

```
kubectl get pods -o wide
```

Uncordon the node to allow new pods to be scheduled there again.

```
kubectl uncordon <node name>
```

Let's Clean Up, run:
```bash
minikube node delete
```
