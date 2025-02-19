
# Kubernetes & Docker Assignment: Flask & MySQL Deployment


## Overview

In this assignment, you are required to build a containerized application that consists of a Flask web application and a MySQL database. The two components will be deployed on a public cloud Kubernetes cluster in separate namespaces with proper configuration management using ConfigMaps and Secrets.

You are provided with:
- **A Python Flask application script (`app.py`)**
- **A MySQL initialization script (`initdb.sql`)**
- **An architecture diagram (see below)**

Your task is to containerize the Flask application, build and push its Docker image to Docker Hub, and deploy both the Flask app and MySQL database on Kubernetes following the guidelines described below.

---

## Provided Artifacts

### 1. Flask Application Script (`app.py`)

```python
from flask import Flask, jsonify
import os
import mysql.connector
from mysql.connector import Error

app = Flask(__name__)

def get_db_connection():
    """
    Establishes a connection to the MySQL database using environment variables.
    Expected environment variables:
      - MYSQL_HOST
      - MYSQL_DB
      - MYSQL_USER
      - MYSQL_PASSWORD
    """
    host = os.environ.get("MYSQL_HOST", "localhost")
    database = os.environ.get("MYSQL_DB", "flaskdb")
    user = os.environ.get("MYSQL_USER", "flaskuser")
    password = os.environ.get("MYSQL_PASSWORD", "flaskpass")
    
    try:
        connection = mysql.connector.connect(
            host=host,
            database=database,
            user=user,
            password=password
        )
        if connection.is_connected():
            return connection
    except Error as e:
        app.logger.error(f"Error connecting to MySQL: {e}")
    return None

@app.route("/")
def index():
    return f"Welcome to the Flask App running in {os.environ.get('APP_ENV', 'development')} mode!"

@app.route("/dbtest")
def db_test():
    """
    A simple endpoint to test the MySQL connection.
    Executes a query to get the current time from the database.
    """
    connection = get_db_connection()
    if connection is None:
        return jsonify({"error": "Failed to connect to MySQL database"}), 500
    try:
        cursor = connection.cursor()
        cursor.execute("SELECT NOW();")
        current_time = cursor.fetchone()
        return jsonify({
            "message": "Successfully connected to MySQL!",
            "current_time": current_time[0]
        })
    except Error as e:
        return jsonify({"error": str(e)}), 500
    finally:
        if connection and connection.is_connected():
            cursor.close()
            connection.close()

if __name__ == "__main__":
    debug_mode = os.environ.get("DEBUG", "false").lower() == "true"
    app.run(host="0.0.0.0", port=5000, debug=debug_mode)
```

---

### 2. MySQL Initialization Script (`initdb.sql`)

```sql
CREATE DATABASE IF NOT EXISTS flaskdb;
CREATE USER 'flaskuser'@'%' IDENTIFIED BY 'flaskpass';
GRANT ALL PRIVILEGES ON flaskdb.* TO 'flaskuser'@'%';
FLUSH PRIVILEGES;
```

---

### 3. Architecture Diagram

![Architecture Diagram](Python-MySQL-Project.png)


---

## Assignment Requirements

1. **Namespaces & Isolation:**
   - Create two separate Kubernetes namespaces: `flask-app` and `mysql`.

2. **Containerization & Docker:**
   - Create a Dockerfile for the Flask application that installs all dependencies and copies the provided `app.py`.
   - Include any additional packages required (e.g., `iputils-ping` for troubleshooting).
   - Build the Docker image and push it to Docker Hub (e.g., `becloudready/my-flask-app:latest`).

3. **Kubernetes Manifests:**
   - **ConfigMaps:**
     - Create a ConfigMap in the `flask-app` namespace for the Flask configuration (include environment variables like `APP_ENV`, `DEBUG`, `MYSQL_DB`, and `MYSQL_HOST`).
     - Create a ConfigMap in the `mysql` namespace that includes the provided `initdb.sql`.
   - **Secrets:**
     - Create a Secret in the `flask-app` namespace for the database credentials. (The correct values are: `MYSQL_USER: flaskuser`, `MYSQL_PASSWORD: flaskpass`, and `MYSQL_DB: flaskdb`.)
   - **MySQL Deployment:**
     - Deploy MySQL using a StatefulSet in the `mysql` namespace.
     - Use a PVC (via volumeClaimTemplates) for persistent storage.
     - Mount the initdb ConfigMap at `/docker-entrypoint-initdb.d` so that MySQL initializes correctly.
   - **Flask Deployment:**
     - Deploy the Flask application using a Deployment in the `flask-app` namespace.
     - Configure the Deployment to load environment variables from the ConfigMap and Secret.
   - **Services:**
     - Create a LoadBalancer Service in the `flask-app` namespace to expose the Flask application externally.
     - Create a ClusterIP Service for MySQL in the `mysql` namespace.

4. **Validation & Testing:**
   - Ensure that the Flask application can connect to the MySQL database using the provided credentials.
   - Verify connectivity by accessing the `/dbtest` endpoint on the Flask application.
   - Demonstrate that the database initialization script runs only once (i.e., on a fresh persistent volume).

5. **Documentation:**
   - Provide a README file that outlines:
     - How to build the Docker image.
     - How to push the image to Docker Hub.
     - Steps to deploy the Kubernetes manifests.
     - Any troubleshooting steps or considerations.

---

## Deliverables

- **Dockerfile** for the Flask application.
- **Kubernetes YAML manifests** for:
  - Namespaces
  - ConfigMaps (Flask and MySQL)
  - Secrets
  - MySQL StatefulSet and Service
  - Flask Deployment and Service
- **README file** with build and deployment instructions.
- **Source code** for the Flask application (`app.py`) and the MySQL init script (`initdb.sql`).

---

## Evaluation Criteria

- **Correctness:** The deployment must correctly initialize MySQL with the given init script and allow the Flask app to connect to it.
- **Kubernetes Best Practices:** Use of namespaces, ConfigMaps, Secrets, Deployments, StatefulSets, and Services appropriately.
- **Containerization:** Proper Dockerfile that builds a functioning image.
- **Documentation:** Clear instructions on how to build, push, and deploy the solution.
- **Troubleshooting:** Ability to verify and debug connectivity between the Flask app and MySQL.

---

Good luck! This assignment will test your ability to integrate Docker and Kubernetes in a real-world application scenario while adhering to best practices for container orchestration and configuration management.
