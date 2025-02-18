### **Lab: Kubernetes Node Labels and Taints**

In this lab, you will practice using **node labels** and **taints** in Kubernetes to control pod scheduling. Node labels allow you to group nodes based on certain characteristics, and taints allow you to repel or attract pods to certain nodes based on specific conditions. You'll be working with labels and taints to demonstrate how they can affect pod placement.

---

### **Learning Objectives:**
By the end of this lab, you should be able to:
1. Label nodes in Kubernetes.
2. Use taints and tolerations to control pod scheduling.
3. Understand how labels, taints, and tolerations work together to manage pod placement.
4. Implement node affinity and anti-affinity.

---

### **Lab: Using Node Labels and Taints to Control Pod Placement**

---

### **Step 1: Set Up Your Kubernetes Cluster**

If you don't already have a cluster, set one up using a method suitable for your environment, such as:

- **Minikube** (Local single-node cluster).
- **K3s** (Lightweight Kubernetes cluster).
- **GKE, EKS, or AKS** (Cloud-managed clusters).

Make sure you have access to your `kubectl` command to interact with the cluster.

---

### **Step 2: Apply Node Labels**

Node labels allow you to categorize nodes by characteristics such as region, hardware capabilities, or other custom labels.

1. **Label Nodes:**
   
   Let's assume you have two nodes (`node-1` and `node-2`) and you want to label them based on their roles.

   - Label `node-1` as a `web` node:

     ```bash
     kubectl label nodes node-1 role=web
     ```

   - Label `node-2` as a `db` node:

     ```bash
     kubectl label nodes node-2 role=db
     ```

2. **Verify Node Labels:**

   After labeling, you can check the node labels:

   ```bash
   kubectl get nodes --show-labels
   ```

   You should see `role=web` on `node-1` and `role=db` on `node-2`.

---

### **Step 3: Create Pods Using Node Labels**

Now, we will create two pods: one will be scheduled on the `web` node and the other on the `db` node based on the labels you created.

1. **Pod for Web Application (web-pod.yaml):**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: web-pod
   spec:
     containers:
       - name: nginx
         image: nginx
     nodeSelector:
       role: web
   ```

   Apply the pod configuration:

   ```bash
   kubectl apply -f web-pod.yaml
   ```

2. **Pod for Database Application (db-pod.yaml):**

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: db-pod
   spec:
     containers:
       - name: mysql
         image: mysql:5.7
         env:
           - name: MYSQL_ROOT_PASSWORD
             value: "password"
     nodeSelector:
       role: db
   ```

   Apply the pod configuration:

   ```bash
   kubectl apply -f db-pod.yaml
   ```

3. **Verify Pod Scheduling:**

   Check if the pods are running and verify that they are scheduled to the correct nodes:

   ```bash
   kubectl get pods -o wide
   ```

---

### **Step 4: Use Taints to Control Pod Placement**

Taints allow you to prevent pods from being scheduled on specific nodes unless they have matching tolerations.

1. **Taint Node-1 (web node):**

   Let’s say you want to prevent pods from running on `node-1` unless they specifically tolerate the taint.

   Apply the taint to `node-1`:

   ```bash
   kubectl taint nodes node-1 role=web:NoSchedule
   ```

   This means that no pod will be scheduled on `node-1` unless it has a matching toleration.

2. **Create Pod with Toleration (tolerate-taint-pod.yaml):**

   Create a pod that tolerates the taint applied to `node-1`:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: tolerate-taint-pod
   spec:
     containers:
       - name: nginx
         image: nginx
     tolerations:
       - key: "role"
         operator: "Equal"
         value: "web"
         effect: "NoSchedule"
   ```

   Apply the pod:

   ```bash
   kubectl apply -f tolerate-taint-pod.yaml
   ```

3. **Check Pod Scheduling:**

   Verify that the pod `tolerate-taint-pod` is scheduled to `node-1` despite the taint:

   ```bash
   kubectl get pods -o wide
   ```

   The pod should now be scheduled on `node-1`, as it has the correct toleration.

4. **Remove the Taint:**

   If you want to allow normal pod scheduling on `node-1` again, remove the taint:

   ```bash
   kubectl taint nodes node-1 role=web:NoSchedule-
   ```

---

### **Step 5: Experiment with Affinity and Anti-Affinity**

**Node Affinity** allows you to control which nodes a pod can be scheduled on based on labels, similar to node selectors, but more flexible. Anti-affinity prevents pods from being scheduled on nodes that have specific characteristics.

1. **Create a Pod with Node Affinity (affinity-pod.yaml):**

   We will create a pod that prefers to run on a node labeled as `role=web`.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: affinity-pod
   spec:
     containers:
       - name: nginx
         image: nginx
     affinity:
       nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
             - matchExpressions:
                 - key: "role"
                   operator: In
                   values:
                     - web
   ```

   Apply the pod configuration:

   ```bash
   kubectl apply -f affinity-pod.yaml
   ```

2. **Create a Pod with Anti-Affinity (anti-affinity-pod.yaml):**

   We will create a pod that should not be scheduled on any node that already has a pod with the label `role=web`.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: anti-affinity-pod
   spec:
     containers:
       - name: nginx
         image: nginx
     affinity:
       podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           - labelSelector:
               matchExpressions:
                 - key: "role"
                   operator: In
                   values:
                     - web
             topologyKey: "kubernetes.io/hostname"
   ```

   Apply the pod configuration:

   ```bash
   kubectl apply -f anti-affinity-pod.yaml
   ```

3. **Check Pod Scheduling:**

   Verify that the **affinity-pod** is scheduled on a node with the `role=web` label, and the **anti-affinity-pod** is not scheduled on a node that already has a pod labeled `role=web`:

   ```bash
   kubectl get pods -o wide
   ```

---

### **Step 6: Clean Up**

After completing the exercises, you should clean up the resources.

1. **Delete Pods and Resources:**

   ```bash
   kubectl delete pod web-pod db-pod tolerate-taint-pod affinity-pod anti-affinity-pod
   kubectl delete nodes node-1 node-2
   ```

2. **Delete Taints and Labels:**

   Remove taints and labels from the nodes:

   ```bash
   kubectl taint nodes node-1 role=web:NoSchedule-
   kubectl label nodes node-1 role-
   kubectl label nodes node-2 role-
   ```

---

### **Assignment:**

1. **Task 1:** Create a 3-node cluster with labels like `role=app`, `role=db`, and `role=cache`. Label the nodes accordingly. Create different pods and use `nodeSelector` to ensure that they are placed on the correct node based on the label.
   
2. **Task 2:** Apply taints to one node with `role=cache`, so no pods can be scheduled there unless they tolerate the taint. Create a pod that tolerates this taint.

3. **Task 3:** Experiment with **nodeAffinity** and **podAntiAffinity** by creating a pod that prefers a certain node (`role=app`), and another pod that should not be scheduled on the same node.

4. **Task 4:** Consider a scenario where you need to run a pod on a node only if the node has a certain label. Use **affinity** and **anti-affinity** to solve this problem.

---

