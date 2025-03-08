# **Kubernetes Bootcamp - Pre-Attendance Checklist**

## **1. Technical Knowledge Requirements**
To ensure a smooth learning experience, participants should have:
- **Basic Linux command-line skills**: Understanding of commands like `cd`, `ls`, `grep`, `curl`.
- **Fundamentals of Containers & Docker**:
  - Running containers: `docker run`
  - Viewing running containers: `docker ps`
  - Stopping containers: `docker stop`
- **Networking Concepts**:
  - Basic understanding of **IP addresses, ports, DNS, and firewalls**.

## **2. Software & Tools Setup**
Before attending the bootcamp, install and configure the following:


### **Mandatory Tools:**
✅ **Kubernetes CLI (`kubectl`)** – For managing Kubernetes clusters. Install using:
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

✅ **A Kubernetes Cluster** (choose one):
- **Minikube** (for local development):
  ```sh
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  sudo install minikube-linux-amd64 /usr/local/bin/minikube
  ```
- **Kind (Kubernetes in Docker)** (alternative for local clusters):
  ```sh
  curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
  chmod +x ./kind
  mv ./kind /usr/local/bin/kind
  ```
  Validate the Setup

```
#kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```
Ensure kubectl is working properly for kind cluster

```
kubectl get po -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-668d6bf9bc-ncsfn                     1/1     Running   0          2m30s
kube-system          coredns-668d6bf9bc-z9c6f                     1/1     Running   0          2m30s
kube-system          etcd-kind-control-plane                      1/1     Running   0          2m37s
kube-system          kindnet-z7xs9                                1/1     Running   0          2m30s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          2m37s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          2m37s
kube-system          kube-proxy-6rg82                             1/1     Running   0          2m30s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          2m37s
local-path-storage   local-path-provisioner-7dc846544d-ccs86      1/1     Running   0          2m30s
```
  
- **Cloud Kubernetes Cluster** (AWS EKS, GKE, AKS, or DigitalOcean Kubernetes). If using a cloud provider, ensure you have an active account.

✅ **Docker** (For container creation and deployment):
- Install via official Docker instructions: [Docker Installation](https://docs.docker.com/get-docker/)

✅ **YAML Basics** – Kubernetes uses YAML files for configurations. Learn to read and write basic YAML files.

### **Optional but Recommended Tools:**
☑ **Helm** – Kubernetes package manager:
```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

☑ **Git & CI/CD Basics**
- **GitHub/GitLab account** (for storing Kubernetes manifests).
- Install Git:
  ```sh
  sudo apt install git -y  # Ubuntu
  brew install git  # macOS
  ```
- Basic commands: `git clone`, `git commit`, `git push`.

## **3. Cloud Account (For Cloud-Based Clusters)**
If you plan to use a managed Kubernetes cluster (e.g., AWS EKS, GKE, or AKS), ensure you have:
- A cloud provider account (AWS, GCP, Azure, DigitalOcean, etc.).
- Configured CLI tools (e.g., `awscli`, `gcloud`, `az`).

## **4. Test Your Environment**
Run the following commands to verify setup:
```sh
kubectl version --client
minikube version  # If using Minikube
docker --version
git --version
```
If any command fails, troubleshoot before the bootcamp.

---

## **Final Checklist Before Attending**
✅ Installed `kubectl`, Docker, and Kubernetes cluster ✅
✅ Have a cloud provider account (if applicable) ✅
✅ Familiar with YAML and Kubernetes concepts ✅
✅ Tested commands and verified installations ✅

### **Need Help?**
If you face issues setting up your environment, reach out to the organizers before the bootcamp!

🚀 Get ready for an exciting Kubernetes learning experience!

