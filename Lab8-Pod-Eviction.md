# **Kubernetes Pod Eviction Lab**  


---

## **Objectives**
1. **Simulate pod eviction due to memory pressure**  
2. **Simulate pod eviction due to ephemeral storage pressure**  
3. **Evict pods manually by draining a node**  
4. **Protect critical pods using Pod Priority during eviction**  
5. **Observe eviction events and troubleshoot using `kubectl`**

---

## **Prerequisites**
- A Kubernetes cluster (Minikube, kind, or cloud-based)
- `kubectl` CLI configured
- `stress` image for simulating resource pressure (`polinux/stress` or `progrium/stress`)
- A test namespace (e.g., `kubectl create namespace eviction-lab`)

---

## **Task 1: Simulate Pod Eviction Due to Memory Pressure**  

### **Step 1: Deploy a High Memory Usage Pod**
Create a YAML file named **`memory-stress.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-stress
  namespace: eviction-lab
spec:
  containers:
  - name: memory-stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "2", "--vm-bytes", "1G", "--timeout", "120s"]
    resources:
      requests:
        memory: "500Mi"
      limits:
        memory: "1Gi"
```

### **Step 2: Deploy a Low Priority Pod**
Create a YAML file named **`low-priority-pod.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-priority
  namespace: eviction-lab
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 600"]
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
```

### **Step 3: Apply and Observe Eviction**
Deploy both pods:

```bash
kubectl apply -f memory-stress.yaml
kubectl apply -f low-priority-pod.yaml
```

Monitor pod status:

```bash
kubectl get pods -n eviction-lab -w
```

If the node has limited memory, Kubernetes will evict the **low-priority** pod first.

Check eviction reason:

```bash
kubectl describe pod low-priority -n eviction-lab
```

---

## **Task 2: Simulate Pod Eviction Due to Ephemeral Storage Pressure**

### **Step 1: Deploy a Pod That Uses Ephemeral Storage**
Create a YAML file named **`storage-stress.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-stress
  namespace: eviction-lab
spec:
  containers:
  - name: storage-stress
    image: busybox
    command: ["sh", "-c", "dd if=/dev/zero of=/data/bigfile bs=1M count=2048; sleep 300"]
    volumeMounts:
    - name: ephemeral-storage
      mountPath: /data
    resources:
      requests:
        ephemeral-storage: "200Mi"
      limits:
        ephemeral-storage: "500Mi"
  volumes:
  - name: ephemeral-storage
    emptyDir: {}
```

### **Step 2: Apply and Observe Eviction**
Deploy the pod:

```bash
kubectl apply -f storage-stress.yaml
```

Monitor eviction:

```bash
kubectl get pods -n eviction-lab -w
```

Check node storage pressure:

```bash
kubectl describe node <node-name>
```

You should see a **DiskPressure** condition if the storage fills up.

---

## **Task 3: Evict Pods by Draining a Node**
### **Step 1: Identify a Node**
List available nodes:

```bash
kubectl get nodes
```

### **Step 2: Cordon the Node**
This prevents new pods from being scheduled:

```bash
kubectl cordon <node-name>
```

### **Step 3: Drain the Node**
This will evict all non-DaemonSet pods:

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

Monitor eviction:

```bash
kubectl get pods -o wide -n eviction-lab
```

Once testing is complete, uncordon the node:

```bash
kubectl uncordon <node-name>
```

---

## **Task 4: Protect Critical Pods Using Pod Priority**

### **Step 1: Create a High-Priority Class**
Create a YAML file named **`high-priority.yaml`**:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "Priority class for critical pods."
```

Apply the priority class:

```bash
kubectl apply -f high-priority.yaml
```

### **Step 2: Deploy a Critical Pod**
Create a YAML file named **`critical-pod.yaml`**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
  namespace: eviction-lab
spec:
  priorityClassName: high-priority
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "500Mi"
      requests:
        memory: "200Mi"
```

### **Step 3: Apply and Observe**
Apply the manifest:

```bash
kubectl apply -f critical-pod.yaml
```

Re-run the **memory-stress.yaml** pod from **Task 1** and check which pod gets evicted first:

```bash
kubectl apply -f memory-stress.yaml
```

The **low-priority** pod should be evicted, while the **critical pod** remains running.

---

## **Observations & Troubleshooting**
- Check events related to eviction:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp -n eviction-lab
```

- Check node conditions:

```bash
kubectl describe node <node-name>
```

- Describe a specific pod for eviction details:

```bash
kubectl describe pod <pod-name> -n eviction-lab
```

---

## **Cleanup**
After completing the lab, clean up the created resources:

```bash
kubectl delete namespace eviction-lab
kubectl delete priorityclass high-priority
kubectl uncordon <node-name> # If previously drained
```

---

## **Conclusion**
This lab demonstrated different reasons why Kubernetes evicts pods, including:
✅ **Memory pressure** leading to eviction of low-priority pods  
✅ **Ephemeral storage pressure** causing evictions when disk space is low  
✅ **Node drain operations** forcing pod eviction  
✅ **Pod Priority** ensuring critical workloads remain unaffected during resource pressure  

