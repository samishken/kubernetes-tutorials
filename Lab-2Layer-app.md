### **Lab: Two-Layer Application with Flask and Persistent Storage for Backend**

In this lab, we will create a two-layer application using Flask (Frontend and Backend), where the backend application stores data on a **Persistent Volume** (PVC) to ensure data persistence even if the backend pod is killed or recreated.

---

### **Objective:**
In this lab, you will:
1. Create a **frontend Flask app** that communicates with a **backend Flask app**.
2. Use **Persistent Volumes (PVC)** to ensure the data written by the backend persists, even after the backend pod is deleted.
3. Deploy both the Flask apps to Kubernetes with services for communication between frontend and backend.

---

### **Pre-requisites:**
1. **Kubernetes Cluster** (can be a local cluster like Minikube or a cloud-based one).
2. **kubectl** installed and configured to interact with your Kubernetes cluster.
3. **Docker** installed on your machine to build and push the Docker images of the Flask apps.
4. **Docker Hub** account to push the Docker images.

---

### **Lab Steps:**

#### **Part 1: Create the Flask Backend Application with Persistent Storage**

1. **Create a directory for the Flask Backend app**:
   - On your local machine, create a directory for the backend Flask app.

   ```bash
   mkdir flask-backend
   cd flask-backend
   ```

2. **Create the Flask Backend Application (`app.py`)**:
   - Inside the `flask-backend` directory, create a Python file called `app.py`.

   **app.py:**

   ```python
   from flask import Flask, jsonify, request
   import os

   app = Flask(__name__)

   # Define the file path for the persistent data
   data_file_path = "/data/persistent_data.txt"

   @app.route('/data', methods=['GET'])
   def get_data():
       try:
           with open(data_file_path, 'r') as file:
               data = file.read()
       except FileNotFoundError:
           data = "No data found."
       return jsonify({"message": data})

   @app.route('/data', methods=['POST'])
   def save_data():
       new_data = request.json.get("message", "No data provided")
       with open(data_file_path, 'w') as file:
           file.write(new_data)
       return jsonify({"message": "Data saved successfully!"}), 200

   if __name__ == "__main__":
       app.run(host='0.0.0.0', port=5001)
   ```

   - This Flask backend app exposes two routes:
     - `GET /data`: Fetches the current data saved in a file located in a persistent volume.
     - `POST /data`: Saves data into the persistent volume.

3. **Create a `requirements.txt` file**:
   - Create a `requirements.txt` file to list the dependencies for the Flask backend application.

   **requirements.txt:**

   ```
   Flask
   ```

4. **Dockerize the Backend Flask Application**:
   - Create a `Dockerfile` to containerize the backend Flask app.

   **Dockerfile:**

   ```Dockerfile
   # Use an official Python runtime as a parent image
   FROM python:3.8-slim

   # Set the working directory in the container
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   COPY . /app

   # Install any needed packages specified in requirements.txt
   RUN pip install --no-cache-dir -r requirements.txt

   # Expose port 5001 to the outside world
   EXPOSE 5001

   # Run app.py when the container launches
   CMD ["python", "app.py"]
   ```

5. **Build and Push the Docker Image for the Backend Flask App**:
   - Build and push the Docker image for the backend Flask app.

   ```bash
   docker build -t <your-dockerhub-username>/flask-backend .
   docker push <your-dockerhub-username>/flask-backend
   ```

---

#### **Part 2: Create the Flask Frontend Application**

1. **Create a directory for the Flask Frontend app**:
   - Create another directory for the frontend Flask app.

   ```bash
   mkdir flask-frontend
   cd flask-frontend
   ```

2. **Create the Flask Frontend Application (`app.py`)**:
   - Inside the `flask-frontend` directory, create a Python file called `app.py`.

   **app.py:**

   ```python
   from flask import Flask, jsonify
   import requests

   app = Flask(__name__)

   @app.route('/')
   def index():
       # Fetch data from the backend Flask app
       backend_url = "http://flask-backend-service:5001/data"
       response = requests.get(backend_url)
       data = response.json()
       return jsonify({
           "frontend_message": "Frontend is calling the backend!",
           "backend_response": data
       })

   if __name__ == "__main__":
       app.run(host='0.0.0.0', port=5000)
   ```

   - This Flask frontend app calls the backend Flask app via an HTTP GET request and displays the data it receives.

3. **Create a `requirements.txt` file**:
   - Create a `requirements.txt` file for the Flask frontend app.

   **requirements.txt:**

   ```
   Flask
   requests
   ```

4. **Dockerize the Frontend Flask Application**:
   - Create a `Dockerfile` to containerize the frontend Flask app.

   **Dockerfile:**

   ```Dockerfile
   # Use an official Python runtime as a parent image
   FROM python:3.8-slim

   # Set the working directory in the container
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   COPY . /app

   # Install any needed packages specified in requirements.txt
   RUN pip install --no-cache-dir -r requirements.txt

   # Expose port 5000 to the outside world
   EXPOSE 5000

   # Run app.py when the container launches
   CMD ["python", "app.py"]
   ```

