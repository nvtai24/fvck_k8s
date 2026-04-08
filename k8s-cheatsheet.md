# ☸️ Kubernetes (kubectl) Cheat Sheet

---

## 🔧 Cluster & Context

```bash
kubectl version                          # Show client/server version
kubectl cluster-info                     # Display cluster info
kubectl config get-contexts              # List all contexts
kubectl config current-context           # Show active context
kubectl config use-context <name>        # Switch context
kubectl config set-context --current --namespace=<ns>  # Set default namespace
```

---

## 📦 Namespaces

```bash
kubectl get namespaces                   # List all namespaces
kubectl create namespace <name>          # Create a namespace
kubectl delete namespace <name>          # Delete a namespace
```

---

## 🚀 Pods

```bash
kubectl get pods                         # List pods in current namespace
kubectl get pods -n <namespace>          # List pods in specific namespace
kubectl get pods -A                      # List pods across all namespaces
kubectl get pods -o wide                 # List pods with node info
kubectl describe pod <pod-name>          # Detailed pod info
kubectl logs <pod-name>                  # Fetch pod logs
kubectl logs <pod-name> -c <container>   # Logs from specific container
kubectl logs -f <pod-name>               # Stream (follow) logs
kubectl exec -it <pod-name> -- /bin/bash # Exec into pod shell
kubectl exec -it <pod-name> -c <container> -- sh  # Exec into specific container
kubectl delete pod <pod-name>            # Delete a pod
kubectl run <name> --image=<image>       # Run a pod from image
```

---

## 📄 Deployments

```bash
kubectl get deployments                              # List deployments
kubectl describe deployment <name>                   # Deployment details
kubectl create deployment <name> --image=<image>     # Create deployment
kubectl apply -f <file.yaml>                         # Apply config from file
kubectl delete deployment <name>                     # Delete deployment
kubectl scale deployment <name> --replicas=<n>       # Scale deployment
kubectl rollout status deployment/<name>             # Check rollout status
kubectl rollout history deployment/<name>            # Show rollout history
kubectl rollout undo deployment/<name>               # Rollback deployment
kubectl set image deployment/<name> <container>=<new-image>  # Update image
```

---

## 🌐 Services

```bash
kubectl get services                     # List all services
kubectl get svc                          # Shorthand
kubectl describe service <name>          # Service details
kubectl expose deployment <name> --port=<port> --type=NodePort  # Expose deployment
kubectl delete service <name>            # Delete service
kubectl port-forward <pod-name> <local>:<remote>     # Port-forward to pod
kubectl port-forward svc/<svc-name> <local>:<remote> # Port-forward to service
```

---

## ⚙️ ConfigMaps & Secrets

```bash
# ConfigMaps
kubectl get configmaps                              # List configmaps
kubectl create configmap <name> --from-literal=key=val  # Create from literal
kubectl create configmap <name> --from-file=<file>  # Create from file
kubectl describe configmap <name>                   # ConfigMap details
kubectl delete configmap <name>                     # Delete configmap

# Secrets
kubectl get secrets                                 # List secrets
kubectl create secret generic <name> --from-literal=key=val  # Create secret
kubectl describe secret <name>                      # Secret details
kubectl get secret <name> -o jsonpath='{.data.<key>}'  # Get secret value (base64)
```

---

## 📊 Nodes

```bash
kubectl get nodes                        # List all nodes
kubectl get nodes -o wide                # Nodes with IP/OS info
kubectl describe node <name>             # Node details
kubectl cordon <node>                    # Mark node as unschedulable
kubectl uncordon <node>                  # Mark node as schedulable
kubectl drain <node> --ignore-daemonsets # Drain node (for maintenance)
kubectl top nodes                        # Show node resource usage
kubectl top pods                         # Show pod resource usage
```

---

## 📁 Apply / Create / Delete (Declarative)

```bash
kubectl apply -f <file.yaml>             # Apply a resource file
kubectl apply -f <directory>/            # Apply all yamls in folder
kubectl create -f <file.yaml>            # Create resource from file
kubectl delete -f <file.yaml>            # Delete resource from file
kubectl diff -f <file.yaml>              # Diff current vs file state
```

---

## 🔍 Inspecting & Debugging

```bash
kubectl get all                          # Get all resources in namespace
kubectl get all -A                       # Get all resources in all namespaces
kubectl get events                       # View cluster events
kubectl get events --sort-by='.lastTimestamp'  # Events sorted by time
kubectl describe <resource> <name>       # Describe any resource
kubectl get <resource> -o yaml           # Output resource as YAML
kubectl get <resource> -o json           # Output resource as JSON
kubectl explain <resource>               # Show resource field docs
kubectl explain <resource>.spec          # Explain specific fields
```

---

## 🏷️ Labels & Selectors

```bash
kubectl get pods -l app=<label>                  # Filter by label
kubectl label pod <name> env=prod                # Add label to pod
kubectl label pod <name> env-                    # Remove label from pod
kubectl get pods --selector="app=myapp"          # Select by label
```

---

## 🔄 ReplicaSets & StatefulSets

```bash
kubectl get replicasets                  # List ReplicaSets
kubectl describe rs <name>               # ReplicaSet details
kubectl get statefulsets                 # List StatefulSets
kubectl describe sts <name>              # StatefulSet details
kubectl scale sts <name> --replicas=<n>  # Scale StatefulSet
```

---

## 📦 Persistent Volumes

```bash
kubectl get pv                           # List PersistentVolumes
kubectl get pvc                          # List PersistentVolumeClaims
kubectl describe pv <name>               # PV details
kubectl describe pvc <name>              # PVC details
kubectl delete pvc <name>                # Delete PVC
```

---

## 🛡️ RBAC

```bash
kubectl get roles                        # List Roles (namespace-scoped)
kubectl get clusterroles                 # List ClusterRoles
kubectl get rolebindings                 # List RoleBindings
kubectl get clusterrolebindings          # List ClusterRoleBindings
kubectl create role <name> --verb=get,list --resource=pods  # Create role
kubectl create rolebinding <name> --role=<role> --user=<user>  # Bind role
kubectl auth can-i <verb> <resource>     # Check permissions (current user)
kubectl auth can-i <verb> <resource> --as=<user>  # Check as other user
```

---

## 🧩 Ingress

```bash
kubectl get ingress                      # List ingresses
kubectl describe ingress <name>          # Ingress details
kubectl delete ingress <name>            # Delete ingress
```

---

## 🗂️ Useful Shortcuts

| Shorthand | Full Resource          |
| --------- | ---------------------- |
| `po`      | pods                   |
| `svc`     | services               |
| `deploy`  | deployments            |
| `ds`      | daemonsets             |
| `sts`     | statefulsets           |
| `rs`      | replicasets            |
| `cm`      | configmaps             |
| `ns`      | namespaces             |
| `no`      | nodes                  |
| `pv`      | persistentvolumes      |
| `pvc`     | persistentvolumeclaims |
| `ing`     | ingresses              |
| `sa`      | serviceaccounts        |

---

## 💡 Tips & Tricks

```bash
# Dry-run (don't apply, just preview)
kubectl apply -f file.yaml --dry-run=client

# Generate YAML from a command
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Watch resources live
kubectl get pods -w

# Delete all pods matching a label
kubectl delete pods -l app=myapp

# Force delete a stuck pod
kubectl delete pod <name> --grace-period=0 --force

# Copy files from/to a pod
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file

# Get resource in multiple namespaces
kubectl get pods -n ns1 && kubectl get pods -n ns2
```

---

> ☸️ **Cheat sheet generated for Kubernetes exam preparation — Good luck!**
