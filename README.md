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


asdf plugin-add k9s https://github.com/looztra/asdf-k9s
asdf install k9s latest
asdf global k9s latest

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.64.2
kubectl apply -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s-1.11.yaml"

# copy output for worker nodes
kubeadm token create --print-join-command

```

# bootstrap control tier

```
# Your Kubernetes control-plane has initialized successfully!

# To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=~/.kube/config

Then you can join any number of worker nodes by running the following on each as root:

sudo kubeadm join 192.168.64.2:6443 --token xauv91.bm08wbjs0oq2vpw7 \
        --discovery-token-ca-cert-hash sha256:80a18aad5ba414b917d66a7d3200db766ea76fe06dfc24f00d1520a49c664ad0

```


### join cluster from workers

```
# use this on worker nodes

sudo kubeadm join 192.168.64.2:6443 --token 7d3vz0.bn4j5y8x6exmycn8 --discovery-token-ca-cert-hash sha256:80a18aad5ba414b917d66a7d3200db766ea76fe06dfc24f00d1520a49c664ad0

```

### ctop

```
sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.7/ctop-0.7.7-linux-arm64 -O /usr/local/bin/ctop
sudo chmod +x /usr/local/bin/ctop
```


### kubectx tools

```
curl -s -L 'https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubectx_v0.9.5_linux_arm64.tar.gz' | tar xvz -C ~/.local/src
cp -a ~/.local/src/kubectx ~/.local/bin/
curl -s -L 'https://github.com/ahmetb/kubectx/releases/download/v0.9.5/kubens_v0.9.5_linux_arm64.tar.gz' | tar xvz -C ~/.local/src
cp -a ~/.local/src/kubens ~/.local/bin/

rm ~/.local/src/kubectx
rm ~/.local/src/kubens
```

### kubetail

```
asdf plugin-add kubetail https://github.com/janpieper/asdf-kubetail.git
asdf install kubetail latest
asdf global kubetail latest

```

### Weave scope

```
kubectl apply -f https://github.com/weaveworks/scope/releases/download/v1.13.2/k8s-scope.yaml
```


### helm

```
asdf plugin-add helm https://github.com/Antiarchitect/asdf-helm.git
asdf install helm latest
asdf global helm latest


helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