5. **Build and Push the Docker Image for the Frontend Flask App**:
   - Build and push the Docker image for the frontend Flask app.

   ```bash
   docker build -t <your-dockerhub-username>/flask-frontend .
   docker push <your-dockerhub-username>/flask-frontend
   ```

---

#### **Part 3: Create Persistent Volume and Deploy Both Applications**

1. **Create a Persistent Volume (PV) and Persistent Volume Claim (PVC)**:
   - Create a PV to store data on your cluster, and a PVC to request storage for the backend application.

   **persistent-volume.yaml**:

   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: flask-backend-pv
   spec:
     capacity:
       storage: 1Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: /mnt/data/flask-backend # Path on the worker node
   ```

   **persistent-volume-claim.yaml**:

   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: flask-backend-pvc
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```

2. **Create the Kubernetes Deployment for the Backend Flask App**:
   - Use the PVC for the backend app to store data persistently.

   **backend-deployment.yaml**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: flask-backend
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: flask-backend
     template:
       metadata:
         labels:
           app: flask-backend
       spec:
         containers:
         - name: flask-backend
           image: <your-dockerhub-username>/flask-backend
           ports:
           - containerPort: 5001
           volumeMounts:
           - name: flask-backend-storage
             mountPath: /data
         volumes:
         - name: flask-backend-storage
           persistentVolumeClaim:
             claimName: flask-backend-pvc
   ```

3. **Create the Kubernetes Service for the Backend Flask App**:

   **backend-service.yaml**:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-backend-service
   spec:
     selector:
       app: flask-backend
     ports:
       - protocol: TCP
         port: 5001
         targetPort: 5001
     clusterIP: None
   ```

4. **Create the Kubernetes Deployment for the Frontend Flask App**:

   **frontend-deployment.yaml**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: flask-frontend
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: flask-frontend
     template:
       metadata:
         labels:
           app: flask-frontend
       spec:
         containers:
         - name: flask-frontend
           image: <your-dockerhub-username>/flask-frontend
           ports:
           - containerPort: 5000
   ```

5. **Create the Kubernetes Service for the Frontend Flask App**:

   **frontend-service.yaml**:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-frontend-service
   spec:
     selector:
       app: flask-frontend
     ports:
       - protocol: TCP
         port: 80
         targetPort: 5000
     type: LoadBalancer
   ```

---

#### **Part 4: Apply the Deployments and Verify**

1. **Apply the Kubernetes Resources**:

   ```bash
   kubectl apply -f persistent-volume.yaml
   kubectl apply -f persistent-volume-claim.yaml
   kubectl apply -f backend-deployment.yaml
   kubectl apply -f backend-service.yaml
   kubectl apply -f frontend-deployment.yaml
   kubectl apply -f frontend-service.yaml
   ```

2. **Check the Deployments and Services**:

   ```bash
   kubectl get pods
   kubectl get svc
   ```

3. **Test the Application**:
   - Use `kubectl port-forward` to access the frontend service if you are not using LoadBalancer:

   ```bash
   kubectl port-forward svc/flask-frontend-service 8080:80
   ```

   Visit [http://localhost:8080](http://localhost:8080) to see the frontend app. It will display data fetched from the backend app.

---

#### **Part 5: Verify Data Persistence**

1. **Post Data to the Backend**:
   - Use a tool like `curl` or Postman to send a `POST` request to the backend app to save some data.

   ```bash
   curl -X POST http://<backend-ip>:5001/data -H "Content-Type: application/json" -d '{"message": "Hello, Kubernetes!"}'
   ```

2. **Delete the Backend Pod**:
   - Simulate a failure by deleting the backend pod.

   ```bash
   kubectl delete pod <backend-pod-name>
   ```

3. **Check if Data is Persisted**:
   - After the backend pod is recreated, check if the data is still there by making a `GET` request to the backend app:

   ```bash
   curl http://<backend-ip>:5001/data
   ```

   You should see that the data is still present and persisted.

---

### **Deliverables:**

1. **Docker Images**: Submit the Docker image names (from Docker Hub) used for both Flask applications.
2. **Kubernetes YAML Files**:
   - `persistent-volume.yaml`
   - `persistent-volume-claim.yaml`
   - `backend-deployment.yaml`
   - `backend-service.yaml`
   - `frontend-deployment.yaml`
   - `frontend-service.yaml`
3. **Testing Results**:
   - Provide evidence that data is being persisted even after deleting and recreating the backend pod.

---

### **Assessment Criteria:**
- **Data Persistence**: The backend app should correctly store and persist data even if the pod is killed or recreated.
- **Service Communication**: The frontend Flask app should successfully call the backend Flask app and display the data.
- **Kubernetes Resources**: Correct use of persistent volumes (PVCs) and proper configuration of Kubernetes deployments and services.

