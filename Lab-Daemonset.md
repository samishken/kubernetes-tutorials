### **Kubernetes DaemonSet Lab: Deploying NGINX on All Nodes with a Load Balancer**  

This hands-on lab will guide you through deploying **NGINX as a DaemonSet** on a Kubernetes cluster. Each node will run an NGINX pod, and we'll expose it via a **LoadBalancer service**.

---

## **📌 Lab Objectives**
✅ Deploy an **NGINX DaemonSet** to ensure every node runs an NGINX instance  
✅ Verify pod distribution across all nodes  
✅ Expose NGINX using a **LoadBalancer**  
✅ Test accessibility and troubleshoot  

---

## **🛠 Prerequisites**
- A running Kubernetes cluster (`minikube`, `kind`, or cloud-based cluster)
- `kubectl` installed  
- A cloud provider or MetalLB (for local LoadBalancer support)

---

## **Step 1: Create the DaemonSet YAML File**
Create a file called **`nginx-daemonset.yaml`** with the following content:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  labels:
    app: nginx
spec:
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
        resources:
          limits:
            cpu: "500m"
            memory: "256Mi"
          requests:
            cpu: "250m"
            memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

## **Step 2: Deploy the DaemonSet**
Apply the YAML file:

```bash
kubectl apply -f nginx-daemonset.yaml
```

---

## **Step 3: Verify Deployment**
Check if NGINX pods are running on all nodes:

```bash
kubectl get pods -o wide
```

You should see **one NGINX pod per node**.

---

## **Step 4: Expose the DaemonSet with LoadBalancer**
Check if the LoadBalancer service is created:

```bash
kubectl get svc nginx-lb
```

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80

```
Then, configure an IP range for MetalLB to use.

---

## **Step 5: Access the NGINX DaemonSet**
Find the external IP:

```bash
kubectl get svc nginx-lb
```

Open the **External IP** in your browser or run:

```bash
curl http://<EXTERNAL-IP>
```

---

## **Step 6: Cleanup**
To delete the resources:

```bash
kubectl delete -f nginx-daemonset.yaml
```

---

## **🎯 Key Takeaways**
- **DaemonSets** ensure each node runs an instance of a pod.  
- **LoadBalancers** distribute traffic across all running instances.  
- Useful for **log collectors, monitoring agents, or node-wide applications**.

---

💬 **Questions? Drop a comment below!** 🚀
