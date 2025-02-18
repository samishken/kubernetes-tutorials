### **Assignment: Building and Deploying a Flask Application with MySQL on Kubernetes**

---

### **Objective:**
In this assignment, you will learn how to:
1. Build a Docker container for a simple Flask application.
2. Push the container to Docker Hub.
3. Set up a MySQL database.
4. Expose the Flask app using a LoadBalancer Service in Kubernetes.
5. Connect the Flask app to the MySQL backend, ensuring it can read and write data.

---

### **Instructions:**

1. **Set up a Kubernetes cluster** (e.g., Minikube, K3s, or a cloud provider like GKE, EKS, or AKS).
2. **Ensure that Docker** is installed and running on your local machine.
3. **Ensure that you have a Docker Hub account** and can push images to your repository.

Follow the steps below to complete the assignment.

---

### **Tasks:**

#### **Part 1: Create a Flask Application**

1. **Create a Flask Application**:
   - Write a simple Flask application that connects to a MySQL database. The app should have one route that interacts with the MySQL database by adding and retrieving data.
   
   **Python Code (`app.py`):**

   ```python
   from flask import Flask, request, jsonify
   import mysql.connector
   import os

   app = Flask(__name__)

   # MySQL connection settings
   db_config = {
       'host': os.getenv('MYSQL_HOST', 'localhost'),
       'user': os.getenv('MYSQL_USER', 'root'),
       'password': os.getenv('MYSQL_PASSWORD', 'password'),
       'database': os.getenv('MYSQL_DB', 'flaskdb')
   }

   # Establish MySQL connection
   def get_db_connection():
       return mysql.connector.connect(**db_config)

   @app.route('/')
   def index():
       return "Flask App connected to MySQL!"

   @app.route('/add', methods=['POST'])
   def add_data():
       data = request.json
       conn = get_db_connection()
       cursor = conn.cursor()

       # Insert data into MySQL
       cursor.execute('INSERT INTO users (name, age) VALUES (%s, %s)', (data['name'], data['age']))
       conn.commit()

       cursor.close()
       conn.close()
       return jsonify({"message": "Data added successfully!"}), 201

   @app.route('/users', methods=['GET'])
   def get_users():
       conn = get_db_connection()
       cursor = conn.cursor()

       cursor.execute('SELECT * FROM users')
       users = cursor.fetchall()

       cursor.close()
       conn.close()
       return jsonify(users)

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000, debug=True)
   ```

2. **Create a MySQL initialization script (`init.sql`)**:
   - This script will initialize the database and create a table called `users`.

   **MySQL Initialization Script (`init.sql`):**

   ```sql
   CREATE DATABASE IF NOT EXISTS flaskdb;

   USE flaskdb;

   CREATE TABLE IF NOT EXISTS users (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100),
       age INT
   );
   ```

#### **Part 2: Dockerize the Flask Application**

1. **Create a Dockerfile**:
   - Create a `Dockerfile` to containerize the Flask app.

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

   # Define environment variables for MySQL
   ENV MYSQL_HOST=mysql
   ENV MYSQL_USER=root
   ENV MYSQL_PASSWORD=password
   ENV MYSQL_DB=flaskdb

   # Run app.py when the container launches
   CMD ["python", "app.py"]
   ```

2. **Create a `requirements.txt`**:
   - List the Python dependencies for the Flask application.

   **requirements.txt:**

   ```
   Flask==2.0.1
   mysql-connector-python==8.0.26
   ```

3. **Build the Docker Image**:
   - Build your Docker image using the following command:

   ```bash
   docker build -t <your-dockerhub-username>/flask-app .
   ```

4. **Push the Docker Image to Docker Hub**:
   - Push the Docker image to Docker Hub:

   ```bash
   docker login
   docker push <your-dockerhub-username>/flask-app
   ```

#### **Part 3: Set Up MySQL on Kubernetes**

1. **Create a MySQL Deployment and Service**:
   - Create a MySQL deployment (`mysql-deployment.yaml`) and a service to expose it within the Kubernetes cluster.

   **MySQL Deployment and Service (`mysql-deployment.yaml`):**

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mysql
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: mysql
     template:
       metadata:
         labels:
           app: mysql
       spec:
         containers:
         - name: mysql
           image: mysql:5.7
           env:
           - name: MYSQL_ROOT_PASSWORD
             value: password
           volumeMounts:
           - name: mysql-data
             mountPath: /var/lib/mysql
           ports:
           - containerPort: 3306
     volumes:
     - name: mysql-data
       emptyDir: {}

   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql
   spec:
     selector:
       app: mysql
     ports:
       - protocol: TCP
         port: 3306
         targetPort: 3306
     clusterIP: None
   ```

2. **Deploy MySQL to Kubernetes**:
   - Deploy MySQL to your Kubernetes cluster:

   ```bash
   kubectl apply -f mysql-deployment.yaml
   ```

3. **Run MySQL Initialization Script**:
   - Run the MySQL initialization script to create the database and table:

   ```bash
   kubectl exec -it <mysql-pod-name> -- mysql -u root -p < init.sql
   ```

   Replace `<mysql-pod-name>` with the actual pod name of your MySQL container.

#### **Part 4: Deploy Flask App to Kubernetes**

1. **Create a Kubernetes Deployment and Service for the Flask App**:
   - Create a deployment (`flask-app-deployment.yaml`) to deploy your Flask application to Kubernetes.
   - Also, create a service to expose the Flask application using a LoadBalancer.

   **Flask Deployment and Service (`flask-app-deployment.yaml`):**

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
           image: <your-dockerhub-username>/flask-app
           ports:
           - containerPort: 5000
           env:
           - name: MYSQL_HOST
             value: mysql
           - name: MYSQL_USER
             value: root
           - name: MYSQL_PASSWORD
             value: password
           - name: MYSQL_DB
             value: flaskdb

   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-app
   spec:
     selector:
       app: flask-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 5000
     type: LoadBalancer
   ```

2. **Deploy Flask App to Kubernetes**:
   - Deploy your Flask application to Kubernetes:

   ```bash
   kubectl apply -f flask-app-deployment.yaml
   ```

3. **Verify the Deployment**:
   - Verify that the Flask app and MySQL services are running:

   ```bash
   kubectl get pods
   kubectl get services
   ```

#### **Part 5: Testing the Application**

1. **Test MySQL Connection**:
   - Use `kubectl port-forward` to connect to the Flask application via the LoadBalancer service:
   
   ```bash
   kubectl port-forward svc/flask-app 8080:80
   ```

   - Visit `http://localhost:8080` to test if the Flask application is running and connected to MySQL.

2. **Test Flask Endpoints**:
   - Use `curl` or Postman to test the following:
     - **POST /add**: Add a user (e.g., `{"name": "John", "age": 30}`).
     - **GET /users**: Retrieve all users from the MySQL database.

---

### **Deliverables:**

1. **Docker Image**: Push the Flask Docker image to Docker Hub.
2. **Kubernetes YAML Files**:
   - `mysql-deployment.yaml`
   - `flask-app-deployment.yaml`
3. **Commands**:
   - Output of `kubectl get pods` and `kubectl get services`.
4. **Testing Results**:
   - Provide screenshots or outputs of testing the `/add` and `/users` endpoints to confirm the Flask app is working and connected to the MySQL database.

---

### **Assessment Criteria:**
- Correct Dockerization and pushing of the Flask app to Docker Hub.
- Successful deployment and connection of Flask to MySQL in Kubernetes.
- Proper creation and use of Kubernetes services for Load Balancing.
- Verification of Flask app functionality by interacting with MySQL database.

