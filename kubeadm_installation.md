## Setup a Kubernetes Cluster on AWS EC2 Instance with Ubuntu using kubeadm

We will use the "kubeadm" tool to set up the cluster. Kubeadm is a tool built to provide "kubeadm init" and "kubeadm join" for creating Kubernetes clusters. 

STEPS:

1. Spin up two nodes, one will be control, one will be worker
- t2.medium
- 20g ram 
2. Connect to the nodes using Putty or mobiXterm
3. Once connected , perform the following:

## Pre-requisites
3 Ubuntu 18.04 Servers with minimum 2 GBs RAM and 2 CPUs.
A system user with "sudo" access on each server. 
## What we will do
Setup a Kubernetes Cluster with kubeadm
Setup a Kubernetes Cluster with kubeadm

Use the following commands to set a hostname on each server. 
After executing the following commands on each server, re-login to the servers so that the servers will get a new Hostname. 

$ sudo hostnamectl set-hostname "master"
$ sudo hostnamectl set-hostname "node1"
$ sudo hostnamectl set-hostname "node2"

## Get the Docker gpg key (Execute the following command on All the Nodes):

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

## Add the Docker repository(Execute the following command on All the Nodes):

sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
  stable"
  
 ## Get the Kubernetes gpg key(Execute the following command on All the Nodes):
 
 curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
 
 ## Add the Kubernetes repository(Execute the following command on All the Nodes):
 
 cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

## Update your packages(Execute the following command on All the Nodes): 

sudo apt-get update

## Install Docker, kubelet, kubeadm, and kubectl(Execute the following command on All the Nodes):

sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00

## Hold them at the current version(Execute the following command on All the Nodes):

sudo apt-mark hold docker-ce kubelet kubeadm kubectl

## Add the iptables rule to sysctl.conf (Execute the following command on All the Nodes):

echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

## Enable iptables immediately(Execute the following command on All the Nodes:

sudo sysctl -p

## On Master: 

## Initialize the cluster (Execute the following command only on the Master node):

sudo kubeadm init --pod-network-cidr=10.244.0.0/16

## Set up local kubeconfig(Execute the following command only on the Master node):

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## Apply Flannel CNI network overlay(Execute the following command only on the Master node):
kubectl -n kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

## On Node1 and Node 2: 

## Join the worker nodes to the cluster (Execute the following command only on Node1 and Node2):

## And don't forget to do the following command to enable scheduling on the created cluster: ( may not be necessary)

kubectl taint nodes --all node-role.kubernetes.io/master-


## READ 
https://kubernetes.io/docs/tasks/run-application/configure-pdb/
===================================================================================================================================
## Known errors

Container runtime network not ready: cni config uninitialized

Make sure to apply a network flannel to the nodes

https://stackoverflow.com/questions/49112336/container-runtime-network-not-ready-cni-config-uninitialized

##  Always remember to Add pod network add-on

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
If flannel doesn't work, then try calico -

kubectl apply -f https://docs.projectcalico.org/manifests/calico-typha.yaml


