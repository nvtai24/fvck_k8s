# Lab 11 — Capstone: Full-Stack App Deployment

## 🎯 Objective
Deploy a complete 3-tier application entirely from scratch using **only the knowledge you built in Labs 01–10**.  
No step-by-step instructions — only the requirements. You decide the approach.

## ⏱️ Estimated Time: 60–90 minutes

## 📋 Setup
```bash
kubectl create namespace capstone
kubectl config set-context --current --namespace=capstone
```

---

## 🏗️ Architecture to Deploy

```
Internet
   │
[Ingress]  →  lab11.local
   │
   ├── /     → [frontend Service] → [frontend Deployment]  (2 replicas)
   │
   └── /api  → [backend Service]  → [backend Deployment]   (2 replicas)
                                          │
                               [mysql-headless Service]
                                          │
                               [mysql StatefulSet]         (1 replica)
```

---

## ✅ Requirements

### Namespace & Config
- [ ] All resources in namespace `capstone`
- [ ] Create a ConfigMap `app-config` with:
  - `DB_HOST=mysql-headless.capstone.svc.cluster.local`
  - `DB_PORT=3306`
  - `DB_NAME=capstone_db`
- [ ] Create a Secret `app-secret` with:
  - `MYSQL_ROOT_PASSWORD=RootPass123!`
  - `DB_PASSWORD=AppPass456!`

### MySQL (StatefulSet)
- [ ] StatefulSet named `mysql`, 1 replica
- [ ] Image: `mysql:8.0`
- [ ] Env: `MYSQL_ROOT_PASSWORD` and `MYSQL_DATABASE` from ConfigMap/Secret
- [ ] Liveness probe: `mysqladmin ping`
- [ ] Requests: `cpu: 250m, memory: 512Mi`, Limits: `cpu: 1, memory: 1Gi`
- [ ] PVC via `volumeClaimTemplates`: `1Gi, ReadWriteOnce`
- [ ] Headless Service: `mysql-headless`, port `3306`

### Backend (Deployment)
- [ ] Deployment named `backend`, 2 replicas of `nginx:1.25` (simulates backend)
- [ ] Inject all keys from `app-config` and `app-secret` as env vars
- [ ] Readiness probe on `GET / port 80`
- [ ] Requests: `cpu: 100m, memory: 128Mi`, Limits: `cpu: 500m, memory: 256Mi`
- [ ] ClusterIP Service: `backend-svc`, port `80`
- [ ] Pod anti-affinity: prefer different nodes

### Frontend (Deployment)
- [ ] Deployment named `frontend`, 2 replicas of `nginx:1.25`
- [ ] Liveness and readiness probes on port `80`
- [ ] Requests: `cpu: 100m, memory: 64Mi`, Limits: `cpu: 200m, memory: 128Mi`
- [ ] ClusterIP Service: `frontend-svc`, port `80`

### Ingress
- [ ] Ingress named `capstone-ingress`
- [ ] Host: `lab11.local`
- [ ] Route `/` → `frontend-svc:80`
- [ ] Route `/api` → `backend-svc:80`

### RBAC
- [ ] ServiceAccount `backend-sa` in `capstone` namespace
- [ ] Role that allows `get`, `list` on `pods` and `configmaps`
- [ ] RoleBinding: bind role to `backend-sa`
- [ ] Backend Deployment uses `backend-sa`

### HPA
- [ ] HPA for `backend` deployment
- [ ] Min: `2`, Max: `6`
- [ ] Scale when CPU > `60%`

### PodDisruptionBudget
- [ ] PDB for `backend`: `minAvailable: 1`
- [ ] PDB for `frontend`: `minAvailable: 1`

---

## ✔️ Verification Checklist

Run through these after applying all your manifests:

```bash
# All pods running
kubectl get pods -n capstone

# PVC bound
kubectl get pvc -n capstone

# Services available
kubectl get svc -n capstone

# Ingress created
kubectl get ingress -n capstone

# HPA watching
kubectl get hpa -n capstone

# RBAC permissions check
kubectl auth can-i list pods \
  --as=system:serviceaccount:capstone:backend-sa -n capstone

# Test internal connectivity
kubectl run test --image=busybox:1.36 --rm -it -n capstone -- \
  wget -qO- http://backend-svc

# Test ingress (add to hosts file first)
# echo "$(minikube ip)  lab11.local" | sudo tee -a /etc/hosts
curl http://lab11.local/
curl http://lab11.local/api
```

---

## 📁 Your Work

Put your manifest files in this folder:
- `capstone-namespace-config.yaml` — Namespace, ConfigMap, Secret
- `capstone-mysql.yaml` — StatefulSet + Headless Service
- `capstone-backend.yaml` — Deployment + Service
- `capstone-frontend.yaml` — Deployment + Service
- `capstone-ingress.yaml` — Ingress
- `capstone-rbac.yaml` — SA + Role + RoleBinding
- `capstone-autoscaling.yaml` — HPA + PDB

Or combine them all in one `capstone-all.yaml` — your choice!

---

## 📖 Reference Solution
See `solutions/capstone-solution.yaml` — only look after you've tried!

## 🧹 Cleanup
```bash
kubectl delete namespace capstone
```
