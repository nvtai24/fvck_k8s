# Lab 05 — Storage (PV, PVC, StorageClass)

## 🎯 Objectives
- Understand PersistentVolume, PersistentVolumeClaim, and StorageClass
- Create and bind PVs and PVCs
- Mount persistent storage into pods
- Observe data persistence across pod restarts

## ⏱️ Estimated Time: 35 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab05
kubectl config set-context --current --namespace=lab05
```

---

## Exercise 1 — Static PV + PVC + Pod

**Task:** Fill in `exercise-01-pv-pvc.yaml` to:
1. Create a PersistentVolume:
   - `capacity: 1Gi`
   - `accessModes: ReadWriteOnce`
   - Use `hostPath: /tmp/lab05-pv` (works on minikube)
   - `reclaimPolicy: Retain`
   - `storageClassName: manual`
2. Create a PersistentVolumeClaim:
   - Request `500Mi` of storage
   - Use `storageClassName: manual` to match the PV
3. Create a pod that mounts the PVC at `/data`

**Verify:**
```bash
kubectl apply -f exercise-01-pv-pvc.yaml

kubectl get pv
kubectl get pvc -n lab05
# PVC should be in "Bound" state

# Write data to the persistent volume
kubectl exec -it pv-pod -n lab05 -- sh -c "echo 'persistent data' > /data/test.txt"
kubectl exec -it pv-pod -n lab05 -- cat /data/test.txt
```

---

## Exercise 2 — Data Persistence Test

**Task:** (No YAML — all kubectl commands)

Prove that data persists across pod deletion and recreation:

```bash
# 1. Write data to the pod
kubectl exec -it pv-pod -n lab05 -- sh -c "echo 'I survive pod restarts!' > /data/persistent.txt"

# 2. Delete the pod
kubectl delete pod pv-pod -n lab05

# 3. Apply again to recreate the pod
kubectl apply -f exercise-01-pv-pvc.yaml

# 4. Verify data is still there
kubectl exec -it pv-pod -n lab05 -- cat /data/persistent.txt
# Should still show: "I survive pod restarts!"
```

---

## Exercise 3 — Dynamic Provisioning (StorageClass)

**Task:** Fill in `exercise-03-dynamic-pvc.yaml` to:
1. Create a PVC using the default StorageClass (don't specify `storageClassName`, or use the cluster default)
2. Mount it in a pod

```bash
# Check available storage classes
kubectl get storageclass

kubectl apply -f exercise-03-dynamic-pvc.yaml
kubectl get pvc -n lab05
# Should auto-provision a PV and bind immediately
```

---

## Exercise 4 — emptyDir vs hostPath vs PVC

**Task:** Apply `exercise-04-volume-types.yaml` and observe the difference:
- `emptyDir`: data lost when pod is deleted
- `hostPath`: data on the node, survives pod restart (but tied to node)
- `PVC`: data persists independent of pod and node

```bash
kubectl apply -f exercise-04-volume-types.yaml
kubectl get pods -n lab05
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab05
kubectl delete pv lab05-pv   # PVs are cluster-scoped, not namespace-scoped
```

## 📖 Solutions
See `solutions/` folder.
