# Lab 01 — Pods

## 🎯 Objectives
By the end of this lab, you will be able to:
- Create and manage pods imperatively and declaratively
- Configure environment variables, resource limits, and restart policies
- Use liveness and readiness probes
- Build multi-container pods (sidecar pattern)
- Use init containers to run pre-start tasks

## ⏱️ Estimated Time: 30–45 minutes

## 📋 Prerequisites
```bash
# Create lab namespace
kubectl create namespace lab01

# Set default namespace for this lab
kubectl config set-context --current --namespace=lab01
```

---

## Exercise 1 — Create a Basic Pod

**Task:** Fill in the `# TODO` sections in `exercise-01-basic-pod.yaml` to:
1. Use the `nginx:1.25` image
2. Set container port to `80`
3. Add an environment variable `APP_ENV=production`
4. Set CPU request `100m`, memory request `64Mi`, CPU limit `200m`, memory limit `128Mi`
5. Add a liveness probe that checks `GET /` on port `80` every 10 seconds with 5s initial delay
6. Add a readiness probe that checks `GET /` on port `80` every 5 seconds with 3s initial delay

**Apply:**
```bash
kubectl apply -f exercise-01-basic-pod.yaml
```

**Verify:**
```bash
kubectl get pod basic-pod -n lab01
kubectl describe pod basic-pod -n lab01
kubectl logs basic-pod -n lab01

# Pod should show Running status and both probes
kubectl get pod basic-pod -n lab01 -o wide
```

**Expected:** Pod in `Running` state, `READY 1/1`

---

## Exercise 2 — Multi-Container Pod (Sidecar)

**Task:** Fill in `exercise-02-multicontainer.yaml` to:
1. Create a pod with 2 containers sharing an `emptyDir` volume
2. Container 1 (`writer`): uses `busybox:1.36`, writes the current date to `/shared/index.html` every 5 seconds in a loop
3. Container 2 (`reader`): uses `nginx:1.25`, serves files from `/usr/share/nginx/html` (mount the shared volume there)
4. Both containers should mount the same volume

**Apply:**
```bash
kubectl apply -f exercise-02-multicontainer.yaml
```

**Verify:**
```bash
kubectl get pod sidecar-pod -n lab01
kubectl logs sidecar-pod -c writer -n lab01
kubectl logs sidecar-pod -c reader -n lab01

# Exec into nginx and check the file is being updated
kubectl exec -it sidecar-pod -c reader -n lab01 -- cat /usr/share/nginx/html/index.html
```

---

## Exercise 3 — Init Container

**Task:** Fill in `exercise-03-init-container.yaml` to:
1. Create a pod with 1 init container and 1 main container
2. Init container (`setup`): uses `busybox:1.36`, writes `<h1>Initialized!</h1>` to `/workdir/index.html`
3. Main container (`nginx`): serves the initialized file from nginx
4. Both containers share a volume named `workdir`

**Apply:**
```bash
kubectl apply -f exercise-03-init-container.yaml
```

**Verify:**
```bash
kubectl get pod init-pod -n lab01
# Watch init container complete, then main container start
kubectl get pod init-pod -n lab01 -w

# Check the content was initialized
kubectl exec -it init-pod -n lab01 -- cat /usr/share/nginx/html/index.html
# Should output: <h1>Initialized!</h1>
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab01
```

## 📖 Solutions
See the `solutions/` folder. Try the exercises first!
