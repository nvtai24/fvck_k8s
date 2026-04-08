# Lab 09 — NetworkPolicy

## 🎯 Objectives
- Understand that by default all pods can talk to all pods
- Apply a default-deny policy and observe traffic being blocked
- Selectively allow traffic between specific pods
- Restrict egress traffic

## ⏱️ Estimated Time: 40 minutes

## ⚠️ Prerequisite: CNI with NetworkPolicy support
NetworkPolicy requires a CNI plugin that supports it. Verify:
```bash
# On minikube — use calico or cilium driver:
minikube start --cni=calico

# Or install Calico manually:
# kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
```

## 📋 Setup
```bash
kubectl create namespace lab09
kubectl config set-context --current --namespace=lab09

# Deploy test pods
kubectl apply -f setup-pods.yaml
```

---

## Exercise 1 — Observe Default Open Traffic

```bash
# Apply the setup (3 pods: frontend, backend, database)
kubectl apply -f setup-pods.yaml

# By default, all pods can talk to each other.
# Test from frontend → backend:
kubectl exec -it frontend-pod -n lab09 -- wget -qO- http://backend-svc
# Should succeed

# Test from frontend → database:
kubectl exec -it frontend-pod -n lab09 -- wget -qO- http://db-svc
# Should succeed (we want to block this in Exercise 2)
```

---

## Exercise 2 — Default Deny All

**Task:** Fill in `exercise-02-default-deny.yaml` to apply a default-deny-all policy to the `lab09` namespace.

```bash
kubectl apply -f exercise-02-default-deny.yaml

# Now ALL traffic is blocked
kubectl exec -it frontend-pod -n lab09 -- wget -qO- --timeout=5 http://backend-svc
# Should FAIL (timeout)
```

---

## Exercise 3 — Allow Frontend → Backend

**Task:** Fill in `exercise-03-allow-frontend-backend.yaml` to:
1. Allow ingress to `backend` pods FROM pods with label `app: frontend`
2. Allow only port `80`

```bash
kubectl apply -f exercise-03-allow-frontend-backend.yaml

# frontend → backend: should work
kubectl exec -it frontend-pod -n lab09 -- wget -qO- --timeout=5 http://backend-svc
# ✅ Should succeed

# frontend → database: should still be BLOCKED
kubectl exec -it frontend-pod -n lab09 -- wget -qO- --timeout=5 http://db-svc
# ❌ Should fail (timeout)
```

---

## Exercise 4 — Allow Backend → Database Only

**Task:** Fill in `exercise-04-allow-backend-db.yaml` to:
1. Allow ingress to `database` pods ONLY FROM pods with label `app: backend`
2. Allow port `80` (simulated — in reality this would be 5432 for postgres)

```bash
kubectl apply -f exercise-04-allow-backend-db.yaml

# backend → database: should work
kubectl exec -it backend-pod -n lab09 -- wget -qO- --timeout=5 http://db-svc
# ✅ Should succeed

# frontend → database: still blocked
kubectl exec -it frontend-pod -n lab09 -- wget -qO- --timeout=5 http://db-svc
# ❌ Should fail
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab09
```

## 📖 Solutions
See `solutions/` folder.
