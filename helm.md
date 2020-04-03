https://kubernetes.github.io/ingress-nginx/deploy/#using-helm

### What is Helm?
Used to administer software inside kubernetes cluster.

1. Install Helm v3:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```
3. Install Ingress-Nginx:
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm install my-nginx stable/nginx-ingress --set rbac.create=true
```
