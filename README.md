# 🚀 Full-Stack Application Deployment on AWS (EC2 + EKS + RDS + Docker + Kubernetes)

This document provides a **high-level technical overview** of deploying a production-ready full-stack application using **AWS Cloud, Docker, and Kubernetes**.

---
<img width="733" height="499" alt="image" src="https://github.com/user-attachments/assets/a8447cda-6d63-428b-98d0-66878d876677" />

## 🏗️ Architecture Overview
```
Developer → GitHub → EC2 Bastion
                                   ↳ Docker Build → Docker Hub
AWS RDS (MariaDB) ← Backend Pods (EKS) ← Cluster Nodes
                                   ↳ NodePort / LB
Frontend Pods (EKS) → LoadBalancer → End Users
```
---

## 🧰 Technologies & AWS Services Used

### **AWS Cloud**
- **EC2** → Bastion host + Docker build environment  
- **EKS** → Kubernetes cluster for frontend & backend  
- **RDS (MariaDB)** → Managed SQL database  
- **ELB (LoadBalancer)** → Public access point for frontend  

### **DevOps / Tools**
- **Docker & Docker Hub** → Containerization & image registry  
- **Kubernetes** → Pods, Deployments, Services  
- **kubectl & eksctl** → Cluster provisioning & management  
- **Git & GitHub** → Source code & version control  
- **Apache Web Server** → Serving production frontend  

---

## 🔑 Modules Deployed

### **1. Backend (Spring Boot)**
- Built using **Maven**
- Containerized into `backend-app` Docker image  
- Pushed to **Docker Hub**  
- Deployed as **Pod** in EKS  
- Exposed via **NodePort / LoadBalancer**

### **2. Frontend (React + Vite)**
- Production build using **Node**
- Served via **Apache** inside Docker container  
- Containerized into `frontend-app` Docker image  
- Deployed as **Deployment (3 replicas)** in EKS  
- Exposed via **AWS LoadBalancer**

### **3. Database (MariaDB on AWS RDS)**
- Hosted on **Amazon RDS**
- Connected using JDBC  
- Schema + table created manually  
- Accessed securely by backend pods  

---

## ⚙️ Deployment Workflow (Blueprint)

1. Launch EC2 instance → install:
   - Docker
   - kubectl
   - AWS CLI  

2. Connect EC2 to EKS cluster using IAM & kubeconfig  
3. Create RDS database → user → tables  
4. Containerize backend → build → tag → push  
5. Deploy backend to EKS → Pod + Service  
6. Configure frontend `.env` with backend LB endpoint  
7. Containerize frontend → build → tag → push  
8. Deploy frontend to EKS → Deployment + LoadBalancer  
9. Access final application using LoadBalancer URL  

---

## 📦 Kubernetes Components

| Component     | Purpose |
|--------------|---------|
| **Pods**      | Run backend & frontend containers |
| **Deployments** | Manage replicas, self-healing (frontend) |
| **Services** | NodePort for backend, LoadBalancer for frontend |
| **LoadBalancer** | Public access point for users |

---

## ✔️ Final Outcome

### 🎉 A Fully Production-Ready Cloud-Native Application

- Backend + Frontend fully **containerized**
- Highly available **Kubernetes deployments**
- Secure and scalable **AWS EKS cluster**
- Persistent and managed **AWS RDS MariaDB**
- CI-ready workflow using **Docker Hub**
- Application accessible globally via **AWS LoadBalancer**

---

## 🎯 Why This Architecture Works Well

- **Scalable** → Kubernetes auto-manages replicas & load  
- **Reliable** → Pod auto-recovery & rolling updates  
- **Secure** → RDS over local DB containers  
- **Portable** → Docker images run anywhere  
- **Cost-efficient** → Only pay for node groups, RDS & LB  
