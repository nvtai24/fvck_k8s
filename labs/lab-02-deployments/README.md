# Lab 02 — Deployments

## 🎯 Objectives
- Create a Deployment from scratch
- Scale replicas up and down
- Perform rolling updates with zero downtime
- Rollback a bad deployment
- Understand `maxSurge` and `maxUnavailable`

## ⏱️ Estimated Time: 30–45 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab02
kubectl config set-context --current --namespace=lab02
```

---

## Exercise 1 — Create a Deployment

**Task:** Fill in `exercise-01-deployment.yaml` to create a Deployment that:
1. Has `3` replicas of `nginx:1.24` (intentionally old version)
2. Uses `RollingUpdate` strategy with `maxSurge: 1` and `maxUnavailable: 0`
3. Sets a liveness probe on `GET / port 80` with 10s initial delay
4. Sets resource requests: `cpu: 100m, memory: 64Mi`
5. Sets resource limits: `cpu: 200m, memory: 128Mi`
6. Has label `app: web` on both the Deployment and pod template

**Apply and verify:**
```bash
kubectl apply -f exercise-01-deployment.yaml
kubectl get deployment web -n lab02
kubectl get pods -n lab02
kubectl describe deployment web -n lab02
```

---

## Exercise 2 — Scale and Update

**Task:** (Run these commands directly — no YAML needed)

**2a. Scale the deployment to 5 replicas:**
```bash
# TODO: scale the "web" deployment to 5 replicas using kubectl scale
```

**2b. Update the nginx image to `nginx:1.25` (rolling update):**
```bash
# TODO: update the image using kubectl set image
# Watch the rollout in real time
kubectl rollout status deployment/web -n lab02
kubectl get pods -n lab02 -w
```

**2c. Check rollout history:**
```bash
# TODO: show the rollout history of the "web" deployment
```

**Verify:** All 5 pods should eventually be running `nginx:1.25`
```bash
kubectl get pods -n lab02 -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
```

---

## Exercise 3 — Rollback

**Task:**

**3a. Simulate a bad deployment — update to a broken image:**
```bash
kubectl set image deployment/web nginx=nginx:BROKEN_VERSION -n lab02
# Watch pods failing
kubectl get pods -n lab02 -w
# Wait ~30 seconds, then rollback
```

**3b. Rollback to the previous version:**
```bash
# TODO: roll back the deployment using kubectl rollout undo
```

**3c. Rollback to a specific revision:**
```bash
# TODO: rollback to revision 1 specifically
# Hint: use --to-revision flag
```

**Verify:**
```bash
kubectl rollout history deployment/web -n lab02
kubectl get pods -n lab02
# All pods should be running again
```

---

## Exercise 4 — Deployment with ConfigMap (Bonus)

**Task:** Update `exercise-04-deployment-configmap.yaml` to:
1. Create a ConfigMap with `APP_COLOR=blue` and `APP_VERSION=1.0`
2. Inject these as environment variables into the nginx container
3. Apply and verify:
```bash
kubectl exec -it <pod-name> -n lab02 -- env | grep APP
```

---

## 📝 Key Commands Summary
```bash
kubectl create deployment <name> --image=<image> --replicas=3   # Imperative create
kubectl scale deployment <name> --replicas=5
kubectl set image deployment/<name> <container>=<new-image>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<n>
```

## 🧹 Cleanup
```bash
kubectl delete namespace lab02
```

## 📖 Solutions
See `solutions/` folder.
