# Common installation steps

Those steps are common for master and worker nodes.

## disable SWAP

```bash
    sudo systemctl status *swap

    ### note the swap file name (ex: dev-nvme0n1p3.swap)
    sudo systemctl mask dev-nvme0n1p3.swap
    sudo swapoff -a
    sudo nano /etc/fstab

    # Comment out the swap line by adding a `#` at the beginning of the line, then save and exit the file.
    sudo reboot
```

to check if swap is disabled:

```bash
    free -h
```

if swap is set to 0, then swap is disabled.

## Forward IPV4 traffic

```bash
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay && sudo modprobe br_netfilter

    # sysctl params required by setup, params persist across reboots
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Apply sysctl params without reboot
    sudo sysctl --system
```

## Install containerd

```bash

sudo apt-get update && sudo apt-get upgrade
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install containerd.io

```

## Install CNI plugins

```bash
sudo mkdir -p /opt/cni/bin

wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz

sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

## Systemd cgroup driver

```bash

sudo nano /etc/containerd/config.toml

# Enable the cni plugin by commenting out the line below
   #disabled_plugins = ["cri"]

# Add the following lines to the file:
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
   SystemdCgroup = true

```

## Restart containerd

```bash
sudo systemctl restart containerd
```

# Master node installation steps

## Install kubeadm, kubelet, and kubectl

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Enable kubelet, it will crash in loop because it waits for kubeadm to be enabled

```bash
sudo systemctl enable kubelet
```

Pull the kubeadm image (it can take a while)

```bash
sudo kubeadm config images pull
```

Enable kubeadm

```bash
sudo kubeadm init --pod-network-cidr=168.0.0/16
```

where POD_CIDR is the CIDR block that the pod IPs will be assigned from. The most common value is `192.168.0.0/16`.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install network plugin (Calico)

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Verify that the master node is ready

```bash
kubectl get nodes
```
