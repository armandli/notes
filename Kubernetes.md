### Kubernetes

minikube is a kubernetes experimentation tool
kubernetes has a dashboard available to see the status

commands
```
minikube version #obtain kubernetes version
minikube start   #start kubernetes cluster

# creating a deployment
kubectl run <name> --image=<image-path> --port=<no> --replicas=<number copy> --hostport=<internal_port_no> # start off a pod with name <name> using image <image-path> on port <no>, exposing internal port no <internal_port_no>
# using --hostport exposes the pod via Docker Port Mapping, as a result kubectl get svc will not see it

# making service available externally
kubectl expose deployment <name> --port=<port_no> --type=NodePort #container can be exposed by different means, one way is a NodePort, otherwise a container is not exposed externally
kubectl expose deployment <name> --external-ip="<ip>" --port=<internal_port_no> --target-port=<external_port_no> # expose the container port <external_port_no> on the host <internal_port_no> binding to the <ip> of the host

# scale containers
kubectl scale --replicas=<new replica number> deployment <pod_name> #increase/decrease the number of replicas of a container

# using yaml configurations for deployment
kubectl create -f <deployment yaml file> #can be a deployment or service
kubectl apply -f <deployment yaml file> #updates deployment based on changes in the yaml file


# obtaining kubernetes information
kubectl cluster-info #general info regarding kubernetes cluster
kubectl get nodes    #show all nodes
kubectl get deployments # view status of all deployments
kubectl get pods     #show all pods and pod status
kubectl get svc <podname> -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{end}}{{end}}' #obtain the NodePort of a pod with name podname
kubectl describe deployment <name> # description include how many replicas, labels specified, events associated with, including errors
kubectl describe svc <pod name> #view the endpoints and the associated pods (ip:port combinations of all replicas)

# obtain docker information
docker ps # show exposed ports through docker, this is important when kubernetes uses docker to do port mapping
```

first step in creating a cluster is launching the master node, who is running the control plane components, etcd, API server.

```
kubeadm init --token=<tokenid> --kubernetes-version $(kubeadm version -o short) #start kubernetes master
```

it is recommended to skip the token to let kubeadm to generate one automatically

additional node can join the cluster as long as they have the correct token. to see all tokens:

```
kubeadm token list
```

to join a kube cluster:

```
kubeadm join --discovery-token-unsafe-skip-ca-certification --token=<token> <master-ip>:6443
```

kubeadm creates management configurations for user when cluster is created, and save them into `/etc/kubernetes/admin.conf`. These configurations need to be copied into user space before able to use commands such as `kubectl`

```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

container network interface need to be established before nodes in a cluster become ready. there are many network system we can use, one is WeaveWorks.

```
kubectl apply -f /opt/weave-kube
```

WeaveWorks will deploy series of pods on the cluster, the status can be viewed using command `kubectl get pod -n kube-system`

pods contains docker images running a certain application.

to deploy the dashboard yaml

```
kubectl apply -f dashboard.yaml
```

#### deployment.yaml
the deployment object defines the container spec required, along with the name and labels used by other parts of kubernetes to discover and connect to the application. e.g. deployment yaml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

#### service.yaml
controls the networking configuration. the service selects all applications with the label <name>. As multiple replicas or instances are deployed. it automatically load balance based on the common label. the service makes the application available via NodePort. e.g.
  
```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1
```

#### controller.yaml
configuration for controlling the replication of a server. e.g.

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: redis:3.0.7-alpine
        ports:
        - containerPort: 6379
```

#### Networking
Cluster IP is the default approach when creating Kubernetes Service. The service allocate an internal IP that other components can use to access the pods. Having a single IP address allows the service to be load balanced across multiple Pods. e.g.

```
apiVersion: v1
kind: Service
metadata:
  name: webapp1-clusterip-svc
  labels:
    app: webapp1-clusterip
spec:
  ports:
  - port: 80
  selector:
    app: webapp1-clusterip
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webapp1-clusterip-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: webapp1-clusterip
    spec:
      containers:
      - name: webapp1-clusterip-pod
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
---
```

the cluster IP can be found using `kubectl get services/<ClusterIPService> -o go-template='{{(index .spec.clusterIP)}}'`

Target ports allow the separation between the port the service is available on from the port the application is listening on. Target Port is the port the application is listening on. It is the port to be accessed outside.

NodePort exposes the service on each node's IP via the defined static port. No matter which node within the cluster is accessed, the service will be reachable based on the port number defined.

Service can also use External IP Address to make it available to the outside.

When running in the cloud, it's possible to configure and assign a public IP address issued via cloud provider. This will be issued via a Load Balancer such as ELB. This allows additional public IP addressed to be allocated to a Kubernetes cluster without interacting directly with the cloud provider.

