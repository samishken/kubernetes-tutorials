# **Kubernetes Storage Lab**

## **Objective**
This hands-on lab will introduce students to Kubernetes storage concepts using a **DigitalOcean Kubernetes (DOKS) cluster**. Students will learn how to work with various Kubernetes storage methods, including **emptyDir, hostPath, configMap, secret, and PVC**.

---

## **Prerequisites**
-  Installed and configured **kubectl**.
-  basic understanding of Kubernetes concepts.

---

## **Lab Outline**
### **Module 1: Using `emptyDir` for Temporary Storage**
`emptyDir` creates an ephemeral storage volume that exists as long as the Pod is running.

#### **Step 1: Deploy a Pod with `emptyDir`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/tmp/data"
      name: temp-storage
  volumes:
  - name: temp-storage
    emptyDir: {}
```
Apply the Pod:
```sh
kubectl apply -f emptydir-pod.yaml
```
Check the volume:
```sh
kubectl exec -it emptydir-demo-pod -- ls /tmp/
```

---

### **Module 2: Using `hostPath` to Mount a Host Directory**
`hostPath` mounts a directory from the Kubernetes node into the Pod.

#### **Step 2: Deploy a Pod with `hostPath`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/host-data"
      name: host-storage
  volumes:
  - name: host-storage
    hostPath:
      path: "/mnt/data"
      type: DirectoryOrCreate
```
Apply the Pod:
```sh
kubectl apply -f hostpath-pod.yaml
```

---

### **Module 3: Using `configMap` for Configuration Data**

#### **Step 3: Create a ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.json: |
    {
      "key": "value"
    }
```
Apply the ConfigMap:
```sh
kubectl apply -f configmap.yaml
```

#### **Step 4: Mount `configMap` in a Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/etc/config"
      name: config-volume
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```
Apply the Pod:
```sh
kubectl apply -f configmap-pod.yaml
```

- Assignment Validate if the ConfigMap file has been correctly mounted by validating the file contentn
---

### **Module 4: Using `secret` for Sensitive Data**

#### **Step 5: Create a Secret**
```sh
kubectl create secret generic my-secret --from-literal=password=supersecret
```

#### **Step 6: Mount `secret` in a Pod**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/etc/secret"
      name: secret-volume
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```
Apply the Pod:
```sh
kubectl apply -f secret-pod.yaml
```

---

### **Module 5: Using PVC for Persistent Storage**

#### **Step 7: Create a Persistent Volume Claim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: do-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: do-block-storage 
```
Apply the PVC:
```sh
kubectl apply -f pvc.yaml
```

#### **Step 8: Deploy a Pod with PVC**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage-volume
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: do-pvc
```
Apply the Pod:
```sh
kubectl apply -f pvc-pod.yaml
```

---

### **Module 6: Cleaning Up**
To delete all resources:
```sh
kubectl delete pod emptydir-demo-pod hostpath-demo-pod configmap-demo-pod secret-demo-pod pvc-demo-pod
kubectl delete pvc do-pvc
kubectl delete secret my-secret
kubectl delete configmap app-config
kubectl delete sc do-block-storage-class
```

---

## **Assignment**
### **Task 1: Deploy a Multi-Container Pod with Shared Storage**
- Create a pod with **two containers**.
- Use `emptyDir` to share data between the containers.
- One container writes a file, and the other container reads it.

### **Task 2: Configure a PersistentVolume and Use it in a Deployment**
- Create a **PersistentVolume (PV)** manually.
- Create a **PersistentVolumeClaim (PVC)** that binds to the PV.
- Deploy an **NGINX Deployment** that uses the PVC for storage.

### **Task 3: Secure Application Configurations**
- Create a **ConfigMap** and **Secret**.
- Deploy a pod that mounts the **ConfigMap** and **Secret**.
- Verify the contents inside the container.


