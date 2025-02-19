

# Kubernetes Scheduling Assignment

## Overview

In this assignment, you will deploy two applications (NGINX and Flask) with proper labeling, and then use pod affinity to ensure that a BusyBox container is scheduled on the same node as the Flask application.

## Objectives

1. **Deploy NGINX and Flask Applications:**
   - Create two separate Kubernetes Deployments:
     - One for an NGINX application.
     - One for a Flask application.
   - Use appropriate labels on each deployment (e.g., `app: nginx` for NGINX and `app: flask` for Flask).
   - Ensure that both applications are deployed on the desired nodes (you may use node affinity or ensure proper node selection via labels if needed).

2. **Deploy BusyBox with Pod Affinity:**
   - Create a deployment for a BusyBox container.
   - Use pod affinity rules so that the BusyBox container is scheduled on the same node(s) where the Flask application is deployed.

## Tasks

### Task 1: Deploy NGINX and Flask Applications

1. **NGINX Deployment:**
   - Create a YAML manifest (`nginx-deployment.yaml`) for an NGINX Deployment.
   - Add labels such as:
     ```yaml
     labels:
       app: nginx
       tier: web
     ```
   - Optionally, use node affinity rules if you want to ensure that NGINX runs on a specific set of nodes.

2. **Flask Deployment:**
   - Create a YAML manifest (`flask-deployment.yaml`) for a Flask Deployment.
   - Add labels such as:
     ```yaml
     labels:
       app: flask
       tier: web
     ```
   - Configure the container to use environment variables for configuration as needed.

### Task 2: Deploy BusyBox with Pod Affinity to Flask

1. **BusyBox Deployment:**
   - Create a YAML manifest (`busybox-deployment.yaml`) for a BusyBox Deployment.
   - Use pod affinity rules in the manifest so that the BusyBox pod is scheduled on nodes where the Flask pod is running.
   - For example, in the affinity section, you can specify that the pod should prefer or require to be co-located with pods labeled `app: flask`.

## Deliverables

- **YAML Manifests:**
  - `nginx-deployment.yaml`
  - `flask-deployment.yaml`
  - `busybox-deployment.yaml`

- **README File:**
  - Include instructions on how to deploy the applications.
  - Explain your labeling strategy and how your affinity rules ensure the BusyBox pod is co-located with the Flask pod.
  - Provide verification steps (e.g., using `kubectl get pods -o wide` and `kubectl describe pod <pod-name>`).

## Verification

- Deploy your resources with `kubectl apply -f <manifest.yaml>` commands.
- Use the following commands to verify your deployment:
  - List pods along with their labels and nodes:
    ```bash
    kubectl get pods -o wide --show-labels
    ```
  - Describe the BusyBox pod to check its affinity details:
    ```bash
    kubectl describe pod <busybox-pod-name>
    ```
- Ensure that the BusyBox pod is scheduled on the same node as at least one Flask pod.

