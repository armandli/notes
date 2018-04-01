### Kubernetes

minikube is a kubernetes experimentation tool
kubernetes has a dashboard available to see the status

commands
```
minikube version #obtain kubernetes version
minikube start   #start kubernetes cluster
kubectl cluster-info #general info regarding kubernetes cluster
kubectl get nodes    #show all nodes
kubectl run <name> --image=<image-path> --port=<no> --replicas=<number copy># start off a pod with name <name> using image <image-path> on port <no>
kubectl get pods     #show all pods and pod status
kubectl expose deployment first-deployment --port=80 --type=NodePort #container can be exposed by different means, one way is a NodePort, otherwise a container is not exposed externally
kubectl get svc <podname> -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{end}}{{end}}' #obtain the NodePort of a pod with name podname
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
