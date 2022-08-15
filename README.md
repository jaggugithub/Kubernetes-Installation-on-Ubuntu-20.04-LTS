# Kubernetes-Installation-on-Ubuntu-20.04-LTS
This repo is for Installation of latest version of docker &amp; k8s cluster on Ubuntu 20.04 LTS

#### Please follow from step 1 to step 4 on every node, step 5 & step 6 only on master node and step 7 on only worker nodes.

### **Step 1: Install Kubernetes Servers**

> Once the servers are ready, update them.
```
sudo apt update
```
> After updating,upgrade the servers and reboot the system
```
sudo apt -y upgrade && sudo systemctl reboot
```
### **Step 2: Install kubelet, kubeadm and kubectl**

> After the servers are rebooted, add Kubernetes repository for Ubuntu 20.04 to all the servers.
```
sudo apt update
```
```
sudo apt -y install curl apt-transport-https
```
> Add the GPG key for kubernetes

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
> Add the kubernetes repository
```
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
> Update the repository
```
sudo apt update
```
> Install Kubernetes packages.
```
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
```
> To hold the versions so that the versions will not get accidently upgraded.
```
sudo apt-mark hold kubelet kubeadm kubectl
```
> Confirm installation by checking the version of kubectl
```
kubectl version --client && kubeadm version
```
### **Step 3: Disable Swap**

> Turn off swap
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
```
sudo swapoff -a
```
### **Enable kernel modules and configure sysctl**

> Enable kernel modules and configure sysctl
```
sudo modprobe overlay
```
```
sudo modprobe br_netfilter
```
> Enable the iptables bridge(Set a value in the sysctl file , to allow proper network settings for Kubernetes on all the servers)
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```
> Reload sysctl
```
sudo sysctl -p
```
### **Step 4: Install Container runtime**

#### Installing Docker Run Time:

```
sudo apt update
```
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
> Add the GPG key for Docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –
```
> Add docker Repo
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
```
sudo apt update
```
> Install Docker Packages
```
sudo apt install -y docker-ce
```
> To Hold The Version Of Docker
```
sudo apt-mark hold docker-ce
```
> Start and enable Docker Services
```
sudo systemctl restart docker
```
sudo systemctl enable docker
```
# **Step 5 & Step 6 installation has to be done only on Master Node.**

### **Step 5: Initialize master node**

> Login to the server to be used as **Master Node** and make sure that the br_netfilter module is loaded
```
lsmod | grep br_netfilter
```
> Enable kubelet service
```
sudo systemctl enable kubelet
```
####Initialize the cluster by passing the cidr value.
> For Flannel Network
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
> For calico Network
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
> To start using the cluster with current user.
```
mkdir -p $HOME/.kube
```
```
sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config
```
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
To check the Check cluster status:
```
kubectl cluster-info
```
### **Step 6: Install network plugin on Master**

In this guide we’ll use Flannel.

> To Set the flannel networking
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
> To Set the Calico networking
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### **Step 7: Add worker nodes**

> Joining the node to the cluster:
```
#Please do not copy this command, this is just an example
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```
## TIP
> If the joining code is lost, it can be retrieved using below command
```
kubeadm token create --print-join-command
```

##### Now please login to the master node and check whether the worker nodes has joined the cluster or not.

> Using below command
```
kubectl get nodes
```
##### Similarly follow the same steps for creating the other worker node or for remaining nodes as well.
