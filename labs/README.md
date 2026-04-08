# ☸️ Kubernetes Hands-On Labs

> **Practice > Theory.** Each lab is self-contained with objectives, tasks, verification steps, and solutions.

---

## 🗂️ Lab Structure

```
labs/
├── lab-01-pods/           Core pod concepts
├── lab-02-deployments/    Deployments, rolling updates, rollbacks
├── lab-03-services/       All service types + Ingress
├── lab-04-config/         ConfigMaps & Secrets
├── lab-05-storage/        PV, PVC, StorageClass
├── lab-06-workloads/      StatefulSet, DaemonSet, Job, CronJob
├── lab-07-rbac/           ServiceAccount, Role, RoleBinding
├── lab-08-scheduling/     Affinity, Taints & Tolerations
├── lab-09-networking/     NetworkPolicy
├── lab-10-autoscaling/    HPA + resource management
└── lab-11-capstone/       Full-stack app combining everything
```

---

## ⚙️ Prerequisites

| Tool | Version | Install |
|---|---|---|
| `kubectl` | >= 1.28 | https://kubernetes.io/docs/tasks/tools/ |
| Local cluster | minikube / kind / k3s | See below |

### Option A — Minikube (recommended for beginners)
```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --cpus=4 --memory=4096 --driver=docker

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
```

### Option B — kind (Kubernetes in Docker)
```bash
# Install kind
go install sigs.k8s.io/kind@latest

# Create cluster
kind create cluster --name k8s-lab

# Get kubeconfig
kind get kubeconfig --name k8s-lab > ~/.kube/config
```

### Verify your cluster
```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

---

## 📋 Lab Progress Tracker

| Lab | Topic | Status |
|---|---|---|
| Lab 01 | Pods | ⬜ Not started |
| Lab 02 | Deployments | ⬜ Not started |
| Lab 03 | Services | ⬜ Not started |
| Lab 04 | ConfigMap & Secrets | ⬜ Not started |
| Lab 05 | Storage | ⬜ Not started |
| Lab 06 | Workloads | ⬜ Not started |
| Lab 07 | RBAC | ⬜ Not started |
| Lab 08 | Scheduling | ⬜ Not started |
| Lab 09 | NetworkPolicy | ⬜ Not started |
| Lab 10 | Autoscaling | ⬜ Not started |
| Lab 11 | Capstone | ⬜ Not started |

---

## 🧹 Cleanup Between Labs

```bash
# Delete all resources in a namespace
kubectl delete all --all -n lab

# Or delete and recreate the namespace
kubectl delete namespace lab
kubectl create namespace lab
```

---

## 💡 Tips

- Always **read the README** in each lab folder before starting
- Try to fill in the `# TODO` sections yourself before looking at solutions
- Use `kubectl describe` and `kubectl logs` to debug
- `kubectl explain <resource>.<field>` is your best friend
