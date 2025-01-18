
## **Lab: Working with Pods, ReplicaSets, and Services**

### **Prerequisites**
1. A Kubernetes cluster set up using Minikube, Kind, or any other method.
2. `kubectl` installed and configured to connect to the cluster.
3. Basic knowledge of YAML syntax.

---

### **Step 1: Create and Manage a Pod using YAML**
1. **Objective:** Deploy a simple Pod running an NGINX web server.
   
2. **YAML Manifest:** Create a file named `nginx-pod.yaml` with the following content:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx-pod
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:latest
       ports:
       - containerPort: 80
   ```

3. **Commands:**
   - Apply the manifest to create the Pod:
     ```bash
     kubectl apply -f nginx-pod.yaml
     ```
   - Verify the Pod is running:
     ```bash
     kubectl get pods
     ```
   - Describe the Pod to inspect its details:
     ```bash
     kubectl describe pod nginx-pod
     ```
   - Access the Pod's logs:
     ```bash
     kubectl logs nginx-pod
     ```

4. **Cleanup:** Delete the Pod (optional):
   ```bash
   kubectl delete pod nginx-pod
   ```

---

### **Step 2: Scale the Application using ReplicaSets**
1. **Objective:** Use a ReplicaSet to ensure multiple replicas of the NGINX Pod.

2. **YAML Manifest:** Create a file named `nginx-replicaset.yaml` with the following content:
   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: nginx-replicaset
   spec:
     replicas: 3
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
           image: nginx:latest
           ports:
           - containerPort: 80
   ```

3. **Commands:**
   - Apply the manifest to create the ReplicaSet:
     ```bash
     kubectl apply -f nginx-replicaset.yaml
     ```
   - Verify the ReplicaSet and Pods are running:
     ```bash
     kubectl get replicasets
     kubectl get pods
     ```
   - Scale the ReplicaSet to 5 replicas:
     ```bash
     kubectl scale replicaset nginx-replicaset --replicas=5
     ```
   - Verify the updated replica count:
     ```bash
     kubectl get replicasets
     kubectl get pods
     ```

4. **Cleanup:** Delete the ReplicaSet (optional):
   ```bash
   kubectl delete replicaset nginx-replicaset
   ```

---

### **Step 3: Expose the Application with a Service**
1. **Objective:** Expose the NGINX Pods to make them accessible externally.

2. **YAML Manifest:** Create a file named `nginx-service.yaml` with the following content:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
   spec:
     selector:
       app: nginx
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: NodePort
   ```

3. **Commands:**
   - Apply the manifest to create the Service:
     ```bash
     kubectl apply -f nginx-service.yaml
     ```
   - Verify the Service is created and note the NodePort:
     ```bash
     kubectl get services
     ```
   - Access the application using the service:
     - Obtain the Minikube cluster IP or node IP:
       ```bash
       minikube ip
       ```
     - Access the application in a browser or curl:
       ```bash
       curl http://<MINIKUBE_IP>:<NODE_PORT>
       ```

4. **Cleanup:** Delete the Service (optional):
   ```bash
   kubectl delete service nginx-service
   ```

---

### **Lab Completion**
After completing these steps, you will have:
- Created a Pod running NGINX.
- Scaled the application using a ReplicaSet.
- Exposed the application with a Service for external access.

