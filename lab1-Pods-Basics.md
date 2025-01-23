### **Lab: Hands-On with Kubernetes Pods**

This lab is designed to help students experiment with Kubernetes Pods, understand their behavior, and learn how to manage them.

---

### **Objective**
By the end of this lab, students will:
1. Create and manage Kubernetes Pods.
2. Understand Pod specifications.
3. Experiment with multi-container Pods.
4. Observe and debug Pod behavior using Kubernetes commands.

---

### **Prerequisites**
1. A Kubernetes cluster (e.g., Minikube, Kind, or a cloud-based cluster).
2. `kubectl` command-line tool installed and configured.
3. Basic YAML knowledge.

---

### Setup kubectl command

```
export KUBECONFIG=<PATH TO KUBECONFIG FILE>
```
Ensure it is working

```
kubectl get nodes
NAME                   STATUS   ROLES    AGE    VERSION
pool-4jt93kjok-e1i0q   Ready    <none>   125m   v1.31.1
```

### **Step 1: Create a Simple Pod**

#### 1. Write a Pod YAML Manifest
Create a file named `simple-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: demo
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

#### 2. Apply the Manifest
```bash
kubectl apply -f simple-pod.yaml
```

#### 3. Verify the Pod
```bash
kubectl get pods
```

#### 4. Check Logs
```bash
kubectl logs my-first-pod
```

#### 5. Access the Pod
```bash
kubectl port-forward my-first-pod 8080:80
```
Visit `http://localhost:8080` in your browser.

---

### **Step 2: Create a Multi-Container Pod**

#### 1. Write a Multi-Container Pod YAML
Create a file named `multi-container-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  labels:
    app: demo
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox:latest
    command: ["sh", "-c", "while true; do echo Hello from Sidecar; sleep 5; done"]
```

#### 2. Apply the Manifest
```bash
kubectl apply -f multi-container-pod.yaml
```

#### 3. Observe Logs from Both Containers
```bash
kubectl logs multi-container-pod -c nginx
kubectl logs multi-container-pod -c sidecar
```

---

### **Step 3: Explore Pod Networking**

#### 1. Create a Networking Pod
Write a manifest `networking-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: network-pod
  labels:
    app: demo
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "sleep 3600"]
```

#### 2. Apply the Manifest
```bash
kubectl apply -f networking-pod.yaml
```

#### 3. Exec into the Pod
```bash
kubectl exec -it network-pod -- sh
```

#### 4. Test DNS Resolution
Inside the Pod:
```bash
nslookup kubernetes.default
```

#### 5. Exit the Pod
```bash
exit
```

---

### **Step 4: Debug Pod Issues**

#### 1. Create a Failing Pod
Write a manifest `failing-pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: failing-pod
spec:
  containers:
  - name: failing-container
    image: busybox:latest
    command: ["sh", "-c", "exit 1"]
```

#### 2. Apply the Manifest
```bash
kubectl apply -f failing-pod.yaml
```

#### 3. Check the Pod Status
```bash
kubectl get pods
```

#### 4. Describe the Pod
```bash
kubectl describe pod failing-pod
```

#### 5. Delete the Pod
```bash
kubectl delete pod failing-pod
```

---

### **Step 5: Clean Up**
Delete all resources created during the lab:
```bash
kubectl delete pod my-first-pod multi-container-pod network-pod
```

---

### **What Students Will Learn**
1. How to create and manage Pods using YAML manifests.
2. How to inspect and debug Pods.
3. How Pods handle networking and multiple containers.
4. How to troubleshoot common Pod issues.

