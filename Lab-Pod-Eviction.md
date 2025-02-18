To help students practice the concept of **Pod Priority** and **PriorityClasses** in Kubernetes, we can create a **hands-on lab** where students will create and manage **PriorityClass** objects and assign them to pods. This lab will demonstrate how pod priority affects scheduling and eviction behavior in resource-constrained environments.

Here’s a step-by-step guide for the lab:

---

### **Lab: Practice with Kubernetes Pod Priority and PriorityClasses**

#### **Prerequisites:**
- A working Kubernetes cluster 
- `kubectl` CLI tool installed and configured to interact with your cluster.

---

### **Lab Setup Steps:**

#### **1. Create Priority Classes**

In this step, students will create different **PriorityClass** resources to assign priorities to pods.

**Task:**
- Create three different PriorityClasses with varying priority values:
  - **high-priority**: A high priority value for critical workloads.
  - **medium-priority**: A moderate priority value.
  - **low-priority**: A low priority value for non-essential workloads.

**Instructions:**
1. Create a YAML file `priority-classes.yaml`:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority pods for critical workloads"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: medium-priority
value: 500000
globalDefault: false
description: "Medium priority pods"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100000
globalDefault: false
description: "Low priority pods"
```

2. Apply the PriorityClass resources to the cluster:

```bash
kubectl apply -f priority-classes.yaml
```

3. Verify the PriorityClasses have been created:

```bash
kubectl get priorityclass
```

This should list the three PriorityClasses (`high-priority`, `medium-priority`, `low-priority`).

---

#### **2. Create Pods with Different Priority Classes**

Now, students will create pods that use the **PriorityClass** resources they just created.

**Task:**
- Create three pods:
  - A **high-priority** pod.
  - A **medium-priority** pod.
  - A **low-priority** pod.

**Instructions:**
1. Create a YAML file `pods-with-priority.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

---
apiVersion: v1
kind: Pod
metadata:
  name: medium-priority-pod
spec:
  priorityClassName: medium-priority
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

---
apiVersion: v1
kind: Pod
metadata:
  name: low-priority-pod
spec:
  priorityClassName: low-priority
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

2. Apply the pod configuration:

```bash
kubectl apply -f pods-with-priority.yaml
```

3. Verify that the pods have been created:

```bash
kubectl get pods
```

The output will show that the pods `high-priority-pod`, `medium-priority-pod`, and `low-priority-pod` are running.

---

#### **3. Simulate Resource Pressure (Optional for Eviction)**

Next, students will simulate resource pressure on the cluster by running resource-intensive pods and observing how Kubernetes evicts lower-priority pods.

**Task:**
- Simulate resource pressure by creating **BestEffort** pods that consume a lot of resources and trigger the eviction process.

**Instructions:**
1. Create a YAML file `best-effort-pods.yaml` for resource-hogging pods:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-hogger-pod-1
spec:
  containers:
  - name: stress-test
    image: polinux/stress
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"

---
apiVersion: v1
kind: Pod
metadata:
  name: resource-hogger-pod-2
spec:
  containers:
  - name: stress-test
    image: polinux/stress
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"
```

2. Apply the resource-hogging pods to the cluster:

```bash
kubectl apply -f best-effort-pods.yaml
```

3. Monitor the pod eviction process by checking the pod status:

```bash
kubectl get pods
```

- Pods with lower priority, like the `low-priority-pod`, may be evicted first to make room for higher-priority pods when the node reaches its resource limits.

4. **Verify Pod Evictions**:
   - Use `kubectl describe pod <pod-name>` to check the reason for eviction, e.g.,:
   ```bash
   kubectl describe pod low-priority-pod
   ```

   The **Event** section will show eviction details like:
   ```
   Evicted: The node was under pressure, and pod was evicted
   ```

---

#### **4. Clean Up**

Once the practice is complete, students should clean up the resources they’ve created to keep the cluster clean.

**Task:**
- Delete the created pods and PriorityClasses.

**Instructions:**
1. Delete the created pods:

```bash
kubectl delete pod high-priority-pod medium-priority-pod low-priority-pod
```

2. Delete the resource-hogging pods:

```bash
kubectl delete pod resource-hogger-pod-1 resource-hogger-pod-2
```

3. Delete the PriorityClasses:

```bash
kubectl delete priorityclass high-priority medium-priority low-priority
```

---

### **Conclusion:**
In this lab, students practiced:
- Creating and managing **PriorityClass** resources.
- Assigning different **PriorityClasses** to pods.
- Simulating **node resource pressure** to observe how Kubernetes handles pod evictions.
- Observing how **higher-priority pods** are less likely to be evicted than **lower-priority pods** under pressure.
