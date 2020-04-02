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
