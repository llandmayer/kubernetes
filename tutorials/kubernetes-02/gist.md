# Kubernetes 101

#### K8s commands

##### create kind cluster
    `kind create cluster`

##### get basic info about k8s components
    `kubectl get node`
    `kubectl get pod`
    `kubectl get svc`
    `kubectl get all`

##### get extended info about components
    `kubectl get pod -o wide`
    `kubectl get node -o wide`

##### get detailed info about a specific component
    `kubectl describe svc {svc-name}`
    `kubectl describe pod {pod-name}`

##### get application logs
    `kubectl logs {pod-name}`

### To apply everything.
`kubectl apply -f demo/ -n default`

## Install docker
### Linux and Mac
`brew install derailed/k9s/k9s`

Install brew on Linux https://docs.brew.sh/Homebrew-on-Linux