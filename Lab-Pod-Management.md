
# Kubernetes Pod Management & Deployments Lab  

## Chapter 5: Exercises  

### 1. Create and Label Multiple Pods  

#### Step 1: Create Three Pods with Labels  

Create three Pods using the `nginx` image:  

```sh
kubectl run pod-1 --image=nginx --restart=Never \
--labels=tier=frontend,team=artemidis

kubectl run pod-2 --image=nginx --restart=Never \
--labels=tier=backend,team=artemidis

kubectl run pod-3 --image=nginx --restart=Never \
--labels=tier=backend,team=artemidis
```

#### Step 2: Verify Pod Labels  

```sh
kubectl get pods --show-labels
```

---

### 2. Annotate Pods  

#### Step 1: Assign Annotations  

Annotate `pod-1` and `pod-3` with the key `deployer` and use your name as the value:  

```sh
kubectl annotate pod pod-1 pod-3 deployer="Benjamin Muschko"
```

#### Step 2: Verify Annotations  

```sh
kubectl describe pod pod-1 pod-3 | grep Annotations:
```

---

### 3. Select Pods Using Label Filters  

Retrieve all Pods that belong to either the `artemidis` or `aircontrol` teams and are categorized as backend services:  

```sh
kubectl get pods -l tier=backend,'team in (artemidis,aircontrol)' --show-labels
```

---

### 4. Create and Troubleshoot a Deployment  

#### Step 1: Create a Deployment  

Create a new Deployment named `server-deployment` using the image `grand-server:1.4.6`:  

```sh
kubectl create deployment server-deployment --image=grand-server:1.4.6
```

#### Step 2: Verify Deployment Status  

```sh
kubectl get deployments
kubectl get pods
kubectl describe pod server-deployment-<POD_ID>
```

The Deployment will fail because `grand-server:1.4.6` does not exist.  

#### Step 3: Fix the Deployment  

Change the Deployment's image to `nginx`:  

```sh
kubectl set image deployment server-deployment grandserver=nginx
```

#### Step 4: Inspect Rollout History  

```sh
kubectl rollout history deployments server-deployment
```

There should be **two revisions**:  
1. Initial Deployment with `grand-server:1.4.6` (failed).  
2. Updated Deployment with `nginx`.  

---

### 5. Create and Configure a CronJob  

#### Step 1: Create a CronJob  

Create a CronJob named `google-ping` that runs a `curl` command every **2 minutes** using the `nginx` image:  

```sh
kubectl create cronjob google-ping --schedule="*/2 * * * *" \
--image=nginx -- /bin/sh -c 'curl google.com'
```

#### Step 2: Monitor CronJob Executions  

```sh
kubectl get cronjob -w
```

#### Step 3: Retain Execution History  

Modify the CronJob to retain the last **7 successful executions**:  

```sh
kubectl patch cronjob google-ping --type='merge' -p \
'{"spec": {"successfulJobsHistoryLimit": 7}}'
```

#### Step 4: Prevent Concurrent Executions  

Modify the CronJob to **forbid** a new execution if the current execution is still running:  

```sh
kubectl patch cronjob google-ping --type='merge' -p \
'{"spec": {"concurrencyPolicy": "Forbid"}}'
```

---

## Summary  

- **Created and labeled Pods** for frontend and backend services.  
- **Annotated Pods** and queried Pods using labels.  
- **Deployed a failing Deployment**, debugged it, and fixed the issue.  
- **Created and configured a CronJob** with execution history and concurrency limits.  

This lab covers **core Kubernetes resource management**, **troubleshooting**, and **batch job scheduling**. 🚀
