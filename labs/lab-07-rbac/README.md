# Lab 07 — RBAC (Role-Based Access Control)

## 🎯 Objectives
- Create ServiceAccounts and understand their role
- Create Roles and ClusterRoles with specific permissions
- Bind roles to ServiceAccounts via RoleBindings
- Verify permissions with `kubectl auth can-i`

## ⏱️ Estimated Time: 35 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab07
kubectl config set-context --current --namespace=lab07
```

---

## Exercise 1 — ServiceAccount + Role + RoleBinding

**Task:** Fill in `exercise-01-rbac.yaml` to:
1. Create a ServiceAccount named `pod-reader-sa` in `lab07`
2. Create a Role named `pod-reader-role` that allows: `get`, `list`, `watch` on `pods`
3. Create a RoleBinding that binds the role to the ServiceAccount

**Verify permissions:**
```bash
kubectl apply -f exercise-01-rbac.yaml

# Check: can pod-reader-sa list pods?
kubectl auth can-i list pods \
  --as=system:serviceaccount:lab07:pod-reader-sa -n lab07
# Expected: yes

# Check: can pod-reader-sa create pods? (should be denied)
kubectl auth can-i create pods \
  --as=system:serviceaccount:lab07:pod-reader-sa -n lab07
# Expected: no

# Check: can pod-reader-sa list deployments? (should be denied)
kubectl auth can-i list deployments \
  --as=system:serviceaccount:lab07:pod-reader-sa -n lab07
# Expected: no
```

---

## Exercise 2 — Pod using ServiceAccount

**Task:** Fill in `exercise-02-pod-with-sa.yaml` to:
1. Create a pod that uses `pod-reader-sa`
2. Inside the pod, use `kubectl` or the API to list pods (to prove the SA token works)

```bash
kubectl apply -f exercise-02-pod-with-sa.yaml

# Exec in and try to use the SA token
kubectl exec -it sa-demo-pod -n lab07 -- sh

# Inside pod — call the Kubernetes API using the mounted SA token:
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -sk -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/lab07/pods | head -50
```

---

## Exercise 3 — ClusterRole (Cross-Namespace)

**Task:** Fill in `exercise-03-clusterrole.yaml` to:
1. Create a ClusterRole named `node-viewer` that allows `get`, `list`, `watch` on `nodes`
2. Create a ClusterRoleBinding that grants `node-viewer` to `pod-reader-sa`

**Verify:**
```bash
kubectl apply -f exercise-03-clusterrole.yaml

kubectl auth can-i list nodes \
  --as=system:serviceaccount:lab07:pod-reader-sa
# Expected: yes

kubectl auth can-i delete nodes \
  --as=system:serviceaccount:lab07:pod-reader-sa
# Expected: no
```

---

## Exercise 4 — Deny All by Default

**Observe:** What can a pod with the **default** ServiceAccount do?

```bash
# Run a pod with default SA (no explicit serviceAccountName)
kubectl run default-sa-pod --image=bitnami/kubectl:latest \
  --command -- sleep 3600 -n lab07

kubectl exec -it default-sa-pod -n lab07 -- \
  kubectl get pods -n lab07
# Should be FORBIDDEN (default SA has no permissions in most clusters)
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab07
kubectl delete clusterrole node-viewer
kubectl delete clusterrolebinding node-viewer-binding
```

## 📖 Solutions
See `solutions/` folder.
