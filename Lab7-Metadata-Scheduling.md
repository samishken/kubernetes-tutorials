
# Kubernetes Metadata & Scheduling Lab

In this lab, you will learn and practice the following concepts:

- **Labels & Selectors:** Organize and query resources.
- **Annotations:** Attach additional metadata to resources.
- **Taints & Tolerations:** Control pod scheduling on nodes.
- **Affinity & Anti-Affinity:** Influence pod placement based on node and pod characteristics.

> **Note:** For this lab, you should have a working Kubernetes cluster and `kubectl` configured.

---

## Lab 1: Labels & Selectors

### **Objective:**
Learn how to assign labels to resources and use label selectors to filter or target these resources.

### **Steps:**

1. **Create a Sample Pod with Labels**

   Create a YAML file named `pod-labels.yaml` with the following content:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: demo-pod
     labels:
       app: demo
       environment: lab
   spec:
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
   ```

   Apply the file:

   ```bash
   kubectl apply -f pod-labels.yaml
   ```

2. **Verify the Pod and Its Labels**

   List the pod with labels:

   ```bash
   kubectl get pods --show-labels
   ```

   Or, filter by a specific label:

   ```bash
   kubectl get pods -l app=demo
   ```

3. **Update Labels (Optional)**

   You can add or modify labels on a running pod using the `kubectl label` command. For example, add a new label:

   ```bash
   kubectl label pod demo-pod version=v1
   ```

   Verify the change:

   ```bash
   kubectl get pod demo-pod --show-labels
   ```

### **Expected Outcome:**
- You will see the `demo-pod` with the labels you defined.
- Using label selectors, you can query and filter the pod.

---

## Lab 2: Annotations

### **Objective:**
Learn how to attach non-identifying metadata to a resource using annotations.

### **Steps:**

1. **Annotate the Existing Pod**

   Add an annotation to the pod (for example, build information):

   ```bash
   kubectl annotate pod demo-pod buildInfo="v1.0-2025-02-19"
   ```

2. **View Annotations**

   Describe the pod to view its annotations:

   ```bash
   kubectl describe pod demo-pod
   ```

   Look for the `Annotations:` section in the output.

3. **Update or Remove an Annotation**

   To update, simply re-run the annotate command with a new value:

   ```bash
   kubectl annotate pod demo-pod buildInfo="v1.1-2025-02-20" --overwrite
   ```

   To remove an annotation:

   ```bash
   kubectl annotate pod demo-pod buildInfo-
   ```

### **Expected Outcome:**
- The pod’s metadata now includes additional information under annotations.
- You can update or remove annotations as needed.

---

## Lab 3: Taints & Tolerations

### **Objective:**
Learn how to restrict pod scheduling using taints on nodes and allow specific pods using tolerations.

### **Steps:**

1. **Taint a Node**

   Identify a node in your cluster:

   ```bash
     kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.taints}{"\n"}{end}'
   ```

   Choose a node name (e.g., `quick-lab`) and apply a taint:

   ```bash
   kubectl taint nodes quick-labs-0-aa0mx quick-labs-0-aaalz quick-labs-0-aaapc key=database:NoSchedule
   ```

   This taint prevents pods without a toleration for `key=database` from being scheduled on the node.

2. **Deploy a Pod Without Toleration**

   Create a file `pod-no-toleration.yaml`:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: no-toleration-pod
   spec:
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
   ```

   Apply the pod:

   ```bash
   kubectl apply -f pod-no-toleration.yaml
   ```

   **Observation:**  
   This pod may remain in `Pending` because it cannot be scheduled on the tainted node.

