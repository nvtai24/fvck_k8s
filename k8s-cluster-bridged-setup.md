# Kubernetes Cluster Setup Guide (Bridged Network)
## 1 Master + 2 Workers | VMware Bridged | Ubuntu 24.04

---

## Cluster Architecture

| Node | Hostname | IP (Static) | Role |
|------|----------|-------------|------|
| Master | k8s-master | 192.168.0.110 | control-plane |
| Worker 1 | k8s-node2 | 192.168.0.111 | worker |
| Worker 2 | k8s-node3 | 192.168.0.112 | worker |

---

## Prerequisites

- VMware Workstation with 3 VMs running Ubuntu 24.04
- Each VM: 2 CPU, 4GB RAM, 20GB disk (minimum)
- VM → Settings → Network Adapter → **Bridged (Automatic)**

---

## Step 1: Set Static IP (All 3 Nodes)

> Change the IP per node: `.110` master, `.111` worker1, `.112` worker2

```bash
sudo tee /etc/netplan/50-cloud-init.yaml <<EOF
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.0.110/24
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
EOF

sudo netplan apply
```

Verify:
```bash
ping -c 4 8.8.8.8
ping -c 4 192.168.0.1
```

---

## Step 2: System Preparation (All 3 Nodes)

```bash
# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Disable firewall
sudo ufw disable

# Load kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Configure sysctl
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## Step 3: Install containerd (All 3 Nodes)

```bash
sudo apt-get update && sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list

sudo apt-get update && sudo apt-get install -y containerd.io
```

Configure containerd:
```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## Step 4: Install kubeadm, kubelet, kubectl (All 3 Nodes)

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## Step 5: Initialize the Cluster (Master Only)

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=192.168.0.110
```

> ⚠️ Copy the `kubeadm join ...` command from the output — needed in Step 8.

Set up kubectl:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Step 6: Install Calico Network Plugin (Master Only)

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

Wait for master to become Ready:
```bash
watch kubectl get nodes
```

---

## Step 7: Join Worker Nodes (Worker 1 and Worker 2)

```bash
sudo kubeadm join 192.168.0.110:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

> If token expired, regenerate on master:
> ```bash
> kubeadm token create --print-join-command
> ```

---

## Step 8: Verify Cluster (Master Only)

```bash
kubectl get nodes -o wide
```

Expected output:
```
NAME         STATUS   ROLES           AGE   VERSION   INTERNAL-IP
k8s-master   Ready    control-plane   5m    v1.29.x   192.168.0.110
k8s-node2    Ready    <none>          2m    v1.29.x   192.168.0.111
k8s-node3    Ready    <none>          2m    v1.29.x   192.168.0.112
```

Check all system pods are running:
```bash
kubectl get pods -n kube-system
```

---

## Troubleshooting

### Reset a node
```bash
sudo kubeadm reset
sudo rm -rf /etc/cni/net.d
sudo iptables -F && sudo iptables -t nat -F
```

### Check kubelet status
```bash
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
```

### Node not Ready
```bash
kubectl describe node <node-name>
kubectl get pods -n kube-system
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `kubectl get nodes -o wide` | List nodes with IPs |
| `kubectl get pods -A` | All pods in all namespaces |
| `kubectl get pods -o wide` | Show pod placement per node |
| `kubectl describe node <n>` | Detailed node info |
| `kubectl drain <node> --ignore-daemonsets` | Safely evict pods |
| `kubectl cordon <node>` | Mark node unschedulable |
| `kubectl uncordon <node>` | Mark node schedulable |
| `kubeadm token create --print-join-command` | Generate new join command |
