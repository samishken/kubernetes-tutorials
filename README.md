
### Prerequisites: 

- Familiarity with Docker and basic Linux commands.
- Familiarity with Git/GitHub
- Visual Studio Code
- Comfortable with Command line interface ( Windows/Linux/Mac )
- Tools: Minikube/Kind, kubectl, Helm

  
**Learning Outcomes:**
- Understand Kubernetes architecture and key concepts.
- Deploy and manage containerized applications.
- Use advanced features like Ingress, Persistent Volumes, and StatefulSets.
- Implement basic security and monitoring practices.

## **Kubernetes Foundations**

### **Session 1: Introduction to Kubernetes**
- **Topics Covered:**
  - What is Kubernetes and why use it?
  - Core Kubernetes components: Nodes, Pods, Clusters
  - The Kubernetes architecture: API server, etcd, kubelet, controller manager, scheduler
  - Overview of declarative vs. imperative configurations
- **Hands-On Lab:**  
  - Set up kubectl with Cloud based Lab cluster 

---

### **Session 2: Kubernetes Building Blocks**
- **Topics Covered:**
  - Pods, ReplicaSets, Deployments
  - Namespaces and labels/selectors
  - Services: ClusterIP, NodePort, LoadBalancer
- **Hands-On Lab:**  
  - Create and manage a Pod using YAML manifests.
  - Scale an application using ReplicaSets.
  - Expose an application with a Service.

---

### **Session 3: Networking in Kubernetes**
- **Topics Covered:**
  - Kubernetes networking basics
  - CNI plugins (e.g., Calico, Flannel)
  - Service discovery and DNS
  - Ingress and egress traffic
- **Hands-On Lab:**  
  - Deploy an NGINX application with an Ingress resource for routing.

---

### **Session 4: Storage in Kubernetes**
- **Topics Covered:**
  - Persistent Volumes (PV) and Persistent Volume Claims (PVC)
  - StorageClasses
  - Dynamic provisioning
  - ConfigMaps and Secrets
- **Hands-On Lab:**  
  - Deploy an application using PVC for storage.
  - Use ConfigMaps and Secrets to manage environment variables.

---

### ** Foundation Wrap-Up**
- Q&A and recap of foundational concepts.
- Assign homework: Create a Deployment and expose it using a LoadBalancer service.

---

## ** Advanced Kubernetes Concepts**

### **Session 1: Managing Workloads**
- **Topics Covered:**
  - StatefulSets and DaemonSets
  - Jobs and CronJobs
  - Horizontal Pod Autoscaling (HPA)
- **Hands-On Lab:**  
  - Deploy a database using StatefulSet.
  - Create a CronJob to run periodic tasks.

---

### **Session 2: Security in Kubernetes**
- **Topics Covered:**
  - Role-Based Access Control (RBAC)
  - Network Policies
  - Securing Secrets
  - Pod security policies
- **Hands-On Lab:**  
  - Create RBAC roles and bind them to users.
  - Implement a NetworkPolicy to restrict pod communication.

---

### **Session 3: Monitoring and Logging**
- **Topics Covered:**
  - Overview of monitoring tools (Prometheus, Grafana)
  - Logging with Fluentd, ELK, or Loki
  - Using `kubectl logs` and `kubectl top`
- **Hands-On Lab:**  
  - Deploy a sample Prometheus and Grafana stack.
  - Visualize metrics and logs for an application.

---

### **Session 4: CI/CD and Kubernetes**
- **Topics Covered:**
  - Kubernetes in a CI/CD pipeline
  - Tools: Helm, ArgoCD, Flux
  - Deploying applications with Helm
- **Hands-On Lab:**  
  - Package and deploy an application using Helm.
  - Implement a basic GitOps pipeline with ArgoCD.

---

### **Final Wrap-Up**
- Recap of advanced concepts.
- Discussion on Kubernetes best practices and real-world use cases.
- Resources for further learning (certifications, documentation, etc.)
- Q&A session.

---

## **Additional Notes**
- **Prerequisites:** Familiarity with Docker and basic Linux commands.
- **Tools:** Minikube/Kind, kubectl, Helm, Prometheus, Grafana, ArgoCD.
- **Learning Outcomes:**  
  - Understand Kubernetes architecture and key concepts.
  - Deploy and manage containerized applications.
  - Use advanced features like Ingress, Persistent Volumes, and StatefulSets.
  - Implement basic security and monitoring practices.

