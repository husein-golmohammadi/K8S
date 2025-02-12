# Installation
##First Configuration
Disable swapp
```bash
swapoff -a
sed -i '/swap/d' /etc/fstab
```
Set sysctl to enable networking
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```

##Install container 
```bash
sudo apt update -y
sudo apt install -y containerd
```
Configuring containerd to work with Kubernetes
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

##Install kubeadm, kubectl, kubelet
Add a Kubernetes repository
```bash
sudo apt install -y apt-transport-https ca-certificates curl
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Installing Kubernetes tools
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo systemctl enable kubelet
```

##Creating a Kubernetes cluster with kubeadm
on the master node
```bash
sudo kubeadm init --pod-network-cidr=10.154.0.0/16 --control-plane-endpoint=$(hostname -I | awk '{print $1}') --upload-certs
```
After finishing the command, it will say that you should type the following commands
‍‍‍‍‍```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##Add worker nodes to the cluster
For each worker node, execute the kubeadm join command that you received from the master
```bash
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

##Network Installation (CNI)
Install Calico
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

##Testing and checking nodes
kubectl get nodes
kubectl get pods -A
