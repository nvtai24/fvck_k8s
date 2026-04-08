# Lab 04 — ConfigMaps & Secrets

## 🎯 Objectives
- Create ConfigMaps from literals, files, and YAML
- Inject config as environment variables and mounted files
- Create and consume Secrets securely
- Understand the difference between env-based and volume-based injection

## ⏱️ Estimated Time: 30 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab04
kubectl config set-context --current --namespace=lab04
```

---

## Exercise 1 — ConfigMap as Environment Variables

**Task:** Fill in `exercise-01-configmap-env.yaml` to:
1. Create a ConfigMap named `app-config` with:
   - `APP_ENV=production`
   - `APP_PORT=8080`
   - `LOG_LEVEL=info`
2. Create a pod that injects **all** ConfigMap keys as environment variables

**Verify:**
```bash
kubectl apply -f exercise-01-configmap-env.yaml
kubectl exec -it configmap-pod -n lab04 -- env | grep -E "APP|LOG"
# Should show APP_ENV=production, APP_PORT=8080, LOG_LEVEL=info
```

---

## Exercise 2 — ConfigMap as Mounted Files

**Task:** Fill in `exercise-02-configmap-volume.yaml` to:
1. Create a ConfigMap named `nginx-config` with key `nginx.conf` containing a basic nginx config
2. Create an nginx pod that mounts the ConfigMap as a file at `/etc/nginx/conf.d/default.conf`

**Verify:**
```bash
kubectl apply -f exercise-02-configmap-volume.yaml
kubectl exec -it configmap-vol-pod -n lab04 -- cat /etc/nginx/conf.d/default.conf
```

---

## Exercise 3 — Secrets

**Task:** Fill in `exercise-03-secret.yaml` to:
1. Create an Opaque Secret named `db-secret` with:
   - `username: admin`
   - `password: S3cr3tP@ss!`
   
   > ⚠️ Use `stringData` (not `data`) so you don't need to manually base64 encode

2. Create a pod that:
   - Injects `username` as env var `DB_USER`
   - Injects `password` as env var `DB_PASS`
   - Mounts the entire secret at `/etc/secrets`

**Verify:**
```bash
kubectl apply -f exercise-03-secret.yaml
kubectl exec -it secret-pod -n lab04 -- env | grep DB_
kubectl exec -it secret-pod -n lab04 -- ls /etc/secrets
kubectl exec -it secret-pod -n lab04 -- cat /etc/secrets/password
```

---

## Exercise 4 — Imperative Commands (No YAML)

Practice creating ConfigMaps and Secrets imperatively:

```bash
# TODO: Create a ConfigMap named "env-config" with key COLOR=green
kubectl create configmap ...

# TODO: Create a ConfigMap from a file (create a test file first)
echo "key1=value1" > /tmp/myconfig.txt
kubectl create configmap ...

# TODO: Create a Secret named "my-secret" with key token=abc123
kubectl create secret generic ...

# Verify
kubectl get configmap env-config -o yaml
kubectl get secret my-secret -o yaml
# Decode the secret value:
kubectl get secret my-secret -o jsonpath='{.data.token}' | base64 -d
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab04
```

## 📖 Solutions
See `solutions/` folder.
