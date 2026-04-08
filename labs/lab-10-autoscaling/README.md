# Lab 10 — Autoscaling & Resource Management

## 🎯 Objectives
- Set resource requests and limits on pods
- Understand QoS classes (Guaranteed, Burstable, BestEffort)
- Create a HorizontalPodAutoscaler and trigger it with load
- Apply ResourceQuota and LimitRange at namespace level

## ⏱️ Estimated Time: 45 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab10
kubectl config set-context --current --namespace=lab10

# Metrics server must be running
kubectl get deployment metrics-server -n kube-system
# If not installed: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## Exercise 1 — Resource Requests & Limits + QoS

**Task:** Fill in `exercise-01-resources.yaml` to create 3 pods demonstrating the 3 QoS classes:
1. **Guaranteed pod**: `requests == limits` for both CPU and memory
2. **Burstable pod**: only `requests` set (no limits), or requests < limits
3. **BestEffort pod**: no `resources` block at all

**Verify:**
```bash
kubectl apply -f exercise-01-resources.yaml

kubectl get pods -n lab10

# Check QoS class of each pod
kubectl get pod guaranteed-pod -n lab10 -o jsonpath='{.status.qosClass}'
kubectl get pod burstable-pod -n lab10 -o jsonpath='{.status.qosClass}'
kubectl get pod besteffort-pod -n lab10 -o jsonpath='{.status.qosClass}'
```

---

## Exercise 2 — HPA (Horizontal Pod Autoscaler)

**Task:** Fill in `exercise-02-hpa.yaml` to:
1. Create a Deployment named `load-test-app` with 1 replica of `nginx:1.25`
2. Set resource requests: `cpu: 100m, memory: 64Mi` and limits: `cpu: 200m, memory: 128Mi`
3. Create an HPA that scales between `1` and `5` replicas targeting `50%` CPU utilization

**Apply and generate load:**
```bash
kubectl apply -f exercise-02-hpa.yaml

# Check HPA status (may show <unknown> until metrics are available)
kubectl get hpa -n lab10 -w

# Generate CPU load in another terminal
kubectl run load-gen --image=busybox:1.36 --rm -it -- \
  sh -c "while true; do wget -q -O- http://load-test-svc.lab10.svc.cluster.local; done"

# Watch HPA scale up
kubectl get hpa -n lab10 -w
kubectl get pods -n lab10 -w
```

---

## Exercise 3 — ResourceQuota

**Task:** Fill in `exercise-03-quota.yaml` to set a ResourceQuota on `lab10` that allows:
- Max `2` pods
- Max CPU requests: `500m`
- Max memory requests: `512Mi`

**Test the quota:**
```bash
kubectl apply -f exercise-03-quota.yaml
kubectl describe resourcequota -n lab10

# Try to create more than 2 pods — the 3rd should be rejected
kubectl run pod1 --image=nginx:1.25 -n lab10
kubectl run pod2 --image=nginx:1.25 -n lab10
kubectl run pod3 --image=nginx:1.25 -n lab10
# pod3 should fail: "exceeded quota"
```

---

## Exercise 4 — LimitRange (default limits)

**Task:** Fill in `exercise-04-limitrange.yaml` to create a LimitRange that:
- Sets default CPU request: `100m`, limit: `300m`
- Sets default memory request: `64Mi`, limit: `128Mi`
- Sets minimum CPU: `50m`, minimum memory: `32Mi`

**Test:**
```bash
kubectl apply -f exercise-04-limitrange.yaml

# Create a pod with NO resource spec — LimitRange should inject defaults
kubectl run no-resources-pod --image=nginx:1.25 -n lab10
kubectl get pod no-resources-pod -n lab10 -o jsonpath='{.spec.containers[0].resources}' | python3 -m json.tool
# Should show the default limits applied by LimitRange
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab10
```

## 📖 Solutions
See `solutions/` folder.
