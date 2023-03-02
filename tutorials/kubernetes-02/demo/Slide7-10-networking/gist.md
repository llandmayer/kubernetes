## Reference Documents
https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/

## We use Amazon VPC Container Network Interface (CNI)
For the purpose of this lesson lets install a new CNI Plugin, but first lets recreate our cluster.

```
minikube delete
```
And start the cluster with cni plugin enabled.
```
minikube start --network-plugin=cni --enable-default-cni
```

Now lets install Cilium Plugin

https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-cilium


## Kubernetes DNS
DNS runs in kube-system namespace

How it resolves domains:
`<pod-ip-address>.<namespace-name>.pod.cluster.local`

---
---
# K8s DNS
## Relevant Documentation
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
## Lesson Reference
Create a descriptor that will set up some sample Pods which you can use to test DNS functionality.
```
vim dnstest-pods.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-dnstest
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
---
apiVersion: v1
kind: Pod
  metadata:
    name: nginx-dnstest
spec:
  containers:
  - name: nginx
    image: nginx:1.19.2
    ports:
    - containerPort: 80
```
```
kubectl create -f dnstest-pods.yml
```
Get the IP address of the nginx-dnstest Pod.
```
kubectl get pods nginx-dnstest -o wide
```
Verify that you can reach the nginx-dnstest over the cluster network using its IP address.
```
kubectl exec busybox-dnstest -- curl <nginx-dnstest IP address>
```
Use the busybox-dnstest Pod to look up the DNS record for the nginx-dnstest Pod.
```
kubectl exec busybox-dnstest -- nslookup <nginx-dnstest-ip>.default.pod.cluster.local
```
Verify that you can reach the nginx-dnstest Pod using its domain name.
```
kubectl exec busybox -- curl <nginx-dnstest-ip>.default.pod.cluster.local
```
---
---
# Using NetworkPolicies
Relevant Documentation
https://kubernetes.io/docs/concepts/services-networking/network-policies/#the-networkpolicy-resource

Lesson Reference

Create a new namespace.
```
kubectl create namespace np-test
```
Add a label to the Namespace.
```
kubectl label namespace np-test team=np-test
```
Create a web server Pod.
```
vim np-nginx.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: np-nginx
  namespace: np-test
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```
```
kubectl create -f np-nginx.yml
```
Create a client Pod.
```
vim np-busybox.yml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: np-busybox
  namespace: np-test
  labels:
    app: client
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 5; done']
```
```
kubectl create -f np-busybox.yml
```
Get the IP address of the nginx Pod and save it to an environment variable.
```
kubectl get pods -n np-test -o wide
NGINX_IP=<np-nginx Pod IP>
```
Attempt to access the nginx Pod from the client Pod. This should succeed since no NetworkPolicies select the client Pod.
```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```
Create a NetworkPolicy that selects the Nginx Pod.
```
vim my-networkpolicy.yml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-networkpolicy
  namespace: np-test
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress

```
```
kubectl create -f my-networkpolicy.yml
```
This NetworkPolicy will block all traffic to and from the Nginx Pod. Attempt to communicate with the Pod again. It should fail
this time.
```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```

Oh HangOn!!! Why isn't this blocking traffic?!?! 

We need to understand what plugin we are using to CNI. And the current one doesn't support network policies.

So Lets restart our minikube using the Calico Plugin.
```
minikube stop
minikube start --network-plugin=cni --cni=calico
```
Check if Calico was actually installed.
````
watch kubectl get pods -l k8s-app=calico-node -A
````
All good? Lets try again.

```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```

Modify the NetworkPolicy so that it allows incoming traffic on port 80 for all Pods in the np-test Namespace.
```
kubectl edit networkpolicy -n np-test my-networkpolicy
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-networkpolicy
  namespace: np-test
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: np-test
    ports:
    - port: 80
      protocol: TCP
```
Attempt to communicate with the Pod again. This time, it should work!
```
kubectl exec -n np-test np-busybox -- curl $NGINX_IP
```