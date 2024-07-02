# Task Devops

### Kubernetes prepare

1. Operation system (macOS)

Kubernetes install on local Machine (To install microK8S following guide https://microk8s.io/docs/install-macos)

##### prerequisite

 - Homebrew is installed

##### install need tools

```sh
brew install ubuntu/microk8s/microk8s
brew install helm
```

#### install micro

```sh
microk8s install
```

##### validate microk8s install

```sh
microk8s status --wait-ready
```

Note: Make easy to use microk8s kubectl command  on terminal
```sh
alias kubectl='microk8s kubectl'
```

##### setup microk8s

Enable addons

```sh
microk8s enable dns
microk8s enable rbac
microk8s enable hostpath-storage
microk8s enable metallb (10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111)
```

#### Install argoCD use official guide (https://argo-cd.readthedocs.io/en/stable/getting_started/)

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

##### Validate argoCD install on microK8S cluster

```sh
kubectl get pod -n argocd
kubectl get svc -n argocd
```

#### Starting use argoCD ui on Local Machine

```sh
kubectl expose service argocd-server --type=NodePort --target-port=8080 --name=argocd-server-ext -n argocd
kubectl describe service argocd-server-ext -n argocd
```

Note: Find cluster ip
```sh
kubectl get nodes -o wide
```
Find NodePort
Find ClusterIp

https://<ClusterIp>:<NodePort>

Example:
https://192.168.64.3:30471

Get Password to connect to argoCD

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode ; echo


##### Check prometheus version

https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack

ArgoCd is up and running prepare argocd deploy kube-prometheus-stack

Open terminal repo

task-infra-spin % kubectl apply -f prometheus-application.yaml

Wait when all service is app in ArgoCD ui


#### Starting use prometheus-grafana ui on Local Machine


#### Get password for grafana dashboard

kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo