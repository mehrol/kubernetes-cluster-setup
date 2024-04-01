****************************************Run these command on Master and Worker node for setup kubernetes cluster**************************************

#This is to inform you that this document run only with debian or ubuntu machines where apt repositories are exists.


 
#Install Docker tool

sudo apt update -y
sudo apt install docker.io
sudo systemctl start docker


# disable swap
sudo swapoff -a

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

#Create directory in /etc/apt/keyrings
sudo mkdir /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet="1.29.0-*" kubectl="1.29.0-*" kubeadm="1.29.0-*"
sudo apt-get update -y
sudo apt-get install -y jq

sudo systemctl enable --now kubelet
sudo systemctl start kubelet



**************************************************Run these command only on Master node**************************************************************
#Run these command with local users.


sudo kubeadm config images pull

sudo kubeadm init

mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config


# Network Plugin = calico
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml

kubeadm token create --print-join-command



*************************************************Execute on ALL of your Worker Node's********************************************************************

#Run these command with root user or you have to setting up some configurations to run with local user too, but here with this we run with root only.

sudo kubeadm reset pre-flight checks

kubeadm join 192.168.1.*:6443 --token ********** #add your token here on worker node to join kubernetes cluster and all machine shows on master server.


************************************************RUN This command on Master node only*******************************************

#Run this command to check all your node connected with you master machine/join cluster.

kubectl get nodes






