
# **Kubernetes `kubectl` Cheat Sheet**

## **1. Basic Cluster and Node Operations**

### **View Cluster Information**
```bash
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>
kubectl get nodes -o wide
```

### **Check Node Conditions (Resource Pressure)**
```bash
kubectl describe node <node-name> | grep -i pressure
```
- **MemoryPressure**: High memory usage
- **DiskPressure**: Low disk space
- **PIDPressure**: Too many processes running

### **Cordon & Uncordon Nodes**
```bash
kubectl cordon <node-name>         # Mark node as unschedulable
kubectl uncordon <node-name>       # Allow scheduling on the node again
```

### **Drain a Node (Evict Pods Before Maintenance)**
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

---

## **2. Pod Management & Evictions**

### **View Running Pods**
```bash
kubectl get pods -A              # List all pods in all namespaces
kubectl get pods -n <namespace>  # List pods in a specific namespace
kubectl get pods -o wide         # Show detailed pod information
kubectl describe pod <pod-name>  # Show pod details, including events
```

### **Check Pod Events & Eviction History**
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod-name> | grep -i eviction
```

### **Manually Evict a Pod**
```bash
kubectl delete pod <pod-name>
```

### **Check QoS of a Pod**
```bash
kubectl get pod <pod-name> -o jsonpath="{.status.qosClass}"
```
- **Guaranteed**: Requests = Limits
- **Burstable**: Requests < Limits
- **BestEffort**: No requests or limits set

---

## **3. Scheduling: Taints, Tolerations, and Affinity**

### **View Taints on a Node**
```bash
kubectl describe node <node-name> | grep -i taint
```

### **Add a Taint to a Node**
```bash
kubectl taint nodes <node-name> key=value:NoSchedule
```

### **Remove a Taint from a Node**
```bash
kubectl taint nodes <node-name> key=value:NoSchedule-
```

### **Deploy a Pod with a Toleration**
```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
```

### **Deploy a Pod with Node Affinity**
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: "region"
          operator: "In"
          values:
          - "us-east"
```

### **Deploy a Pod with Pod Anti-Affinity**
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: "tier"
          operator: "In"
          values:
          - "db"
      topologyKey: "kubernetes.io/hostname"
```

---

## **4. Helm Commands**

### **Add a Helm Repository**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### **Install a Helm Chart**
```bash
helm install my-nginx bitnami/nginx
```

### **List Installed Helm Releases**
```bash
helm list
```

### **Upgrade a Helm Release**
```bash
helm upgrade my-nginx bitnami/nginx -f values.yaml
```

### **Rollback a Helm Release**
```bash
helm rollback my-nginx 1
```

### **Uninstall a Helm Release**
```bash
helm uninstall my-nginx
```

---

## **5. Resource Monitoring**

### **Check Node Resource Usage**
```bash
kubectl top nodes
```

### **Check Pod Resource Usage**
```bash
kubectl top pods --containers
```

### **Describe Resource Requests & Limits of a Pod**
```bash
kubectl describe pod <pod-name> | grep -i "requests\|limits"
```

---

## **6. Debugging & Logs**

### **View Pod Logs**
```bash
kubectl logs <pod-name>
kubectl logs <pod-name> -f         # Follow logs in real-time
kubectl logs <pod-name> -c <container-name> # Logs for a specific container
```

### **Execute a Command in a Running Pod**
```bash
kubectl exec -it <pod-name> -- /bin/sh
```

### **Ping from One Pod to Another**
```bash
kubectl exec -it <pod-name> -- ping <target-pod-name>
```

### **Get Detailed YAML Output for Any Resource**
```bash
kubectl get pod <pod-name> -o yaml
kubectl get node <node-name> -o yaml
```

---

## **7. Namespace Management**

### **View Namespaces**
```bash
kubectl get namespaces
```

### **Create a New Namespace**
```bash
kubectl create namespace my-namespace
```

### **Delete a Namespace**
```bash
kubectl delete namespace my-namespace
```

---

## **8. Cleanup & Deleting Resources**

### **Delete All Pods in a Namespace**
```bash
kubectl delete pods --all -n <namespace>
```

### **Delete a Specific Deployment**
```bash
kubectl delete deployment <deployment-name>
```

### **Delete a Specific Service**
```bash
kubectl delete service <service-name>
```

### **Delete a Persistent Volume Claim (PVC)**
```bash
kubectl delete pvc <pvc-name>
```

---

## **9. Useful One-Liners**

### **Get All Running Pods with Labels**
```bash
kubectl get pods --show-labels
```

### **Check Where Each Pod is Running**
```bash
kubectl get pods -o wide
```

### **Find All Pods in Pending State**
```bash
kubectl get pods --field-selector=status.phase=Pending
```

### **Get Nodes & Their Labels**
```bash
kubectl get nodes --show-labels
```

