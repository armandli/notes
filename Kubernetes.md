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
