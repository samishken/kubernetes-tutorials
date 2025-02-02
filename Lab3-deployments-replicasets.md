
### **Kubernetes Lab: Deployments and ReplicaSets**

#### **Objective:**
Learn how to create, manage, and scale Deployments and ReplicaSets. Understand the difference between them and how Kubernetes ensures high availability and scalability.

#### **Prerequisites:**
- Kubernetes cluster set up (can use Minikube, kind, or a cloud-based cluster)
- kubectl command-line tool installed and configured to access the cluster

---

### **Lab Steps:**

#### **1. Create a ReplicaSet**

**Goal:** Understand the basic functionality of a ReplicaSet, which ensures a specific number of identical pods are running.

1. **Create a simple ReplicaSet YAML file** (`replicaset.yaml`) with the following content:
   
   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: my-replicaset
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   ```

2. **Deploy the ReplicaSet:**
   ```bash
   kubectl apply -f replicaset.yaml
   ```

3. **Verify the ReplicaSet and Pods:**
   ```bash
   kubectl get replicasets
   kubectl get pods
   ```

4. **Scale the ReplicaSet:**
   Increase the number of pods by scaling the ReplicaSet.
   ```bash
   kubectl scale replicaset my-replicaset --replicas=5
   ```

5. **Verify the scaling:**
   ```bash
   kubectl get pods
   ```

---

#### **2. Create a Deployment**

**Goal:** Create a Deployment that manages a ReplicaSet, providing automatic updates and rollbacks.

1. **Create a simple Deployment YAML file** (`deployment.yaml`) with the following content:
   
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
   ```

2. **Deploy the Deployment:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Verify the Deployment and Pods:**
   ```bash
   kubectl get deployments
   kubectl get pods
   ```

4. **Update the Deployment:**
   Update the Deployment to use a different version of Nginx.
   ```bash
   kubectl set image deployment/my-deployment nginx=nginx:1.21
   ```

5. **Verify the rolling update:**
   Check the status of the update.
   ```bash
   kubectl rollout status deployment/my-deployment
   ```

6. **Rollback the Deployment:**
   If something goes wrong, roll back the update.
   ```bash
   kubectl rollout undo deployment/my-deployment
   ```

7. **Verify the rollback:**
   Ensure the deployment has rolled back to the previous version.
   ```bash
   kubectl get pods
   ```

---

#### **3. Compare Deployments and ReplicaSets**

**Goal:** Understand the differences between Deployments and ReplicaSets.

1. **Check if the Deployment created a ReplicaSet:**
   ```bash
   kubectl get replicasets
   ```

2. **Scale the Deployment:**
   Scale the Deployment by changing the number of replicas.
   ```bash
   kubectl scale deployment my-deployment --replicas=4
   ```

3. **Verify the scaling:**
   Check if the pods are updated automatically when scaling the Deployment.
   ```bash
   kubectl get pods
   ```

4. **Remove the Deployment (and the ReplicaSet):**
   Delete the Deployment and observe that the ReplicaSet is also removed.
   ```bash
   kubectl delete deployment my-deployment
   ```

5. **Delete the ReplicaSet (for cleanup):**
   ```bash
   kubectl delete replicaset my-replicaset
   ```

---

### **Post-Lab Questions:**

1. **What happened when you scaled the ReplicaSet? How did Kubernetes ensure the desired number of pods were always running?**
2. **How does a Deployment differ from a ReplicaSet in terms of updates and rollback?**
3. **Why would you prefer using a Deployment over directly managing ReplicaSets?**
4. **What would happen if you manually deleted a pod from a Deployment-managed ReplicaSet?**
5. **What role does the `spec.selector` play in both Deployments and ReplicaSets?**

---

### **Lab Summary:**

- **ReplicaSet** ensures the specified number of identical pods are running but lacks support for updates and rollbacks.
- **Deployment** manages ReplicaSets, supports rolling updates, and provides rollback capabilities, making it easier to manage applications.
