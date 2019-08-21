# Kubernetes

## Components
The primary parts of a kubernetes cluster is broken down into a master and multi slaves. The master comes with the following components:
- Cluster Store (etcd)
- Scheduler
- Controller Manager
- API Server (Central point for communication with the master)

The slaves come with the following components
- Kubelet
- Kube-proxy
- Container Runtime (docker)

## System Requirements
- Linux (Ubuntu/CentOS)
- 2 CPUs
- 2GB RAM
- Disable Swap Space
- Container Runtime Interface (Docker)
- Network connectivity between all nodes and master

## Cluster Network Ports

### API Server
This component runs on port 6443. It is used by anything.

### etcd
This runs on 2379 and 2380. This is used by the API Server and etcd itsel (for redundancy).

### scheduler
It listens on 10251 and it is used by itself.

### Controller Manager
Similar to the scheduler, it runs on 10252 and it is used by itself.

### Kubelet
This runs on 10250. It is used by all the control plane components. This is similar for both the server and the nodes as well.

### NodePort Service
This is a type of kubernetes service that exposes our services on ports on the individual nodes in our cluster. This is allocated between 30000-32767. This is used by anything that needs it.

## Building Cluster
- Install Kubernetes
- Create cluster (This concludes bootstrapping the various components)
- Configure the pod networking
- Join additional nodes to cluster

## Required Packages
The following needs to be installed on all the nodes (and master as well):
- kubelet
- kubeadm (tool for bootstrapping and joining additional nodes)
- kubectl (CLI tool to execute commands)
- docker (container runtime)

### Install Kubernetes

The first step is to add the gpg for the apt repository where kubernetes lives. 
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt-get update
```
The next step is to install the required packages.
```
apt-get install -y kubelet kubeadm kubectl docker.io
```
A crucial next step (which can be avoided per choice) is that we no longer want apt to maintain the upgrading of the various required packages. We can achieve this using:
```
apt-mark hold kubelet kubeadm kubectl docker.io
```
If you are curious to find out what packages you currently are running, the following command could be useful:
```
apt-cache policy kubelet
# apt-cache policy kubelet | head -n 20
```
This can also be useful in determining a good candidate to install and how that compares to the current installed package.

At this point, it is very useful to understand the status of the `kubelet` since no work has been allocated to it.
```
systemctl status kubelet.service
```
It is clear from the logs that the `The Kubernetes Node Agent` exits with a `255` error code (a bad one) even though it has been enabled. This is because it found no job(s) to run. This also be checked with docker service as well. 
```
systemctl status docker.service
```
Happily, the docker service is running. If by any chance, we want to manually start these services, we run:
```
systemctl enable <service_name>
```

### Disable Swap
To disable the swap space on all nodes (including master), use the following:
```
swapoff -a
```
The `-a` option disables all swaps from `/proc/swaps` which is what we need on all nodes. You could confirm by peeping into the `/etc/fstab` file.

## Boostrapping the cluster
Since kubernetes 1.13, `kubeadm` became the preferred tool for bootstrapping the kubernetes cluster.

The first step is to download some yaml manifests that describe some pod network that we want to deploy. The first is `rbac-kdd`;
```
wget https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
```
The second file is the actual deployment file for our pod networking:
```
wget https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```
The next step is to initialize `kubeadm` and define a cidr network range
```
kubeadm init --pod-network-cidr=192.168.0.0/16
```
If there is a need to edit the network range, we could just edit the calico yml file

The first phase is the preflight checks. It would pull the control plane images and perform all the necessary checks. It also creates a CA for both authentication and encryption. It also creates required kubeconfig files for authenticating the various components agains the API Server. The next is that it generates the static pod manifests. After that, it starts the control plane, taints the master (when pods are scheduled, only system pods will execute on the master nodes) and then starts add-on pods (dns and kube-proxy). This process is very customizable though what has been described is the defaults that come with `kubeadm`.

Our account on the master has to be configured to have admin access to the API Server from a non-privileged account.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

We then need to deploy the rbac file and calico yml file so that we could add a new nodes.

```
kubectl apply -f rbac-kdd.yaml
kubectl apply -f calico.yaml
```

Now we are good to go
```
# Get all kube-system pods
kubectl get pods --all-namepaces
# Get all nodes
kubectl get nodes
```

It can also be confirmed that `systemctl status kubelet.service` no more throws an error.
The important configurations can all be found at `/etc/kubernetes`.
