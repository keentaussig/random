## Dive into K8s 

### K8s
* **What is Kubernetes?** system for running many different containers over multiple different machines;
* **Why?** When needed to run multiple different containers with different images;
* **Cluster** assembly of a master and one or more nodes, which can be used to run different sets of containers;
* **Install kubectl** on linux: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux
* **Install minikube** https://kubernetes.io/docs/tasks/tools/install-minikube/
* **Install homebrew** on linux https://docs.brew.sh/Homebrew-on-Linux
* <optional> `brew install docker-machine-driver-vmware`
* start minikube and check the status
```
minikube start --alsologtostderr -v=7
minikube status
kubectl cluster-info
```
* if error `VT-x is disabled in the BIOS for both all CPU modes (VERR_VMX_MSR_ALL_VMX_DISABLED)` enable CPU virtualization advanced setting in BIOS 

### Docker Compose -> Kubernetes
* [DC] Each entry can optionally get docker-compose to build an image -> [K] Kubernetes expects all images to already be built
* [DC] Each entry represetns a container we want to create -> [K] One config file per *object* we want to create
* [DC] Each entry defines the networking requirements (ports) -> [K] We have to manually set up all networking

### K8s configurations
* Config file -> example of oject types: Pod, Service, ReplicaController, StatefulSet. Pods are used to run a container
* apiVersion: v1
  * opens up access to a pre-defined set of object types 
  * componentSTatus, configMap, Pod
* apiVersion: apps/v1
  * ControllerRevision, StatefulSet 
* minikube start -> 
    created VM (Node) -> 
      Node will be used by K8s to run some numbr of objects -> 
      some of them are referred as Pod -> 
      running kubectl we will create a Pod ->
      Pod is a groupping of containers
      
* in K8s world there is no such thing to just run a single container by itself, the smallest thing we can deploy is a Pod to run a single container
* we must to run 1 or more containers inside a Pod
* The purpose of a Pod is to allow groupping containers with a very similar purpose, or very tight integration with each other
* Service (object type) used for setting up networking, subtypes:
  * ClusterIP
  * NodePort (purpose: expose a container to the outside world, good only for dev purposes)
  * LoadBalancer
  * Ingress
* Label selector system
  * metadata->labels->component->web
  * I.e. selector inside the service configuration selects component: web, which is defined in the client Pod configuration
* Ports section defined in the service node config
  * 3 different ports
    * port that another container could access for the client pod (in `clinet-node-port.yaml ports->port)
    * targetPort = is a port inside the cloint pod which is mapped to node container
    * nodePort is the port that users will use in browser to access a multi-container application, this is what gets exposed to the outside port, it is always a numbr between (30000-32767), if it's not specified, it will be assigned randomly
* apply config files:
```
kubectl apply -f client-node-port.yaml 
kubectl apply -f client-pod.yaml
kubectl get pods
```
Output:
```
NAME         READY   STATUS    RESTARTS   AGE
client-pod   1/1     Running   0          77s
```
`1/1` means that one copy of one desired copy is running.
Get status of services:
```
mustard@blubunny:~/ws/simplek8s$ kubectl get services
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
client-node-port   NodePort    10.100.129.107   <none>        3050:31515/TCP   4m32s
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP          62m
```
3050:31515/TCP, 3050 port that other services will use for the service, and 31515/TCP used in browser.
* Get minikube IP
```
minikube ip
```
* Deployment - Object Type which maintains a set of pods. With the pod we are running a single set of containers;
  * Runs a set of identical pods;
  * Monitor the state of each pod, updating as necessary (it will either update existing pod, or recreate a new one)
  * Good for dev
  * Good for prod
* remove pod 
```
kubectl delete -f <config file>
```
* `kubectl get deployments` <- get current deployments
* update docker image
```
sudo docker build -t suspicioushaibt/fib-client .
sudo docker push suspicioushaibt/fib-client
sudo docker login
sudo docker push suspicioushaibt/fib-client
```
* Reconfigure docker cli to look inside the minikube VM `eval $(minikube docker-env)`
* Examine logs inside a container `docker exec -it 208b7f9762c4 sh`
* Clean unused container from minikube `docker system prune -a`
### Path to Prod
* Create config files for ClusterIPs and all Deployments
* Test locally on minikube
* Create a github/travis deployment
### PVC
* PVC - Persistent Volume Claim
* Postgres PVC
* Postgres Deployment -> Postgres Container -> Postgres is a DB, takes in some amount of data and writes it to file system
* If the Pods that wraps it, we will lose all data inside it
* Volume, generally, is a mechanism that allows a container to access a filesystem outside itself 
* Volume, **in the world of Kubernetes,** is an object that allows a container to store data at the pod level
  * Kubernetes Volume is a data storage pocket which is tied to a particular Pod
  * Pros: if a container dies then the new container will still access to that same volume that the previos container had access too* 
  * Cons: if the Pod itself ever fails, gets recreated, etc. then the data is gone
  * Thus, k8s volumes are not appropriate for persistent data storage
* Persistent Volume
 * If Pod crashes, new Pod is created and works with the persisten Volume associated with the crashed instance
* Perisistent Volume Claim (not the same as Volume in k8s)
  * PVC is not an actual storage, it's a declaration of what should be availble for Pods. Something that one can get when Pod is created. 
  * K8s checks through instances of storage options, that have need created ahead of time (to be used right away for storage - statically provisioned), and there is also availble dynamically provisioned option
  * PVC is a declaration of options amoung statically or dynamically available storage options 
  * `kubectl describe storageclass`
  * In cloud: Azure File, Azure Disk, AWS Block Store, Google Cloud Persistent Disk
  * `kubectl get pv`

### Secrets
* `kubectl create secret generic/tls/<OTHER> <SECRET_NAME> --from-literal key=value`
* `kubectl get secrets`

### Load Balancer Service
* Provides network to reach a set of Pods
* Load Balancer Service - reaches out cloud provider and uses AWS or Azure load balancers configuration
* It is going to automatically configure that lb to govern access to a specific set of Pods
* It sounds like it is deprecated, use Ingress Service instead

### Ingress
* Nginx Ingress **ingress-nginx** (community lead project by kubernetes community). GitHub: kubernetes/ingress-nginx
* There is also **kubernetes-ingress** - this is also an nginx ingress, but it's a 100% separate project lead by nginx company
* Blog about Ingress https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html
* Ingress-nginx guide https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command
* To check if the ingress controller pods have started: `kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch`

### Dashboard
```
minikube dashboard
```

### Cert Manager Installation
https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#steps

```
#1. Apply the yaml config file
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/deploy/manifests/00-crds.yaml

# 2. Create the namespace for cert-manager
kubectl create namespace cert-manager

# 3. Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# 4. Update your local Helm chart repository cache
helm repo update

# 5.  Install the cert-manager Helm chart (Helm 3):
helm install cert-manager --namespace cert-manager --version v0.11.0 jetstack/cert-manager
```

### issuer.yaml
https://docs.cert-manager.io/en/latest/tasks/issuers/setup-acme/index.html#creating-a-basic-acme-issuer

1. `apiVersion: cert-manager.io/v1alpha2`
2. Add the solvers property
```
    apiVersion: cert-manager.io/v1alpha2
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        email: "youremail@email.com"
        privateKeySecretRef:
          name: letsencrypt-prod
        solvers:
          - http01:
              ingress:
                class: nginx
```

### certificate.yaml file

1. Update the apiVersion with: `apiVersion: cert-manager.io/v1alpha2`

### ingress-service.yaml file
```
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```
