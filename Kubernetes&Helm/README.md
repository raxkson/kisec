# Install Kubernetes & Helm by raxkson

## Kubernetes
### Serveral config & Install Docker for kubernetes
swap off for virtual environment
```bash
sudo swapoff -a
sudo echo 0 > /proc/sys/vm/swappiness
sudo sed -e '/swap/ s/^#*/#/' -i /etc/fstab
```

packages for apt allow HTTPS
```bash
sudo apt-get update && sudo apt-get install -y \
apt-transport-https ca-certificates curl software-properties-common gnupg2
```

Add Docker public GPG key
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -
```

Add Docker apt repo
```bash
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

If Docker exists, erase it for collusion
```bash
sudo apt-get remove docker docker-engine docker.io
```

Install Docker CE
```bash
sudo apt-get update && sudo apt-get install -y \
  containerd.io \
  docker-ce \
  docker-ce-cli
```

Create /etc/docker
```bash
sudo mkdir /etc/docker
```

Config Docker Daemon
```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

Create /etc/systemd/system/docker.service.d
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Restart Docker
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```

Test Docker with hello-world
```bash
sudo docker run hello-world
```


### Install Kubernetes
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

Prevent auto update
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Start Kubernetes and config Fannel
```bash
internal_ip=`(ifconfig ens33 | fgrep -i inet | awk '{print $2}' | head -n 1)`
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=$internal_ip
```

Use Kubernetes with user
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install Fannel for Kubernetes (only for master node)
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

## Helm (for master node)
### Install Helm
Get helm scripts in repo & install
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
sudo chmod 700 get_helm.sh
./get_helm.sh
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

### We have to create another vm and install kubernetes for worker node
Create join command for worker node and enter command in woker node
```bash
kubeadm token create --print-join-command
```
