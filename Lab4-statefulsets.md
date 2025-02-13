### **Kubernetes Lab: StatefulSets**

#### **Objective:**
Learn how to create, scale, and manage **StatefulSets** in Kubernetes, and understand how they handle stateful applications with persistent storage and stable network identities.

#### **Prerequisites:**
- A running Kubernetes cluster (can use Minikube, kind, or a cloud-based cluster).
- `kubectl` command-line tool installed and configured to access the cluster.
- Basic understanding of Kubernetes concepts like pods, deployments, and services.

---

### **Lab Steps:**

#### **1. Create a StatefulSet**

**Goal:** Deploy a simple StatefulSet that manages pods with persistent storage and stable network identities.

1. **Create a StatefulSet YAML file** (`statefulset.yaml`) with the following content:

   ```yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: web
   spec:
     serviceName: "web"
     replicas: 3
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web
       spec:
         containers:
         - name: nginx
           image: nginx:latest
           ports:
           - containerPort: 80
     volumeClaimTemplates:
     - metadata:
         name: web-storage
       spec:
         accessModes: ["ReadWriteOnce"]
         resources:
           requests:
             storage: 1Gi
   ```

   - This defines a StatefulSet with 3 replicas, each running an Nginx container.
   - A persistent volume (1Gi) is created for each pod, and it is tied to the pod via a `volumeClaimTemplates` section.

2. **Deploy the StatefulSet:**
   ```bash
   kubectl apply -f statefulset.yaml
   ```

3. **Verify the StatefulSet and Pods:**
   ```bash
   kubectl get statefulsets
   kubectl get pods
   ```

   Notice that the pod names will be `web-0`, `web-1`, and `web-2`, which are stable and unique.

4. **Check Persistent Volumes:**
   Each pod in the StatefulSet will have a corresponding persistent volume. Verify this by running:
   ```bash
   kubectl get pvc
   ```

---

#### **2. Create a Headless Service for StatefulSet**

**Goal:** Learn how to expose StatefulSet pods using a headless service, which allows direct communication with individual pods.

1. **Create a headless service YAML file** (`web-service.yaml`) with the following content:

   ```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer

   ```

2. **Deploy the headless service:**
   ```bash
   kubectl apply -f web-service.yaml
   ```

3. **Verify the service:**
   ```bash
   kubectl get svc
   ```

This allows pods to be accessed directly using their stable DNS names like `web-0.web`, `web-1.web`, etc.

---

#### **3. Scale the StatefulSet**

**Goal:** Learn how to scale the StatefulSet and observe how Kubernetes handles scaling for stateful applications.

1. **Scale the StatefulSet:**
   Increase the number of replicas from 3 to 5:
   ```bash
   kubectl scale statefulset web --replicas=5
   ```

2. **Verify the scaling:**
   After scaling, check that new pods are created with stable names (`web-3`, `web-4`).
   ```bash
   kubectl get pods
   ```

3. **Check the Persistent Volumes:**
   Ensure new PersistentVolumeClaims (PVCs) are created for the new pods:
   ```bash
   kubectl get pvc
   ```

---

#### **4. Pod Restart and Identity Preservation**

**Goal:** Observe how StatefulSets preserve the identity and storage of a pod even when it is restarted or rescheduled.

1. **Delete a Pod:**
   Delete the pod `web-0` to simulate a failure:
   ```bash
   kubectl delete pod web-0
   ```

2. **Verify the pod recreation:**
   Kubernetes will recreate the deleted pod (`web-0`). The pod will retain its identity (`web-0`), and the associated persistent volume will be reattached to it.
   ```bash
   kubectl get pods
   ```

3. **Check Persistent Volumes:**
   Ensure that the persistent volume for the deleted pod (`web-0`) is reattached:
   ```bash
   kubectl get pvc
   ```

---

#### **5. Rolling Updates with StatefulSet**

**Goal:** Learn how StatefulSets handle rolling updates and ensure that stateful applications are updated in order.

1. **Update the StatefulSet Image:**
   Update the image of the containers to a new version of Nginx:
   ```bash
   kubectl set image statefulset/web nginx=nginx:1.21
   ```

2. **Verify the rolling update:**
   The update will happen one pod at a time, preserving the order of the pods (`web-0`, `web-1`, etc.). Check the update progress:
   ```bash
   kubectl rollout status statefulset/web
   ```

3. **Verify the updated pods:**
   Once the update is complete, verify that the pods are using the updated image:
   ```bash
   kubectl get pods
   ```

---

### **Post-Lab Questions:**

1. **What makes StatefulSets suitable for stateful applications?**
2. **How does a StatefulSet handle scaling differently than a Deployment?**
3. **How does Kubernetes ensure that each pod in a StatefulSet gets its own persistent storage?**
4. **Why is a headless service used with StatefulSets?**
5. **What happens if you delete a pod in a StatefulSet, and how does it relate to pod identity?**
6. **Explain the difference between StatefulSets and Deployments in terms of updating and managing pods.**

---

### **Lab Summary:**

- **StatefulSets** are ideal for applications that require stable network identities and persistent storage.
- **Headless services** allow direct access to StatefulSet pods via their stable DNS names.
- **Scaling** a StatefulSet increases or decreases the number of replicas while ensuring each pod has its own storage.
- **Rolling updates** ensure that stateful applications are updated in a controlled and ordered manner.
