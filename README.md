# Kubernetes with minikube

## Preparation

```bash
brew install minikube kubectl helm
```

## Minikube configuration

```bash
minikube config set driver hyperkit
minikube config set cpus 4
minikube config set memory 10240
minikube config set kubernetes-version v1.22.0

minikube start
```

## JupyterHub deployment

```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update

kubectl create ns jupyterhub

helm upgrade --cleanup-on-fail \
  --install jupyterhub jupyterhub/jupyterhub \
  --namespace jupyterhub \
  --create-namespace \
  --version=1.1.2 \
  --values config.yaml
```

By default helm chart expose with a loadbalancer service to use it do :

```bash
minikube tunnel
```