# cka-prep
Been using k8s for years but still haven't got my CKA yet. It's about time, and this repo will be where I post all of my work while learning


## Quickstart

```
mkdir ~/kodekloud
cd ~/kodekloud
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/apple-silicon

./deploy-virtual-machines.sh
```


### expand disks
```
multipass stop kubemaster
multipass stop kubeworker01
multipass stop kubeworker02
multipass set local.kubemaster.disk=30G
multipass set local.kubeworker01.disk=30G
multipass set local.kubeworker02.disk=30G

multipass start kubemaster
multipass start kubeworker01
multipass start kubeworker02
```

### dotfiles

```
export ZSH_DOTFILES_PREP_GITHUB_USER=bossjones

curl -fsSL https://raw.githubusercontent.com/bossjones/zsh-dotfiles-prep/main/bin/zsh-dotfiles-prereq-installer | bash -s -- --debug

chezmoi init -R --debug -v --apply https://github.com/bossjones/zsh-dotfiles.git

sudo chsh -s $(which zsh) ubuntu
```


### k8s install on each node

```
# ssh to each node and run this


mkdir -p ~/.local/src || true
cd ~/.local/src || true
# Load required kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Persist modules between restarts
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# Set required networking parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

curl -LO https://github.com/containerd/containerd/releases/download/v1.7.0/containerd-1.7.0-linux-arm64.tar.gz
sudo tar Czxvf /usr/local containerd-1.7.0-linux-arm64.tar.gz

curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/lib/systemd/system
sudo mv containerd.service /usr/lib/systemd/system/

sudo mkdir -p /etc/containerd/
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl daemon-reload
sudo systemctl enable --now containerd

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt-get install net-tools -y


curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

KUBE_VERSION=1.26.4
sudo apt-get update
sudo apt-get install -y kubelet=${KUBE_VERSION}-00 jq kubectl=${KUBE_VERSION}-00 kubeadm=${KUBE_VERSION}-00 runc kubernetes-cni=1.2.0-00
sudo apt-mark hold kubelet kubeadm kubectl

sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```
