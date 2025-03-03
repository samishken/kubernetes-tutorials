# Kubernetes Probes & Debugging Lab  


### 1. Create a Pod with an HTTP Startup Probe  

#### Step 1: Generate a Pod YAML Manifest  
We will create a Pod named `web-server` using the `nginx` image and expose port `80`. Use the `kubectl run` command in dry-run mode to generate the YAML file:  

```sh
kubectl run web-server --image=nginx --port=80 --restart=Never \
-o yaml --dry-run=client > probed-pod.yaml
```

#### Step 2: Add a Startup Probe  
Edit the `probed-pod.yaml` file to include a **startup probe** that performs an HTTP GET request on the root (`/`) path of the container:  

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-server
  name: web-server
spec:
  containers:
    - image: nginx
      name: web-server
      ports:
        - containerPort: 80
          name: nginx-port
      startupProbe:
        httpGet:
          path: /
          port: nginx-port
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

---

### 2. Add a Readiness Probe  

Modify the `probed-pod.yaml` to include a **readiness probe**. This probe ensures that the container is ready to accept traffic. It should:  
- Perform an HTTP GET request on `/`.  
- Wait **5 seconds** before the first check.  

```yaml
readinessProbe:
  httpGet:
    path: /
    port: nginx-port
  initialDelaySeconds: 5
```

Updated `probed-pod.yaml`:  

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-server
  name: web-server
spec:
  containers:
    - image: nginx
      name: web-server
      ports:
        - containerPort: 80
          name: nginx-port
      startupProbe:
        httpGet:
          path: /
          port: nginx-port
      readinessProbe:
        httpGet:
          path: /
          port: nginx-port
        initialDelaySeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

---

### 3. Add a Liveness Probe  

Modify `probed-pod.yaml` to include a **liveness probe**. This probe ensures that the container is still running properly. It should:  
- Perform an HTTP GET request on `/`.  
- Wait **10 seconds** before the first check.  
- Check every **30 seconds**.  

```yaml
livenessProbe:
  httpGet:
    path: /
    port: nginx-port
  initialDelaySeconds: 10
  periodSeconds: 30
```

Final `probed-pod.yaml`:  

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-server
  name: web-server
spec:
  containers:
    - image: nginx
      name: web-server
      ports:
        - containerPort: 80
          name: nginx-port
      startupProbe:
        httpGet:
          path: /
          port: nginx-port
      readinessProbe:
        httpGet:
          path: /
          port: nginx-port
        initialDelaySeconds: 5
      livenessProbe:
        httpGet:
          path: /
          port: nginx-port
        initialDelaySeconds: 10
        periodSeconds: 30
  dnsPolicy: ClusterFirst
  restartPolicy: Never
```

---

### 4. Deploy and Monitor the Pod  

#### Step 1: Create the Pod  
```sh
kubectl create -f probed-pod.yaml
```

#### Step 2: Verify Pod Status  
```sh
kubectl get pod web-server
```
The output should show the container transitioning from `ContainerCreating` to `Running`.  

#### Step 3: Inspect Probe Configurations  
```sh
kubectl describe pod web-server
```

#### Step 4: Retrieve Pod Metrics  
Use the Kubernetes Metrics Server to check CPU and memory usage:  

```sh
kubectl top pod web-server
```

---

### 5. Debugging a Failed Pod  

#### Step 1: Create a Pod with a Custom Command  

We will create a Pod named `custom-cmd` using the `busybox` image. The container will attempt to run `top-analyzer --all`, which **does not exist** in the image:  

```sh
kubectl run custom-cmd --image=busybox --restart=Never \
-- /bin/sh -c "top-analyzer --all"
```

#### Step 2: Check Pod Status  
```sh
kubectl get pod custom-cmd
```
The output will show the Pod in `Error` state.  

#### Step 3: Troubleshoot the Pod  

Check the logs for failure messages:  
```sh
kubectl logs custom-cmd
```
The output will indicate that `top-analyzer` is not available in the `busybox` image.  

---

## Summary  

- We created a **Pod with probes** to ensure proper startup, readiness, and liveness.  
- We monitored the Pod’s **lifecycle and resource usage**.  
- We troubleshot a **failed Pod** due to a missing command.  

This lab helps in understanding Kubernetes **probes, monitoring, and debugging** in real-world scenarios. 🚀
