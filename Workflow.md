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

