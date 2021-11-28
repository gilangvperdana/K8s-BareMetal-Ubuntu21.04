# SSL Local Ingress Service

```
Environment :
    1. Helm
```

# Install Helm :
```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

# Install Cert Manager :
```
# If your cluster version is Kubernetes >= 1.16
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager.crds.yaml

# If your cluster version is Kubernetes <= 1.15
$ kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager-legacy.crds.yaml

OR YOU CAN INSTALL WITH HELM :

# If you haven't install jetstack helm chart repo yet
$ helm repo add jetstack https://charts.jetstack.io

# Create namespace for cert-manager
$ kubectl create namespace cert-manager

# If you use Helm v3+
$ helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.4

# If you use Helm v2
$ helm install --name cert-manager --namespace cert-manager --version v1.0.4 jetstack/cert-manager

```

# Create Self-Signed Cert Issuer
```
$ kubectl apply -f https://gist.githubusercontent.com/t83714/51440e2ed212991655959f45d8d037cc/raw/7b16949f95e2dd61e522e247749d77bc697fd63c/selfsigned-issuer.yaml
```

# Access :
```
Deploy :
$ kubectl create -f 1.\ IngressSSL_Clock.yaml,2.\ IngresSSL_Bubble.yaml,3.\ IngressSSL_Calculator.yaml

Access on https://clocks.adaptive.local \\ https://bubbles.adaptive.local \\ https://calculators.adaptive.local
```