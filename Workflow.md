# Deploying a Full-Stack Web App Using AWS EC2, AWS EKS, Docker & MariaDB  
*A Complete End-to-End Deployment Guide*

<img width="1000" height="559" alt="image" src="https://github.com/user-attachments/assets/ef46f04f-1425-4737-8c7b-7a6b8b962bc9" />

---

##  Overview
This documentation explains how to deploy a full-stack application using:

- **AWS EC2** (for management & backend testing)  
- **AWS EKS** (for Kubernetes deployment)  
- **Docker** (containerizing backend/frontend)  
- **MariaDB / AWS RDS** (database)

---

### 1️. Launch an EC2 Instance

**Launch an Ubuntu EC2 instance (C7.xlarge).**
Switch to root:

```bash
sudo -i
```
Update system packages:
```bash
apt update -y
```
### 2️ Connect EC2 With AWS EKS Cluster

Follow your full setup guide:

#### EKS + EC2 Setup Guide
```bash
https://github.com/OmkarMemane09/DevOps_by_OmkarMemane09/blob/main/Kubernetes/2.k8s-EKS-Setup-with-EC2.md
```

> This includes:

> Installing AWS CLI

> Installing kubectl

> Configuring kubeconfig

> Connecting EC2 → EKS cluster
---

*Validate EKS connection:*
```bash
kubectl get nodes
```
3️⃣ Install Docker on EC2
```bash
apt install docker.io -y
```
```bash
systemctl enable docker
```
```bash
systemctl start docker
```
```bash
docker --version
```
### 4️. Install SQL
```bash
apt install mysql-client
```
Login into MariaDB:

```bash
mysql -h <rds endpoint> -u root -p
```
```bash
CREATE DATABASE student_db;

GRANT ALL PRIVILEGES ON springbackend.* TO 'username'@'localhost' IDENTIFIED BY 'your_password';

USE student_db;

CREATE TABLE `students` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `course` varchar(255) DEFAULT NULL,
  `student_class` varchar(255) DEFAULT NULL,
  `percentage` double DEFAULT NULL,
  `branch` varchar(255) DEFAULT NULL,
  `mobile_number` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=80 DEFAULT CHARSET=latin1 COLLATE=latin1_swedish_ci;

exit;
```
### 5️. Clone Your Application Repository
```bash
git clone https://github.com/OmkarMemane09/EasyCRUD
```
```bash
cd EasyCRUD
```
### 6️. Update Backend Configuration (application.properties)

Navigate to backend folder:

```bash
cd EasyCRUD/backend
```
```bash
nano application.properties
```
Update the following values:

Replace localhost with your RDS Endpoint or DB container IP

Set correct DB username

Set correct DB password

Example:
```bash
spring.datasource.url=jdbc:mysql://<RDS-ENDPOINT>:3306/student_db
spring.datasource.username=username
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
```
Save and exit:
CTRL + O, ENTER, then CTRL + X

### 7️. Create Backend Dockerfile

Navigate to backend folder if not already there:

```bash
cd EasyCRUD/backend
```
Create Dockerfile:

```bash
nano dockerfile
```
Dockerfile content:

```bash
FROM maven:3.8.3-openjdk-17
COPY . /opt/
WORKDIR /opt
RUN rm -rf src/main/resources/application.properties
RUN cp application.properties src/main/resources/
RUN mvn clean package
WORKDIR /opt/target
EXPOSE 8080
CMD ["java","-jar","student-registration-backend-0.0.1-SNAPSHOT.jar"]
```
Save & exit:
CTRL + O, ENTER, CTRL + X

### 8️. Build Backend Docker Image
```bash
docker build . -t backend-app
```
### 9️. Run Backend Container (Local Test)
```bash
docker run -d -p 8080:8080 backend-app
```

### 1️0️. Tag Backend Image With Docker Hub Repository
```bash
docker tag backend-app <your-dockerhub-username>/backend-app:latest
```
### 1️1️. Log In to Docker Hub
```bash
docker login
```
Enter Docker Hub username & password when prompted.

### 1️2️. Push Backend Image to Docker Hub
```bash
docker push <your-dockerhub-username>/backend-app:latest
```
## 1️3️. Create Kubernetes Pod for Backend

