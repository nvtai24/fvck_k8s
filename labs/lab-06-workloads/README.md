# Lab 06 — Workloads (StatefulSet, DaemonSet, Job, CronJob)

## 🎯 Objectives
- Understand when to use StatefulSet vs Deployment
- Deploy a DaemonSet and see it run on every node
- Create Jobs for one-off tasks and observe completion
- Schedule CronJobs and manually trigger them

## ⏱️ Estimated Time: 45 minutes

## 📋 Prerequisites
```bash
kubectl create namespace lab06
kubectl config set-context --current --namespace=lab06
```

---

## Exercise 1 — StatefulSet

**Task:** Fill in `exercise-01-statefulset.yaml` to create a StatefulSet:
1. Name: `web-sts`, 3 replicas of `nginx:1.25`
2. Requires a headless service named `web-headless`
3. Each pod gets its own PVC via `volumeClaimTemplates` (500Mi, `ReadWriteOnce`)
4. Mount the PVC at `/usr/share/nginx/html`

**Verify:**
```bash
kubectl apply -f exercise-01-statefulset.yaml

# Observe ordered pod creation
kubectl get pods -n lab06 -w

# Pods should be named: web-sts-0, web-sts-1, web-sts-2
kubectl get pods -n lab06

# Each pod has its own PVC
kubectl get pvc -n lab06

# Test stable DNS names
kubectl run dns-test --image=busybox:1.36 --rm -it -- nslookup web-sts-0.web-headless.lab06.svc.cluster.local
```

---

## Exercise 2 — DaemonSet

**Task:** Fill in `exercise-02-daemonset.yaml` to:
1. Create a DaemonSet that runs `nginx:1.25` on every node
2. Use `updateStrategy: RollingUpdate`

**Verify:**
```bash
kubectl apply -f exercise-02-daemonset.yaml
kubectl get daemonset -n lab06
kubectl get pods -n lab06 -o wide
# One pod per node
```

---

## Exercise 3 — Job

**Task:** Fill in `exercise-03-job.yaml` to:
1. Create a Job named `pi-calculator`
2. Runs `perl -Mbignum=bpi -wle 'print bpi(2000)'` to compute Pi (use image `perl:5.34`)
3. Set `restartPolicy: Never` and `backoffLimit: 2`

**Verify:**
```bash
kubectl apply -f exercise-03-job.yaml
kubectl get jobs -n lab06 -w
kubectl logs -l job-name=pi-calculator -n lab06
# Should print Pi to 2000 decimal places
```

---

## Exercise 4 — CronJob

**Task:** Fill in `exercise-04-cronjob.yaml` to:
1. Create a CronJob that runs every minute: `* * * * *`
2. Runs `busybox:1.36` with command: `date && echo "Hello from CronJob"`
3. Keep 3 successful job histories, 1 failed

**Verify:**
```bash
kubectl apply -f exercise-04-cronjob.yaml
kubectl get cronjob -n lab06

# Watch jobs being created every minute
kubectl get jobs -n lab06 -w

# Check logs of a completed job
kubectl logs -l job-name=<job-name> -n lab06

# Manually trigger the CronJob immediately
kubectl create job manual-trigger --from=cronjob/hello-cron -n lab06
```

---

## 🧹 Cleanup
```bash
kubectl delete namespace lab06
```

## 📖 Solutions
See `solutions/` folder.
