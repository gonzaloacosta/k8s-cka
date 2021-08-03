# Install Kubernetes Cluster

## Install kubernetes Cluster

[Here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Execute in all nodes master and workers

Check bridge

```
lsmod | grep br_netfilter
sudo modprobe br_netfilter
```

Letting iptables see bridged traffic

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

Install kubeadm and kubelet

```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Install Docker Runtime

[Here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

```
# (Install Docker CE)
## Set up the repository:
### Install packages to allow apt to use a repository over HTTPS
sudo apt-get update && sudo apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common gnupg2

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key --keyring /etc/apt/trusted.gpg.d/docker.gpg add -

# Add the Docker apt repository:
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# Install Docker CE
sudo apt-get update && sudo apt-get install -y \
  containerd.io=1.2.13-2 \
  docker-ce=5:19.03.11~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.11~3-0~ubuntu-$(lsb_release -cs)

# Set up the Docker daemon
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

# Create /etc/systemd/system/docker.service.d
sudo mkdir -p /etc/systemd/system/docker.service.d

# Restart Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

sudo systemctl enable docker
```

## Install Cluster with Kubeadm

[Here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

In master nodes
```
kubeadm init
```

```
$ kubeadm token create --print-join-command
kubeadm join 172.17.0.43:6443 --token czim10.id17eg6f7tegd7fm     --discovery-token-ca-cert-hash sha256:8dc0f6a7e2c1a9ad216ad2bfef2459d668f488909aa6a827822a723aba7e20dc
```

In worker node01

```
kubeadm join 172.17.0.43:6443 --token czim10.id17eg6f7tegd7fm     --discovery-token-ca-cert-hash sha256:8dc0f6a7e2c1a9ad216ad2bfef2459d668f488909aa6a827822a723aba7e20dc
```

## Install CNI

[Here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#stacked-control-plane-and-etcd-nodes)

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

## Waiting to ready nodes

```
watch kubectl get nodes
```



