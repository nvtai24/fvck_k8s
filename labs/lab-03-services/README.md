# Lab 03 — Services & Ingress

## 🎯 Objectives
- Understand ClusterIP, NodePort, and LoadBalancer service types
- Expose pods via services and test connectivity
- Configure an Ingress with host and path-based routing
- Test service discovery using DNS within the cluster

## ⏱️ Estimated Time: 40–50 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab03
kubectl config set-context --current --namespace=lab03

# Enable ingress addon (minikube)
minikube addons enable ingress
```

---

## Exercise 1 — ClusterIP Service

**Task:** Fill in `exercise-01-clusterip.yaml` to:
1. Create a Deployment named `backend` with 2 replicas of `nginx:1.25` labeled `app: backend`
2. Create a ClusterIP Service named `backend-svc` targeting pods with `app: backend` on port `80`

**Test internal connectivity:**
```bash
# Create a temporary pod to test DNS resolution and connectivity
kubectl run test-pod --image=busybox:1.36 --rm -it -- sh

# Inside the pod:
wget -qO- backend-svc.lab03.svc.cluster.local
nslookup backend-svc.lab03.svc.cluster.local
```

---

## Exercise 2 — NodePort Service

**Task:** Fill in `exercise-02-nodeport.yaml` to:
1. Create a Deployment named `frontend` with 2 replicas of `nginx:1.25` labeled `app: frontend`
2. Create a NodePort Service on port `80` → `nodePort: 30080`

**Test external access:**
```bash
# minikube
minikube service frontend-svc -n lab03 --url

# kind / kubeadm
kubectl get nodes -o wide   # Get node IP
curl http://<NODE_IP>:30080
```

---

## Exercise 3 — Service Discovery with DNS

**Task:** Deploy two pods and verify they can communicate via service DNS:

1. Apply `exercise-03-dns-discovery.yaml` (provided - already filled in)
2. Exec into the `client` pod and test connectivity:

```bash
kubectl exec -it client-pod -n lab03 -- sh

# Inside pod - test DNS resolution:
nslookup server-svc
nslookup server-svc.lab03
nslookup server-svc.lab03.svc.cluster.local

# Test HTTP connectivity:
wget -qO- http://server-svc
wget -qO- http://server-svc.lab03.svc.cluster.local
```

---

## Exercise 4 — Ingress (Host + Path-based routing)

**Task:** Fill in `exercise-04-ingress.yaml` to:
1. Route `lab.local/api` → `backend-svc:80`
2. Route `lab.local/web` → `frontend-svc:80`

**Setup local DNS (add to /etc/hosts or C:\Windows\System32\drivers\etc\hosts):**
```
<minikube-ip>  lab.local
```
```bash
minikube ip   # Get minikube IP
echo "$(minikube ip)  lab.local" | sudo tee -a /etc/hosts
```

**Test:**
```bash
curl http://lab.local/api
curl http://lab.local/web
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab03
```

## 📖 Solutions
See `solutions/` folder.
