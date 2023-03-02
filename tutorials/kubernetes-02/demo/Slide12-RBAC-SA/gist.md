## Relevant Documentation
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

## Lesson Reference
### - RBAC
Create a Role spec file.
```
vim role.yml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "watch", "list"]
```

Create the Role.

```
kubectl apply -f role.yml
```

Bind the role to the dev user.
```
vim rolebinding.yml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader
  namespace: default
subjects:
- kind: User
  name: dev
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Create the RoleBinding.
```
kubectl apply -f rolebinding.yml
```

### - Service Accounts
Create a basic ServiceAccount.
```
vim my-serviceaccount.yml
```
```yaml
apiVersion: v1
kind: ServiceAccount
  metadata:
    name: my-serviceaccount
```
```
kubectl create -f my-serviceaccount.yml
```
Create a ServiceAccount with an imperative command.
```
kubectl create sa my-serviceaccount2 -n default
```

View your ServiceAccount.
```
kubectl get sa
```

Attach a Role to the ServiceAccount with a RoleBinding.
```
vim sa-pod-reader.yml
```
```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-pod-reader
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```
kubectl create -f sa-pod-reader.yml
```
Get additional information for the ServiceAccount.
```
kubectl describe sa my-serviceaccount
```
