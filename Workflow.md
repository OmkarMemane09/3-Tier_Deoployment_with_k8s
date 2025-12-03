# Deploying a Full-Stack Web App Using AWS EC2, AWS EKS, Docker & MariaDB  
*A Complete End-to-End Deployment Guide*

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
  type: NodePort
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
