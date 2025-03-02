
# Kubernetes Networking & Service Management Lab  

## Chapter 6: Exercises  

### 1. Create Frontend and Backend Pods  

#### Step 1: Create the Frontend Pod  

Create a Pod named `frontend` using the `nginx` image. Assign the labels `tier=frontend` and `app=nginx`, and expose port `80`:  

```sh
kubectl run frontend --image=nginx --restart=Never --port=80 \
-l tier=frontend,app=nginx
```

Verify the Pod creation:  

```sh
kubectl get pods
```

#### Step 2: Create the Backend Pod  

Similarly, create a Pod named `backend` with `nginx` and assign the labels `tier=backend` and `app=nginx`:  

```sh
kubectl run backend --image=nginx --restart=Never --port=80 \
-l tier=backend,app=nginx
```

Verify the Pod creation:  

```sh
kubectl get pods
```

---

### 2. Create a Service for the Backend  

#### Step 1: Generate a Service YAML Manifest  

Create a `ClusterIP` Service named `nginx-service` that exposes port `9000` and targets port `8081`:  

```sh
kubectl create service clusterip nginx-service --tcp=9000:8081 \
--dry-run=client -o yaml > nginx-service.yaml
```

#### Step 2: Modify the YAML  

Edit `nginx-service.yaml` to ensure that the Service correctly selects backend Pods. Update the selector and target port as follows:  

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-service
  name: nginx-service
spec:
  ports:
    - port: 9000
      protocol: TCP
      targetPort: 80
  selector:
    tier: backend
    app: nginx
  type: ClusterIP
```

#### Step 3: Deploy the Service  

```sh
kubectl create -f nginx-service.yaml
kubectl get services
```

---

### 3. Verify Service Connectivity  

#### Step 1: Test Connectivity from Within the Cluster  

Run a temporary `busybox` Pod and attempt to reach the `nginx-service`:  

```sh
kubectl run busybox --image=busybox --restart=Never -it --rm -- /bin/sh
```

Inside the shell, attempt to connect to the Service (replace `<SERVICE_IP>` with the actual Service IP from `kubectl get services`):  

```sh
wget --spider --timeout=1 <SERVICE_IP>:9000
```

#### Step 2: Expose the Service to External Traffic  

Convert `nginx-service` to a `NodePort` Service:  

```sh
kubectl patch service nginx-service -p \
'{ "spec": {"type": "NodePort"} }'
```

Verify the change:  

```sh
kubectl get services
```

Retrieve the node’s external IP:  

```sh
kubectl get nodes
kubectl describe node minikube | grep InternalIP:
```

Now, access the Service externally (replace `<NODE_IP>` and `<NODE_PORT>` with actual values from `kubectl get services`):  

```sh
wget --spider --timeout=1 <NODE_IP>:<NODE_PORT>
```

---

### 4. Deploy an Application Stack  

#### Step 1: Create a Namespace  

Create a namespace for the app stack:  

```sh
kubectl create namespace app-stack
```

#### Step 2: Deploy the Application Stack  

Copy the `app-stack.yaml` file and apply it:  

```sh
kubectl apply -f app-stack.yaml
```

Verify the deployment:  

```sh
kubectl get pods -n app-stack
```

---

### 5. Configure Network Policies  

#### Step 1: Create a Network Policy  

Create `app-stack-network-policy.yaml` to allow only **backend** Pods to access the **database** Pods:  

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: todo
              tier: backend
```

Apply the network policy:  

```sh
kubectl create -f app-stack-network-policy.yaml
kubectl get networkpolicy -n app-stack
```

#### Step 2: Restrict Database Access to Port 3306  

Modify the network policy to only allow incoming traffic on **TCP port 3306**:  

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-stack-network-policy
  namespace: app-stack
spec:
  podSelector:
    matchLabels:
      app: todo
      tier: database
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: todo
              tier: backend
      ports:
        - protocol: TCP
          port: 3306
```

Apply the updated policy:  

```sh
kubectl apply -f app-stack-network-policy.yaml
```

Verify the applied network rules:  

```sh
kubectl describe networkpolicy app-stack-network-policy -n app-stack
```

---

## Summary  

- **Created frontend and backend Pods** with appropriate labels.  
- **Deployed a ClusterIP Service**, fixed label selector issues, and exposed it using `NodePort`.  
- **Created and managed an application stack** with three layers: frontend, backend, and database.  
- **Implemented network policies** to restrict database access only to backend Pods.  

This lab covers **Kubernetes networking, services, and security policies** for real-world deployments. 🚀
