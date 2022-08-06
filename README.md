# Kubernetes-Installation-on-Ubuntu-20.04-LTS
This repo is for Installation of latest version of docker &amp; k8s cluster on Ubuntu 20.04 LTS

#### Please follow from step 1 to step 4 on every node, step 5 & step 6 only on master node and step 7 only on worker nodes.

### **Step 1: Disable Swap**

> Turn off swap
```
sudo swapoff -a
```

### **Step 2: Install Kubernetes Servers**

> Once the servers are ready, update them.
```
sudo apt-get update 
```
### **Step 3: Install kubelet, kubeadm and kubectl**

> After the servers are rebooted, add Kubernetes repository for Ubuntu 20.04 to all the servers.
```
sudo apt-get install -y apt-transport-https curl
```
> Add the GPG key for kubernetes

```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
> Add the kubernetes repository
```
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
> Update the repository
```
sudo apt update
```
> Install Kubernetes packages.
```
sudo apt-get install -y kubelet=1.24.0-00 kubeadm=1.24.0-00 kubectl=1.24.0-00 docker.io
```
> To hold the versions so that the versions will not get accidently upgraded.
```
sudo apt-mark hold kubelet kubeadm kubectl
```
> Confirm installation by checking the version of kubectl
```
kubectl version --client && kubeadm version
```
### **Step 4: Install Container runtime**

#### Installing Docker Run Time:
> Start & Enable Docker Packages
```
sudo systemctl start docker && sudo systemctl enable docker
```
# **Step 5 & Step 6 installation has to be done only on Master Node.**

### **Step 5: Initialize master node**

> Login to the server to be used as **Master Node** and Initialize the cluster by passing the cidr value and the value.

>>> **For Flannel Network**
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

>>> **For Calico Network**
```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
> To start using the cluster with current user.
```
mkdir -p $HOME/.kube
```
```
sudo cp -i /etc/kubernetes/admin.conf $home/.kube/config
```
```
sudo chown $(id -u):$(id -g) $home/.kube/config
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
