### **Lab: Overview of Declarative vs. Imperative Configuration in Kubernetes**

In this lab, you'll learn the fundamental differences between **declarative** and **imperative** configuration in Kubernetes, explore their advantages and disadvantages, and practice both approaches to managing Kubernetes resources.

---

### **Learning Objectives**
1. Understand the difference between declarative and imperative approaches in Kubernetes.
2. Learn how to use both methods to manage resources like Pods, Deployments, and Services.
3. Identify scenarios where one approach is more suitable than the other.

---

### **Prerequisites**
- A Kubernetes cluster (can be a local cluster using Minikube, Kind, or a cloud provider like GKE, EKS, or AKS).
- `kubectl` installed and configured to access your cluster.
- Basic understanding of Kubernetes resources.

---

### **1. Key Concepts**

#### **Imperative Configuration**
- You provide commands directly to create, update, or delete resources.
- Changes are made immediately.
- Commands are one-time and not stored as source files (unless explicitly saved).
  
  **Example:**
  ```bash
  kubectl run nginx --image=nginx --restart=Never
  ```

#### **Declarative Configuration**
- You define the desired state of resources in configuration files (YAML or JSON).
- Kubernetes reconciles the cluster to match the desired state.
- Changes are version-controllable and repeatable.

  **Example:**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
  ```
  Apply the file using:
  ```bash
  kubectl apply -f nginx-pod.yaml
  ```

---

### **2. Lab Environment Setup**
Ensure your Kubernetes cluster is up and running. Test connectivity with:
```bash
kubectl get nodes
```

---

### **3. Imperative Approach**
#### Step 1: Create a Pod
Run the following command to create an nginx Pod imperatively:
```bash
kubectl run nginx --image=nginx --restart=Never
```

#### Step 2: Verify the Pod
Check if the Pod is running:
```bash
kubectl get pods
```

#### Step 3: Update the Pod
Imperatively scale the Pod to two replicas using a Deployment:
```bash
kubectl create deployment nginx-deployment --image=nginx
kubectl scale deployment nginx-deployment --replicas=2
```

#### Step 4: Clean Up
Delete the resources imperatively:
```bash
kubectl delete pod nginx
kubectl delete deployment nginx-deployment
```

---

### **4. Declarative Approach**
#### Step 1: Create a YAML File
Create a file named `nginx-deployment.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

#### Step 2: Apply the YAML File
Apply the configuration using `kubectl`:
```bash
kubectl apply -f nginx-deployment.yaml
```

#### Step 3: Verify the Deployment
List the running Pods:
```bash
kubectl get pods
```

#### Step 4: Update the Deployment
Edit the YAML file to update the image to `nginx:1.21`:
```yaml
containers:
- name: nginx
  image: nginx:1.21
```

Apply the changes:
```bash
kubectl apply -f nginx-deployment.yaml
```

#### Step 5: Clean Up
Delete the resources declaratively:
```bash
kubectl delete -f nginx-deployment.yaml
```

---

### **5. Comparison**
| Feature                 | Imperative                          | Declarative                       |
|-------------------------|--------------------------------------|-----------------------------------|
| Ease of Use             | Simple and quick for small tasks    | Better for managing complex setups |
| Version Control         | Not easily version-controllable     | YAML files can be stored in Git   |
| Repeatability           | Not easily repeatable               | Fully repeatable                  |
| Scalability             | Suitable for small changes          | Ideal for large, complex systems  |

---

### **6. Summary**
- **Imperative** configuration is useful for quick, one-off tasks but lacks repeatability and version control.
- **Declarative** configuration is better for managing infrastructure at scale, as it ensures a consistent desired state and is GitOps-friendly.

