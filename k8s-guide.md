# ☸️ Kubernetes — Comprehensive Guide

> **English Only** | Version: K8s v1.28+ | Last updated: 2026

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Pods](#2-pods)
3. [Workloads](#3-workloads)
   - ReplicaSet, Deployment, StatefulSet, DaemonSet, Job, CronJob
4. [Services & Networking](#4-services--networking)
   - ClusterIP, NodePort, LoadBalancer, ExternalName, Ingress, NetworkPolicy
5. [Configuration](#5-configuration)
   - ConfigMap, Secret
6. [Storage](#6-storage)
   - PersistentVolume, PersistentVolumeClaim, StorageClass, Volumes
7. [RBAC & Security](#7-rbac--security)
   - ServiceAccount, Role, ClusterRole, RoleBinding, ClusterRoleBinding
8. [Scheduling & Placement](#8-scheduling--placement)
   - NodeSelector, Affinity, Taints & Tolerations
9. [Probes & Health Checks](#9-probes--health-checks)
10. [Autoscaling](#10-autoscaling)
    - HPA, VPA, KEDA
11. [Resource Management](#11-resource-management)
    - ResourceQuota, LimitRange, Requests & Limits
12. [Policy & Reliability](#12-policy--reliability)
    - PodDisruptionBudget, PriorityClass
13. [Labels, Selectors & Annotations](#13-labels-selectors--annotations)
14. [Helm Basics](#14-helm-basics)
15. [Useful Patterns](#15-useful-patterns)

---

## 1. Architecture Overview

### Control Plane Components

| Component | Role |
|---|---|
| **kube-apiserver** | Single entry point for all API calls; validates and processes REST requests |
| **etcd** | Distributed key-value store — stores all cluster state/config |
| **kube-scheduler** | Assigns pods to nodes based on resource requirements and constraints |
| **kube-controller-manager** | Runs controllers (Node, Replication, Endpoint, etc.) |
| **cloud-controller-manager** | Integrates with cloud providers (AWS, GCP, Azure) |

### Worker Node Components

| Component | Role |
|---|---|
| **kubelet** | Agent on each node; ensures containers run as specified in PodSpecs |
| **kube-proxy** | Maintains network rules on nodes; handles Service networking (iptables/ipvs) |
| **Container Runtime** | Runs containers (containerd, CRI-O, Docker) |

### High-Level Flow

```
User → kubectl → kube-apiserver → etcd (persist)
                              ↓
                   kube-scheduler (assign node)
                              ↓
                     kubelet (run container)
```

---

## 2. Pods

A **Pod** is the smallest deployable unit in Kubernetes. It wraps one or more containers that share:
- Network namespace (same IP, same port space)
- Storage volumes
- Lifecycle

### Pod Lifecycle

```
Pending → Running → Succeeded/Failed
                 ↓
             CrashLoopBackOff (if container keeps failing)
```

### Pod Phases

| Phase | Description |
|---|---|
| `Pending` | Pod accepted by cluster, containers not yet started |
| `Running` | At least one container is running |
| `Succeeded` | All containers exited with status 0 |
| `Failed` | At least one container exited with non-zero status |
| `Unknown` | Pod state can't be determined (node issue) |

### Container Types Inside a Pod

| Type | Description |
|---|---|
| **Main container** | Primary app container |
| **Init container** | Runs to completion before main containers start |
| **Sidecar container** | Helper container (logging, proxy, etc.) |
| **Ephemeral container** | Temporary debug container injected into running pod |

### Manifest Reference → `manifests/01-pods/`

---

## 3. Workloads

### 3.1 ReplicaSet

Ensures a specified number of pod replicas are running at all times. Rarely used directly — use Deployments instead.

**Key fields:** `replicas`, `selector`, `template`

### 3.2 Deployment

Wraps a ReplicaSet and adds:
- **Rolling updates** (zero-downtime)
- **Rollback** capability
- **Revision history**

**Update strategies:**
- `RollingUpdate` (default) — gradually replaces old pods
- `Recreate` — kill all old pods, then start new ones

**Key fields:** `replicas`, `strategy`, `selector`, `template`

### 3.3 StatefulSet

Like a Deployment, but for **stateful apps** (databases, queues). Provides:
- **Stable, unique network identity** (`pod-0`, `pod-1`, ...)
- **Stable, persistent storage** per pod
- **Ordered deployment** and scaling (0 → 1 → 2)
- **Ordered rolling updates** (N-1 → N-2 → ...)

### 3.4 DaemonSet

Ensures a pod runs on **every node** (or a subset with nodeSelectors). Used for:
- Log collectors (Fluentd, Filebeat)
- Node monitoring (Prometheus node exporter)
- Network plugins (Calico, Weave)

### 3.5 Job

Runs a pod to **completion** (not continuously). Useful for batch tasks.

**Key fields:** `completions`, `parallelism`, `backoffLimit`

### 3.6 CronJob

Runs **Jobs on a schedule** (cron syntax). Creates a new Job for each scheduled run.

**Key fields:** `schedule`, `jobTemplate`, `successfulJobsHistoryLimit`

### Manifest Reference → `manifests/02-workloads/`

---

## 4. Services & Networking

### 4.1 Service Types

| Type | Scope | Use Case |
|---|---|---|
| `ClusterIP` (default) | Internal only | Pod-to-pod communication |
| `NodePort` | External via node IP | Dev/testing, basic external access |
| `LoadBalancer` | External via cloud LB | Production external access |
| `ExternalName` | DNS alias | Redirect to external DNS name |

### 4.2 How Services Work

Services use **label selectors** to route traffic to matching pods. `kube-proxy` updates iptables/ipvs rules on each node to forward traffic.

```
Client → Service (ClusterIP:Port) → kube-proxy → Pod (PodIP:TargetPort)
```

### 4.3 Ingress

An **Ingress** is an API object that manages external HTTP/HTTPS routing to internal services. Requires an **Ingress Controller** (nginx, traefik, etc.).

Features:
- Host-based routing (`api.example.com` → service A)
- Path-based routing (`/api` → service A, `/web` → service B)
- TLS termination

### 4.4 NetworkPolicy

Controls **which pods can communicate** with each other (acts like a firewall). By default, all pods can talk to all pods.

**Policy types:** `Ingress` (incoming), `Egress` (outgoing)

Requires a CNI plugin that supports NetworkPolicy (Calico, Cilium, Weave Net).

### Manifest Reference → `manifests/03-services/` & `manifests/07-networking/`

---

## 5. Configuration

### 5.1 ConfigMap

Stores **non-sensitive** key-value configuration data. Can be consumed as:
- **Environment variables**
- **Command-line arguments**
- **Mounted files (volumes)**

### 5.2 Secret

Stores **sensitive** data (passwords, tokens, keys). Encoded in **base64** (not encrypted by default — enable encryption at rest for production).

**Secret types:**
| Type | Use |
|---|---|
| `Opaque` | Arbitrary user-defined data |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/tls` | TLS certificate and key |
| `kubernetes.io/service-account-token` | ServiceAccount token |

### Manifest Reference → `manifests/04-config/`

---

## 6. Storage

### 6.1 Volume Types

| Volume | Lifecycle | Notes |
|---|---|---|
| `emptyDir` | Pod lifetime | Ephemeral; shared between containers in same pod |
| `hostPath` | Node lifetime | Mounts node filesystem path (use with caution) |
| `configMap` / `secret` | External | Inject config/secrets as files |
| `persistentVolumeClaim` | Independent | Backed by PV; survives pod restarts |
| `nfs` | External | Network File System mount |

### 6.2 PersistentVolume (PV)

A **PV** is a piece of storage provisioned by an admin (or dynamically via StorageClass). It's a cluster-level resource (not namespace-scoped).

### 6.3 PersistentVolumeClaim (PVC)

A **PVC** is a request for storage by a user. Kubernetes binds a PVC to a matching PV.

**Binding process:**
```
PVC (request) → matches → PV (provision) → bound
```

### 6.4 StorageClass

Defines **how** storage is provisioned dynamically. Allows on-demand PV creation.

**Access Modes:**
| Mode | Abbreviation | Description |
|---|---|---|
| `ReadWriteOnce` | RWO | One node read/write |
| `ReadOnlyMany` | ROX | Many nodes read-only |
| `ReadWriteMany` | RWX | Many nodes read/write |

### 6.5 Reclaim Policies

| Policy | Behavior when PVC deleted |
|---|---|
| `Retain` | PV kept; manual cleanup required |
| `Delete` | PV and underlying storage deleted |
| `Recycle` | (Deprecated) Scrubs data and makes PV available again |

### Manifest Reference → `manifests/05-storage/`

---

## 7. RBAC & Security

### 7.1 Core RBAC Objects

| Object | Scope | Description |
|---|---|---|
| `ServiceAccount` | Namespace | Identity for pods to authenticate to API |
| `Role` | Namespace | Grants permissions within a namespace |
| `ClusterRole` | Cluster-wide | Grants permissions across the cluster |
| `RoleBinding` | Namespace | Binds a Role/ClusterRole to a user/SA in a namespace |
| `ClusterRoleBinding` | Cluster-wide | Binds a ClusterRole to a user/SA globally |

### 7.2 RBAC Flow

```
Pod (uses ServiceAccount) → has RoleBinding → references Role → allows verbs on resources
```

### 7.3 Common Verbs

`get`, `list`, `watch`, `create`, `update`, `patch`, `delete`, `deletecollection`

### 7.4 Pod Security

**Security Context** — applied at pod or container level:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

### Manifest Reference → `manifests/06-rbac/`

---

## 8. Scheduling & Placement

### 8.1 NodeSelector

Simplest form of node selection — matches node labels.

```yaml
nodeSelector:
  disktype: ssd
```

### 8.2 Node Affinity

More expressive than nodeSelector. Two types:
- `requiredDuringSchedulingIgnoredDuringExecution` — hard rule (must match)
- `preferredDuringSchedulingIgnoredDuringExecution` — soft rule (prefer but not required)

### 8.3 Pod Affinity & Anti-Affinity

Schedule pods **near** or **away from** other pods based on labels.

- **Pod Affinity** — co-locate pods (same node/zone)
- **Pod Anti-Affinity** — spread pods across nodes/zones

### 8.4 Taints & Tolerations

**Taints** applied to **nodes** repel pods. **Tolerations** applied to **pods** allow them to be scheduled on tainted nodes.

```
Node: taint key=value:Effect
Pod:  toleration key=value Effect
```

**Taint effects:**
| Effect | Description |
|---|---|
| `NoSchedule` | New pods without toleration won't be scheduled |
| `PreferNoSchedule` | Prefer not to schedule; not a hard rule |
| `NoExecute` | Evicts existing pods without toleration |

---

## 9. Probes & Health Checks

### Probe Types

| Probe | When | Purpose |
|---|---|---|
| `livenessProbe` | Continuously | Is the container still running? Restart if failed |
| `readinessProbe` | Continuously | Is the container ready to serve traffic? Remove from Service endpoints if failed |
| `startupProbe` | At startup only | Gives slow-starting containers time before liveness kicks in |

### Probe Mechanisms

```yaml
# HTTP GET
httpGet:
  path: /healthz
  port: 8080

# TCP Socket
tcpSocket:
  port: 8080

# Exec command
exec:
  command: ["cat", "/tmp/healthy"]
```

### Key Fields

```yaml
initialDelaySeconds: 5   # Wait before first probe
periodSeconds: 10        # How often to probe
timeoutSeconds: 2        # Timeout per probe
failureThreshold: 3      # Failures before action
successThreshold: 1      # Successes to mark healthy
```

---

## 10. Autoscaling

### 10.1 HorizontalPodAutoscaler (HPA)

Automatically scales the number of pod replicas based on observed metrics (CPU, memory, custom metrics).

```
HPA → watches metrics → adjusts Deployment replicas
```

**Requires:** Metrics Server installed in cluster.

### 10.2 VerticalPodAutoscaler (VPA)

Automatically adjusts CPU/memory **requests and limits** for individual containers.

**Modes:** `Off` (recommend only), `Initial` (set at creation), `Auto` (update in place)

### 10.3 Cluster Autoscaler

Automatically adjusts **number of nodes** in the cluster based on pending pods and node utilization. Cloud-provider specific.

### Manifest Reference → `manifests/08-autoscaling/`

---

## 11. Resource Management

### 11.1 Requests & Limits

```yaml
resources:
  requests:
    cpu: "250m"       # Minimum guaranteed CPU (250 millicores = 0.25 CPU)
    memory: "128Mi"   # Minimum guaranteed memory
  limits:
    cpu: "500m"       # Maximum CPU allowed
    memory: "256Mi"   # Maximum memory; OOMKilled if exceeded
```

**QoS Classes:**
| Class | Condition |
|---|---|
| `Guaranteed` | Requests == Limits for all containers |
| `Burstable` | At least one container has requests < limits |
| `BestEffort` | No requests or limits set |

### 11.2 ResourceQuota

Limits total resource consumption in a namespace.

```yaml
# Can limit: pods, services, CPU, memory, PVCs, secrets, configmaps, etc.
```

### 11.3 LimitRange

Sets **default** requests/limits and enforces min/max per container, pod, or PVC in a namespace.

### Manifest Reference → `manifests/09-policy/`

---

## 12. Policy & Reliability

### 12.1 PodDisruptionBudget (PDB)

Limits voluntary disruptions (drains, updates) to ensure minimum pod availability.

```yaml
minAvailable: 2    # At least 2 pods must be available
# OR
maxUnavailable: 1  # At most 1 pod can be unavailable
```

### 12.2 PriorityClass

Assigns scheduling priority to pods. Higher-priority pods preempt lower-priority ones when resources are scarce.

---

## 13. Labels, Selectors & Annotations

### Labels

Key-value pairs attached to objects. Used for **identification and selection**.

```yaml
labels:
  app: nginx
  env: production
  version: "1.0"
```

### Selectors

Filter resources by labels. Two types:
- **Equality-based:** `app=nginx`, `env!=dev`
- **Set-based:** `app in (nginx, apache)`, `env notin (dev)`

### Annotations

Key-value pairs for **non-identifying metadata** (tool info, build info, documentation links). Not used for selection.

```yaml
annotations:
  description: "Main web server deployment"
  git-commit: "abc1234"
  prometheus.io/scrape: "true"
```

---

## 14. Helm Basics

**Helm** is the package manager for Kubernetes. Packages are called **Charts**.

```bash
# Repo management
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx

# Install / Upgrade
helm install my-release bitnami/nginx
helm upgrade my-release bitnami/nginx --set replicas=3
helm install my-release ./my-chart -f values.yaml

# Manage releases
helm list
helm status my-release
helm rollback my-release 1
helm uninstall my-release

# Inspect
helm show values bitnami/nginx > values.yaml
helm template my-release bitnami/nginx    # Render templates without installing

# Package
helm package ./my-chart
```

### Chart Structure

```
my-chart/
├── Chart.yaml          # Metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl    # Template helpers
└── charts/             # Dependency charts
```

---

## 15. Useful Patterns

### 15.1 Sidecar Pattern

Attach a helper container alongside the main container to extend functionality without modifying the main app (e.g., log shipper, reverse proxy, file sync).

### 15.2 Init Container Pattern

Use init containers to run setup tasks before the main app starts:
- Wait for a dependency to be ready
- Pre-populate a shared volume
- Run database migrations

### 15.3 Ambassador Pattern

Proxy container that handles external communications on behalf of the main container (e.g., Envoy sidecar).

### 15.4 Blue-Green Deployment

Run two identical environments (blue=current, green=new). Switch traffic by updating the Service selector.

```bash
kubectl patch service my-svc -p '{"spec":{"selector":{"version":"green"}}}'
```

### 15.5 Canary Deployment

Route a small percentage of traffic to a new version by running a small number of new pods alongside old ones. Label-based selector balances traffic proportionally.

### 15.6 ConfigMap Hot Reload

Mount ConfigMap as a volume — when ConfigMap is updated, the mounted files update automatically (may need app to detect changes). Environment variables from ConfigMap do NOT hot-reload (require pod restart).

### 15.7 Graceful Shutdown

```yaml
terminationGracePeriodSeconds: 30   # Time for pod to shut down gracefully
lifecycle:
  preStop:
    exec:
      command: ["sleep", "5"]       # Wait before SIGTERM (drain connections)
```

---

## 📁 Manifests Directory Structure

```
manifests/
├── 01-pods/
│   ├── basic-pod.yaml
│   ├── multi-container-pod.yaml
│   └── init-container-pod.yaml
├── 02-workloads/
│   ├── replicaset.yaml
│   ├── deployment.yaml
│   ├── statefulset.yaml
│   ├── daemonset.yaml
│   ├── job.yaml
│   └── cronjob.yaml
├── 03-services/
│   ├── clusterip-service.yaml
│   ├── nodeport-service.yaml
│   ├── loadbalancer-service.yaml
│   └── ingress.yaml
├── 04-config/
│   ├── configmap.yaml
│   └── secret.yaml
├── 05-storage/
│   ├── persistentvolume.yaml
│   ├── persistentvolumeclaim.yaml
│   └── storageclass.yaml
├── 06-rbac/
│   ├── serviceaccount.yaml
│   ├── role.yaml
│   ├── rolebinding.yaml
│   ├── clusterrole.yaml
│   └── clusterrolebinding.yaml
├── 07-networking/
│   └── networkpolicy.yaml
├── 08-autoscaling/
│   └── hpa.yaml
└── 09-policy/
    ├── resourcequota.yaml
    ├── limitrange.yaml
    └── poddisruptionbudget.yaml
```

---

> 📚 **See individual manifest files in `manifests/` for working YAML examples of every concept above.**
