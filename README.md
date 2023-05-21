# Kubernetes Cluster Setup
Foobar is a Python library for dealing with word pluralization.
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
Application Deploymenet using kubectl and Helm.both folder are available in Git repo.
Use the package manager [pip](https://pip.pypa.io/en/stable/) to install foobar.

```bash
pip install foobar
```

## Usage

```python
import foobar

# returns 'words'
foobar.pluralize('word')

# returns 'geese'
foobar.pluralize('goose')

# returns 'phenomenon'
foobar.singularize('phenomena')
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)