
**Lab Title:**  
Using a Pre-Made Helm Chart: Customization, Upgrade, Rollback, and Cleanup

**Objective:**  
Learn how to deploy a ready-made Helm chart, modify its configuration, upgrade the release, roll back to a previous version, and then uninstall the release—all using Helm v3.

**Install Helm**

Linux/MacOS

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

MacOS ( Homebrew )
```
brew install helm
```
Windows 
```
choco install kubernetes-helm
```

**Chart Used:**  
We will use the Bitnami NGINX chart available from the Bitnami repository.

---

**Step 1: Add the Bitnami Repository and Update**

1. Open your terminal.
2. Add the Bitnami Helm repository:
   
   ```
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```
   
3. Update your local repository cache:

   ```
   helm repo update
   ```
---

**Step 2: Install the Pre-Made NGINX Chart**

1. Install the NGINX chart with a release name (we’ll use “my-nginx”) in the default namespace:

   ```
   helm install my-nginx bitnami/nginx
   ```

2. Verify the release installation:
   
   - List Helm releases:
     
     ```
     helm list
     ```
     
   - Check the Kubernetes resources created:
     
     ```
     kubectl get all -l app.kubernetes.io/name=nginx
     ```

---

**Step 3: Customize the Chart**

1. Retrieve the default values for the chart and save them to a file:
   
   ```
   helm show values bitnami/nginx > my-nginx-values.yaml
   ```

2. Open the file “my-nginx-values.yaml” in your favorite text editor.
3. Make a simple customization. For example, change the number of replicas from 1 to 2 by modifying:
   
   ```
   replicaCount: 1
   ```
   
   to:
   
   ```
   replicaCount: 2
   ```

   You can also change other values, such as the service type (e.g., from ClusterIP to LoadBalancer) or image tag.
4. Save your changes.

---

**Step 4: Upgrade the Release with Your Custom Values**

1. Upgrade the “my-nginx” release using your modified values file:

   ```
   helm upgrade my-nginx bitnami/nginx -f my-nginx-values.yaml
   ```

2. Verify the upgrade:
   
   - Check the status of the release:
     
     ```
     helm status my-nginx
     ```
     
   - Use `kubectl` to confirm that the number of replicas has increased:
     
     ```
     kubectl get deployment my-nginx
     ```
     
   The Deployment should now show the updated replica count.

---

**Step 5: Roll Back the Release**

1. If you want to revert the changes and go back to the previous version, execute the rollback command:
   
   ```
   helm rollback my-nginx 1
   ```
   
   (The number “1” refers to the previous revision. You can view the revision history with `helm history my-nginx`.)
   
2. Verify that the rollback was successful:
   
   - Check the release status:
     
     ```
     helm status my-nginx
     ```
     
   - Confirm with `kubectl` that the Deployment now reflects the previous configuration (e.g., replica count returns to 1).

---

**Step 6: Clean Up**

1. Once you’ve completed the lab, uninstall the release to clean up resources:
   
   ```
   helm uninstall my-nginx
   ```
   
2. Verify that all related Kubernetes resources are removed:
   
   ```
   kubectl get all -l app.kubernetes.io/name=nginx
   ```

---
