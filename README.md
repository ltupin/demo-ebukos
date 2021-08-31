# Kubernetes with minikube

## Preparation

```bash
brew install minikube kubectl helm faas-cli
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

Config file location: 

```bash
/usr/local/etc/jupyterhub/secret/values.yaml
```

## Openfaas deployment

```bash
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm repo update

kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml

export PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)

kubectl -n openfaas create secret generic basic-auth \
    --from-literal=basic-auth-user=admin \
    --from-literal=basic-auth-password="${PASSWORD}"

helm upgrade openfaas \
    --install openfaas/openfaas \
    --namespace openfaas \
    --set functionNamespace=openfaas-fn \
    --set basic_auth=true

export OPENFAAS_URL=$(minikube ip):31112

# DOnt forget to && or to open another shell
kubectl port-forward svc/gateway -n openfaas 31112:8080 &&

echo -n ${PASSWORD} | faas-cli login -g http://$OPENFAAS_URL -u admin — password-stdin
```

```bash
faas-cli new --lang python3 hello
faas-cli build -f hello.yml
faas-cli push -f hello.yml

export gw=${OPENFAAS_URL}
faas-cli deploy -f hello.yml --gateway ${gw}
```
