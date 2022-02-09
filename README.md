# Kubernetes-Installation-on-Ubuntu-20.04-LTS
This repo is for Installation of latest version of docker &amp; k8s cluster on Ubuntu 20.04 LTS

Step 1: Install Kubernetes Servers

sudo apt update

sudo apt -y upgrade && sudo systemctl reboot

Step 2: Install kubelet, kubeadm and kubectl

sudo apt update

sudo apt -y install curl apt-transport-https

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add –

echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update

sudo apt -y install vim git curl wget kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

kubectl version --client && kubeadm version

Step 3: Disable Swap

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

sudo swapoff -a

Step 4: Enable kernel modules and configure sysctl

sudo modprobe overlay

sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl –system

Step 4: Install Container runtime
Installing Docker Run Time:

sudo apt update

sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install -y containerd.io docker-ce docker-ce-cli

sudo mkdir -p /etc/systemd/system/docker.service.d

sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker

sudo systemctl enable docker

Note: From Step 5 to Step 6 the installation has to be   done only on Master Node.

Step 5: Initialize master node

# Login to the server to be used as master and make sure that the br_netfilter module is loaded:

lsmod | grep br_netfilter

# Enable kubelet service

sudo systemctl enable kubelet

# We now want to initialize the machine that will run the control plane components which includes etcd (the cluster database) and the API Server.

sudo kubeadm config images pull

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Configure kubectl using the below commands:

mkdir -p $HOME/.kube

sudo cp -f /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

#To check the Check cluster status:

kubectl cluster-info

Step 6: Install network plugin on Master

In this guide we’ll use Flannel.

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Step 7: Add worker nodes

sudo kubeadm join 10.154.0.15:6443 --token 2kep4q.m49l6pywehje5ml7 --discovery-token-ca-cert-hash sha256:a90d5936618b4c695faa2284505fee3a9aa914e55002738f0e81682e7c1d204b

Now please login to the master node and check whether this worker node has joined the cluster or not.

Similarly follow the same steps for creating the other worker node or for remaining nodes as well.







