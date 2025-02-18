### **Lab: Using Environment Variables in Kubernetes with a Python Flask Application**

---

### **Objective:**
In this lab, you will learn how to:
1. Create a simple Python Flask application.
2. Use environment variables to configure the application.
3. Deploy the Flask application on Kubernetes and pass environment variables to the app.
4. Verify that the Flask app is correctly configured with the environment variables and can be accessed externally.

---

### **Pre-requisites:**
1. **Kubernetes Cluster** (can be a local cluster like Minikube or a cloud-based one).
2. **kubectl** installed and configured to interact with your Kubernetes cluster.
3. **Docker** installed on your machine to build and push the Docker image of the Flask app.
4. **Docker Hub** account to push the Docker image.

---

### **Lab Steps:**

#### **Part 1: Create the Flask Application**

1. **Create a directory for the Flask app**:
   - On your local machine, create a directory to hold your Flask application.

   ```bash
   mkdir flask-k8s-app
   cd flask-k8s-app
   ```

2. **Create the Flask Application (`app.py`)**:
   - Inside the `flask-k8s-app` directory, create a Python file called `app.py`.

   **app.py:**

   ```python
   from flask import Flask
   import os

   app = Flask(__name__)

   # Get the environment variable
   env_var = os.getenv('MY_ENV_VAR', 'Default Value')

   @app.route('/')
   def hello():
       return f'Hello from Flask! The environment variable is: {env_var}'

   if __name__ == "__main__":
       app.run(host='0.0.0.0', port=5000)
   ```

   This simple Flask app will display the value of an environment variable named `MY_ENV_VAR`. If the environment variable is not set, it will display `"Default Value"`.

3. **Create a `requirements.txt` file**:
   - Create a `requirements.txt` file to list the dependencies for the Flask application.

   **requirements.txt:**

   ```
   Flask
   ```

#### **Part 2: Dockerize the Flask Application**

1. **Create a Dockerfile**:
   - In the same directory, create a file called `Dockerfile` to containerize the Flask app.

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

2. **Build the Docker image**:
   - Build your Docker image using the following command:

   ```bash
   docker build -t <your-dockerhub-username>/flask-app-env .
   ```

3. **Push the Docker image to Docker Hub**:
   - After building the image, push it to Docker Hub:

   ```bash
   docker login
   docker push <your-dockerhub-username>/flask-app-env
   ```

#### **Part 3: Set Up Kubernetes Resources**

1. **Create a Deployment YAML file for the Flask app (`flask-deployment.yaml`)**:
   - The following YAML file will create a deployment for the Flask app and set an environment variable (`MY_ENV_VAR`) for the container.

   **flask-deployment.yaml**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: flask-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: flask-app
     template:
       metadata:
         labels:
           app: flask-app
       spec:
         containers:
         - name: flask-app
           image: <your-dockerhub-username>/flask-app-env
           ports:
           - containerPort: 5000
           env:
           - name: MY_ENV_VAR
             value: "This is the environment variable!"
   ```

   In this example, the `MY_ENV_VAR` environment variable is passed to the Flask application with the value `"This is the environment variable!"`.

2. **Create a Service YAML file to expose the Flask app (`flask-service.yaml`)**:
   - Create a service to expose the Flask app so you can access it externally.

   **flask-service.yaml**:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-app-service
   spec:
     selector:
       app: flask-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 5000
     type: LoadBalancer
   ```

3. **Apply the Deployment and Service**:
   - Apply both the deployment and service to Kubernetes.

   ```bash
   kubectl apply -f flask-deployment.yaml
   kubectl apply -f flask-service.yaml
   ```

#### **Part 4: Verify the Deployment**

1. **Check the Deployment and Pods**:
   - Verify that the Flask app is deployed correctly and that the pods are running.

   ```bash
   kubectl get pods
   kubectl get deployments
   ```

2. **Verify the Service**:
   - Check if the service has been created successfully.

   ```bash
   kubectl get svc
   ```

   If you're using a **LoadBalancer** type service (on a cloud provider), it will take a few minutes to get an external IP address. Otherwise, you can use `kubectl port-forward` to access the service locally:

   ```bash
   kubectl port-forward svc/flask-app-service 8080:80
   ```

   Then, access the Flask app by visiting [http://localhost:8080](http://localhost:8080).

3. **Verify Environment Variable**:
   - Visit the Flask application in your browser or use `curl` to make sure the environment variable is being correctly passed to the application:

   ```bash
   curl http://localhost:8080
   ```

   You should see something like this:

   ```
   Hello from Flask! The environment variable is: This is the environment variable!
   ```

#### **Part 5: Modify the Environment Variable and Redeploy**

1. **Modify the Environment Variable in the Deployment**:
   - Update the `MY_ENV_VAR` in the `flask-deployment.yaml` file to a new value, for example `"Updated value from Kubernetes!"`.

2. **Apply the Updated Deployment**:
   - After making the change, apply the updated deployment to Kubernetes:

   ```bash
   kubectl apply -f flask-deployment.yaml
   ```

3. **Verify the Update**:
   - Check the logs to ensure that the application is using the new environment variable:

   ```bash
   kubectl logs <pod-name>
   ```

   Alternatively, you can test the Flask app again by visiting [http://localhost:8080](http://localhost:8080) to verify the new value is reflected.

---

### **Deliverables:**

1. **Docker Image**: Submit the Docker image name (from Docker Hub) that you used for the Flask application.
2. **Kubernetes YAML Files**:
   - `flask-deployment.yaml`
   - `flask-service.yaml`
3. **Testing Results**:
   - Provide a screenshot or output from accessing the Flask app, showing that the environment variable is being passed and used.
4. **Log Outputs**:
   - Provide logs or evidence showing the environment variable update worked after the redeployment.

---

### **Assessment Criteria:**
- **Dockerization**: Correctly building and pushing the Flask application Docker image to Docker Hub.
- **Environment Variables**: Correctly passing and using environment variables inside the Flask app.
- **Kubernetes Deployment**: Properly creating and deploying the Flask app using Kubernetes resources (Deployment, Service).
- **Application Access**: Successfully accessing the Flask application via the exposed service and verifying the environment variable is used.

---

By completing this lab, you will understand how to use environment variables in Kubernetes to configure applications dynamically and securely.
