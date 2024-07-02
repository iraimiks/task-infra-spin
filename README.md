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

#### Install microk8s (Task 1 Prepare a Kubernetes Cluster)

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
```

#### Install argoCD (Task 2 Deploy Automation Stack ArgoCD)
(https://argo-cd.readthedocs.io/en/stable/getting_started/)

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

https://ClusterIp:NodePort

Example:
https://192.168.64.3:30471

Get Password to connect to argoCD

kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode ; echo


##### Install Prometheus, Grafana (Task 3 Deploy Monitoring System)

https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack

ArgoCd is up and running prepare argocd deploy kube-prometheus-stack

Open terminal in project repo task-infra-spin

```sh
kubectl apply -f prometheus-application.yaml
```

Wait when all service is app in ArgoCD ui

Use portforwarding for service/prometheus-grafana
Open grafana dashboards

Get password

```sh
kubectl get secret --namespace monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
##### Prepare dashboards

cd monitoring/dashboards

```sh
kubectl apply -f simple_cpu_load_dash.yaml
```

##### Prepare Alerts

https://grafana.com/blog/2023/12/28/how-to-integrate-grafana-alerting-and-telegram/

Create new contact point (telegram-send)


Go to Grafana -> Alerting -> Alertin rules -> + new alert rule

- Enter alert rule name: 
  CPU load 90%
- Define query and alert condition -> code :
Metric browser: 
```sh
(sum(rate(container_cpu_usage_seconds_total{pod="ubuntu"}[5m])) by (pod)) * 100 > 90
```

Explain: {pod="ubuntu"} – This specifies the exact name of the container pod to which the alert is assigned.

- Set evaluation behavior: 
    If not exits create Folder: custom_alert_cpu_rules
    If not exits create Groupe: custom_alert

- Contact point 
    Use precreate contact point: telegram-send





#### Prepare container for load cpu load testing (Task 4 Stress Test)

In repor task-infra-spin/app (Using simple ubuntu container which running)
Manual prepare stress test load for cpu using package stress-ng

Connect to ubuntu pod 

```sh
apt update && apt install stress-ng
```

Run command to load cpu to specific load as 90%

```sh
stress-ng -c 0 -l 90
```

#### Prepare alerts 
