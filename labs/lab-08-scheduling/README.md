# Lab 08 — Scheduling & Placement

## 🎯 Objectives
- Use `nodeSelector` and `nodeAffinity` to control where pods land
- Use `podAntiAffinity` to spread replicas across nodes
- Apply taints to nodes and tolerations to pods
- Observe what happens when scheduling constraints can't be met

## ⏱️ Estimated Time: 40 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab08
kubectl config set-context --current --namespace=lab08

# Label your nodes for the exercises
kubectl get nodes
kubectl label node <your-node-name> disktype=ssd
kubectl label node <your-node-name> region=us-east
```

---

## Exercise 1 — nodeSelector

**Task:** Fill in `exercise-01-nodeselector.yaml`:
1. Create a pod that uses `nodeSelector` to target nodes with `disktype=ssd`

**Verify:**
```bash
kubectl apply -f exercise-01-nodeselector.yaml
kubectl get pod nodeselector-pod -n lab08 -o wide
# Should show it landed on the node you labeled with disktype=ssd

# What if no node matches? Try using a label that doesn't exist:
kubectl run unschedulable-pod --image=nginx:1.25 \
  --overrides='{"spec":{"nodeSelector":{"gpu":"true"}}}' -n lab08
kubectl get pod unschedulable-pod -n lab08
# Status: Pending — because no node has gpu=true
```

---

## Exercise 2 — Node Affinity

**Task:** Fill in `exercise-02-node-affinity.yaml`:
1. HARD rule (required): pod MUST go to a node with `kubernetes.io/os=linux`
2. SOFT rule (preferred): prefer nodes with `region=us-east`

**Verify:**
```bash
kubectl apply -f exercise-02-node-affinity.yaml
kubectl get pod affinity-pod -n lab08 -o wide
kubectl describe pod affinity-pod -n lab08 | grep -A5 "Events"
```

---

## Exercise 3 — Pod Anti-Affinity (Spread Replicas)

**Task:** Fill in `exercise-03-pod-antiaffinity.yaml`:
1. Create a Deployment with 3 replicas of nginx
2. Use `podAntiAffinity` (preferred) to spread pods across different nodes
   - `topologyKey: kubernetes.io/hostname`

**Verify:**
```bash
kubectl apply -f exercise-03-pod-antiaffinity.yaml
kubectl get pods -n lab08 -o wide
# Pods should be on different nodes if available
```

---

## Exercise 4 — Taints & Tolerations

**Task:** 

**4a. Taint a node:**
```bash
# Add a taint to your node
kubectl taint nodes <node-name> dedicated=lab:NoSchedule

# Verify: try creating a pod WITHOUT tolerations (should stay Pending)
kubectl run no-toleration-pod --image=nginx:1.25 -n lab08
kubectl get pod no-toleration-pod -n lab08  # Should be Pending
```

**4b. Fill in `exercise-04-toleration.yaml`** to create a pod with a toleration for `dedicated=lab:NoSchedule`

```bash
kubectl apply -f exercise-04-toleration.yaml
kubectl get pod toleration-pod -n lab08
# Should be Running (tolerates the taint)

# Cleanup: remove the taint
kubectl taint nodes <node-name> dedicated=lab:NoSchedule-
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab08
kubectl taint nodes <node-name> dedicated=lab:NoSchedule-   # Remove taint if still there
kubectl label node <node-name> disktype-                     # Remove label
kubectl label node <node-name> region-
```

## 📖 Solutions
See `solutions/` folder.