3. **Deploy a Pod With a Toleration**

   Create a file `pod-with-toleration.yaml`:

      ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: toleration-pod
   spec:
     tolerations:
     - key: "key"
       operator: "Equal"
       value: "database"
       effect: "NoSchedule"
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]

   ```

   
   Apply the pod:

   ```bash
   kubectl apply -f pod-with-toleration.yaml
   ```

   **Observation:**  
   This pod should schedule on the tainted node.

4. **Cleanup Taints**

   Once testing is complete, remove the taint:

   ```bash
   kubectl taint nodes quick-labs-0-aa0mx quick-labs-0-aaalz quick-labs-0-aaapc key=database:NoSchedule-
   ```

### **Expected Outcome:**
- Pods without tolerations will not schedule on tainted nodes.
- Pods with matching tolerations will be scheduled despite the taint.

---

## Lab 4: Affinity & Anti-Affinity

### **Objective:**
Use node affinity and pod affinity/anti-affinity rules to control the placement of pods.

### **Steps:**

1. **Node Affinity**

   Create a YAML file `node-affinity.yaml` that schedules pods only on nodes with a specific label. First, label one of your nodes:

   ```bash
   kubectl label nodes quick-labs-0-aaapc location=toronto
   ```

   Now, create a pod that uses node affinity:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: node-affinity-pod
   spec:
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
             - key: location
               operator: In
               values:
               - toronto
     containers:
     - name: busybox
       image: busybox
       command: ["sleep", "3600"]
   ```

   Apply the YAML:

   ```bash
   kubectl apply -f node-affinity.yaml
   ```

   **Observation:**  
   The pod should schedule only on nodes labeled with `location=toronto`.

2. **Pod Affinity & Anti-Affinity**

   Create two deployments. First, create a deployment for a “base” app:

   **Base Deployment (base-deployment.yaml):**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: base-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: base
     template:
       metadata:
         labels:
           app: base
       spec:
         containers:
         - name: busybox
           image: busybox
           command: ["sleep", "3600"]
   ```

   Apply it:

   ```bash
   kubectl apply -f base-deployment.yaml
   ```

   Next, create a deployment that has a pod affinity rule to schedule near the base pods:

   **Affinity Deployment (affinity-deployment.yaml):**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: affinity-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: affinity
     template:
       metadata:
         labels:
           app: affinity
       spec:
         affinity:
           podAffinity:
             requiredDuringSchedulingIgnoredDuringExecution:
             - labelSelector:
                 matchExpressions:
                 - key: app
                   operator: In
                   values:
                   - base
               topologyKey: "kubernetes.io/hostname"
         containers:
         - name: busybox
           image: busybox
           command: ["sleep", "3600"]
   ```

   Apply the deployment:

   ```bash
   kubectl apply -f affinity-deployment.yaml
   ```

   **Observation:**  
   Pods in the `affinity-deployment` should schedule on nodes where pods from `base-deployment` are running.

   Now, to experiment with anti-affinity, modify the deployment to include pod anti-affinity rules that prefer to spread pods apart from the base pods. Change the affinity block in the YAML to something like:

   ```yaml
   affinity:
     podAntiAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
       - labelSelector:
           matchExpressions:
           - key: app
             operator: In
             values:
             - base
         topologyKey: "kubernetes.io/hostname"
   ```

   Apply the modified deployment (or create a new file for testing). Observe how scheduling changes.

### **Expected Outcome:**
- **Node Affinity:** Pods are scheduled only on nodes with the matching node label.
- **Pod Affinity:** Pods with affinity rules try to co-locate with similar pods.
- **Pod Anti-Affinity:** Pods with anti-affinity rules avoid being scheduled near specified pods, which is useful for spreading workloads for high availability.

---

## Lab Wrap-Up

### **Review and Discussion:**

- **What did you learn?**  
  - How labels and annotations can be used for organization and metadata.
  - How taints/tolerations control scheduling on nodes.
  - How affinity rules can optimize workload placement.

- **Troubleshooting:**  
  - Use `kubectl get pods --show-labels` and `kubectl describe pod <pod-name>` to inspect metadata and scheduling details.
  - Check node taints with `kubectl describe node <node-name>`.

- **Cleanup (Optional):**  
  Remove test resources once you’re done:
  
  ```bash
  kubectl delete -f pod-labels.yaml
  kubectl delete pod no-toleration-pod toleration-pod node-affinity-pod
  kubectl delete deployment base-deployment affinity-deployment
  ```

