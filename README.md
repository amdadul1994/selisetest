# Kubernetes Cluster Setup
## VM2 -kubernetes_Master
## VM3-kubernetes_Client
## VM1-NFS Server
## Below insatllation process both Master and Client NOode
### Installation
Disable SWAP
```bash
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```
### Update your system
```bash
sudo apt update
```
### Insatll Docker
```bash
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```
### Add kubernetes package repo and install kubeadm,kubelet & kubectl
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### Only in Master Node:
Api integration and cluster network setup.edit flannel ip according to your pod network
```bash
sudo kubeadm init --apiserver-advertise-address=10.128.0.2 --pod-network-cidr=10.0.0.0/16 

 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```
## Install NFS Server
Install the NFS kernel Server package
```bash
sudo apt update
sudo apt install nfs-kernel-server
```
The next step will be to create an NFS directory share.
```bash
sudo mkdir /nfsshare
sudo chown nobody:nogroup /nfsshare
sudo chmod -R 777 /nfsshare
```
So, open the /etc/exports file
```bash
/nfsshare 10.128.0.3(rw,sync,no_subtree_check) 10.128.0.4(rw,sync,no_subtree_check)
```
Export the shared directory.To export the directory and make it available, invoke the command:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-server
```

Now log in to the client system and update the package index as shown.
```bash
sudo apt update
sudo apt install nfs-common
```
Application Deploymenet using kubectl and Helm.both folder are available in Git repo.
