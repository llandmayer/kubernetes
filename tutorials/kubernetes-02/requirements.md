# Requirements

## Install docker
### Linux and Mac
follow the instruction in the docker website https://docs.docker.com/get-docker/

## Install minikube
https://minikube.sigs.k8s.io/docs/start/

## Install kubectl
### Linux
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

### Mac
```bash
brew install kubectl
```