```bash
nano pod.yaml
```
Paste the following content:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    app: backend
spec:
  containers:
    - name: backend-container
      image: omkarmemane09/backend:v1   # Replace with your Docker Hub backend image
      ports:
        - containerPort: 8080
```
Save & exit:
CTRL + O, ENTER, CTRL + X

**Apply the Pod manifest:**
```bash
kubectl apply -f pod.yaml
```
Check Pod status:
```bash
kubectl get pods
```
### 1️4️. Create Backend Service (NodePort)
Create the service file:

```bash
nano svc.yaml
```
Paste the following:

```yaml

apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: LoadBalancer
```
Save & exit:
CTRL + O, ENTER, CTRL + X

**Apply the service manifest:**

```bash
kubectl apply -f svc.yaml
```
### 1️5️. Verify Deployment

Check nodes:
```bash
kubectl get nodes
```
Check services:

```bash
kubectl get svc
```
Check pod details:
```bash
kubectl describe pod backend-pod
```
## 1️6️. Configure Frontend Environment (.env)

Navigate to frontend folder:

```bash
cd EasyCRUD/frontend
```
Create the environment file:
```bash
nano .env
```
Paste your backend endpoint (from kubectl get svc):
```bash
VITE_API_BASE_URL=http://<LOADBALANCER-ENDPOINT>:8080
```
**Make sure the endpoint is from the LoadBalancer service of the backend.**

Save & exit:
CTRL + O, ENTER, CTRL + X

### 1️7️. Create Frontend Dockerfile
Create Dockerfile for your React/Vite frontend:

```bash
nano dockerfile
```
Paste the following:

```bash
FROM node:24-alpine
COPY . /opt
WORKDIR /opt
RUN npm install && npm run build
RUN apk update && apk add apache2
RUN cp -rf dist/* /var/www/localhost/htdocs/
EXPOSE 80
CMD ["httpd", "-D", "FOREGROUND"]
```
Save & exit.

### 1️8️. Build Frontend Docker Image
```bash
docker build . -t frontend-app
```
### 1️9️. Run Frontend Container (Local Test)
```bash
docker run -d -p 80:80 frontend-app
```

### 2️0️. Tag Frontend Image for Docker Hub
```bash
docker tag frontend-app <your-dockerhub-username>/frontend-app:latest
```
### 2️1️. Push Frontend Image to Docker Hub
```bash
docker push <your-dockerhub-username>/frontend-app:latest
```

## 2️⃣2️⃣. Create Frontend Deployment (Kubernetes)

```bash
nano frontend-deploy.yaml
```
Paste the following:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      name: frontend
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: omkarmemane9/frontend:v1   # Replace with your Docker Hub image
          ports:
            - containerPort: 80
```
Save & exit.
Apply the deployment:
```bash
kubectl apply -f frontend-deploy.yaml
```
Check deployment status:
```bash
kubectl get deploy
```
```bash
kubectl get pods
```
### 2️3️. Create Frontend Service (LoadBalancer)
Create the service file:
```bash
nano frontend-svc.yaml
```
Paste the following:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
Save & exit.

Apply service manifest:
```bash
kubectl apply -f frontend-svc.yaml
```

# 24. Access the Application Through LoadBalancer

Once the **frontend-svc** is created, retrieve the LoadBalancer endpoint:

```bash
kubectl get svc frontend-svc
```
You will see an output similar to:

```pgsql

NAME            TYPE           CLUSTER-IP      EXTERNAL-IP                          PORT(S)        AGE
frontend-svc    LoadBalancer   10.0.121.45     a1b2c3d4e5f6-123456.elb.amazonaws.com 80:30080/TCP   2m
```
Copy the EXTERNAL-IP / LoadBalancer URL.

Open it in your browser:
```bash
http://<EXTERNAL-IP>
```
If everything is configured correctly, you will see the fully deployed web application frontend, communicating with the backend running inside your AWS EKS cluster.

## 🎉 Deployment Successful!

You have now successfully deployed a full-stack application using:

AWS EC2

AWS EKS

Docker

Kubernetes Deployments, Pods & Services

LoadBalancer for global frontend access

MariaDB / RDS backend

Docker Hub image storage

Your application is now live and production-ready.
