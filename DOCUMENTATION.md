# 📘 Project Documentation: Kubernetes Challenge & CloudOps

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

This repository contains a complete solution for deploying a three-tier application (**Frontend/Backend**, **Database**, **Cache**) and the infrastructure automation required to host it on AWS.

---

## 1. Architecture Overview

The project is divided into two main pillars:

* **Application Orchestration (Kubernetes):** Configuration to run the application locally using **Minikube**.
* **Infrastructure as Code (Ansible):** Playbooks dedicated to provisioning production resources on **AWS**.

### 🛠️ Technology Stack

| Category | Technology | Version/Detail |
| :--- | :--- | :--- |
| **Containerization** | Docker | - |
| **Orchestration** | Kubernetes (K8s) | Minikube, Kubectl |
| **IaC** | Ansible | Amazon.AWS Collection |
| **Database** | PostgreSQL | Version 14 |
| **Cache** | Redis | Version 7 (Alpine) |
| **Web Server** | Nginx | Ingress & App Container |

---

## 📂 2. Directory Structure

Below is the file organization for the project:

```plaintext
.
├── ansible/                          # AWS Infrastructure Automation
│   ├── playbook.yml                  # Main Playbook
│   ├── requirements.yml              # Dependencies (AWS Collections)
│   └── roles/aws_infra/              # Custom role for resource creation
│       ├── defaults/                 # Default variables
│       ├── tasks/                    # Resource definitions (ECR, EKS, RDS, etc.)
│       └── vars/                     # Environment variables and encrypted secrets
├── Kubernetes/                       # Local Deployment Manifests (Minikube)
│   ├── 02-postgres.yaml              # PostgreSQL StatefulSet
│   ├── 03-redis.yaml                 # Redis Deployment
│   ├── 04-app.yaml                   # Main Application Deployment
│   ├── 05-ingress.yaml               # External routing rules
│   └── 01-secrets.yaml               # (IGNORED BY GIT) Must be created manually
├── README.md                         # Quick start guide for Kubernetes
└── RequirementsSetup.md              # Environment setup guide (Windows/WSL)
```

---

## ☸️ 3. Kubernetes Details (Local Deploy)

The application is designed to run on a local cluster (Minikube) and consists of the following components:

### 🌐 Ingress & Network
* **File:** `05-ingress.yaml`
* **Host:** `challenge.local`
* **Behavior:** Redirects port 80 traffic to the `main-app` service.
* **Requirement:** Requires the ingress addon enabled (`minikube addons enable ingress`).

### 📱 Main Application (Main App)
* **File:** `04-app.yaml`
* **Type:** Deployment (2 replicas) + Service (ClusterIP).
* **Image:** `nginx:latest`.
* **Configuration:**
    * Connects to Postgres and Redis via environment variables (`DB_HOST`, `REDIS_HOST`).
    * Configured with Liveness and Readiness probes checking the `/` route.
    * Resource limits defined (CPU: 250m, Memory: 128Mi).

### 🗄️ Database (PostgreSQL)
* **File:** `02-postgres.yaml`
* **Type:** StatefulSet (Ensures persistence and ordering).
* **Storage:** Requests 1Gi of persistent storage (`postgres-storage`).
* **Security:** Password injected via Secret (`app-secrets`).

### ⚡ Cache (Redis)
* **File:** `03-redis.yaml`
* **Type:** Deployment.
* **Security:** Starts server with `--requirepass` flag, reading the password from Secrets.

---

## ☁️ 4. Ansible Details (AWS Infrastructure)

The Ansible automation aims to create a production environment on AWS similar to the local environment.

### Role: `aws_infra`
Located in `ansible/roles/aws_infra/tasks/main.yml`, this role provisions:

* **ECR Repository:** To store application Docker images.
* **EKS Cluster (Kubernetes):** Managed cluster (version 1.30) with Node Group.
* **RDS Instance:** Managed PostgreSQL database (`db.t3.micro`).
* **ElastiCache:** Managed Redis cluster.
* **SQS Queue:** Message queue (additional architectural resource).
* **CloudFront Distribution:** CDN for content delivery, pointing to a Load Balancer (placeholder).

### 🔐 Variables & Secrets

* **Main Variables:** Region, Names, Instance Types are located in `ansible/roles/aws_infra/vars/main.yml`.
* **Vault:** There is an encrypted `secrets.yml` file protected by Ansible Vault.

> [!TIP]
> **Development Credentials**
> The vault password provided for documentation/testing purposes is: `123456`