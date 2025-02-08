# **Kubernetes Service Concepts - Hands-on Lab**  
This lab will guide students through **Kubernetes Service concepts** using a **DigitalOcean Kubernetes (DOKS) cluster**. The students will deploy applications, expose them using different service types, and explore how traffic is routed within the cluster.


---

## **Lab Overview**  
### **Module 1: Deploying a Sample Application**  
### **Module 2: Understanding Service Types**  
### **Module 3: Exposing the Service to the Internet**  
### **Module 4: Load Balancing and External Access**  
### **Module 5: Network Policies (Bonus Challenge)**  

---

## **Module 1: Deploying a Sample Application**
### **Step 1: Create a Deployment**
We'll deploy an **NGINX-based sample app**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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

Apply the deployment:

```sh
kubectl apply -f my-app-deployment.yaml
```

### **Step 2: Verify Pods**
```sh
kubectl get pods -l app=my-app -o wide
```
Pods should be running.

---

## **Module 2: Understanding Service Types**
### **Step 3: Create a ClusterIP Service (Internal Networking)**
A **ClusterIP** service allows internal communication between pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-clusterip
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Apply and check the service:

```sh
kubectl apply -f my-app-clusterip.yaml
kubectl get svc my-app-clusterip
```

### **Step 4: Test Internal Communication**
```sh
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty
```
Inside the pod, run:
```sh
curl my-app-clusterip.default.svc.cluster.local
```
This confirms that the service is routing requests.

---

## **Module 3: Exposing the Service to the Internet**
### **Step 5: Create a NodePort Service**
A **NodePort** service exposes the app on a static port on each node.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
  type: NodePort
```

Apply it:

```sh
kubectl apply -f my-app-nodeport.yaml
kubectl get svc my-app-nodeport
```

Get the node’s public IP:

```sh
kubectl get nodes -o wide
```

Access the app via:

```
http://<NODE_IP>:30007
```

---

## **Module 4: Load Balancing and External Access**
### **Step 6: Create a LoadBalancer Service**
A **LoadBalancer** service assigns an external IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-loadbalancer
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Apply it:

```sh
kubectl apply -f my-app-loadbalancer.yaml
kubectl get svc my-app-loadbalancer
```

Wait for an **EXTERNAL-IP** to appear and access it via:

```
http://<EXTERNAL-IP>
```

---

## **Module 5: Network Policies (Bonus Challenge)**
### **Step 7: Restrict Pod-to-Pod Communication**
Create a **NetworkPolicy** to allow access only from specific pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-curl-test
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              run: curl-test
      ports:
        - protocol: TCP
          port: 80
```

Apply and test access from **curl-test**.

```sh
kubectl apply -f network-policy.yaml
kubectl exec -it curl-test -- curl my-app-clusterip.default.svc.cluster.local
```

Other pods **should be denied access**.

---

## **Clean Up**
To remove all resources:

```sh
kubectl delete deployment my-app
kubectl delete svc my-app-clusterip my-app-nodeport my-app-loadbalancer
kubectl delete networkpolicy allow-only-curl-test
kubectl delete pod curl-test
```

---

## **Key Takeaways**
✅ ClusterIP is for **internal communication**  
✅ NodePort exposes services on **each node’s IP**  
✅ LoadBalancer creates a **public endpoint**  
✅ NetworkPolicies control **pod access**  

---

## **Next Steps**
- Deploy an **Ingress Controller** for advanced routing.  
- Configure **DNS and TLS** with Kubernetes.  
- Experiment with **Headless Services** and **Service Discovery**.  

This lab provides **hands-on experience** with K8s Services, ensuring students understand **internal/external networking and security policies**. 🚀
