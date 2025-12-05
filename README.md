# Kubernetes Challenge Deployment Guide

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-326CE5?style=for-the-badge&logo=minikube&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/redis-%23DD0031.svg?style=for-the-badge&logo=redis&logoColor=white)

This document outlines the standard operating procedures for deploying the **Kubernetes Challenge** application stack. The deployment utilizes **Minikube** to orchestrate a local Kubernetes cluster containing Postgres, Redis, and the main application, exposed via an Ingress controller.

---

## 📋 Prerequisites

Before proceeding, ensure you have the following tools installed and configured on your local machine:

* **Git**: Version control system.
* **Minikube**: Local Kubernetes cluster manager.
* **kubectl**: Kubernetes command-line tool.

---

## 🚀 1. Environment Setup

Begin by cloning the project repository to your local machine and initializing the local cluster.

### Clone the Repository
Retrieve the necessary manifest files from the source repository:

```bash
git clone https://github.com/AndreCostaFr/Kubernetes-Challenge
cd Kubernetes-Challenge
```

---

### 🔒 Security & Secrets Configuration

**Important:** The configuration file `kubernetes/01-secrets.yaml` is intentionally **excluded from the repository** (via `.gitignore`). This follows production security best practices to prevent sensitive credentials from being committed to version control.

To run the application locally, you must create this file manually immediately after cloning the repository:

1.  Create a new file located at: `kubernetes/01-secrets.yaml`
2.  Paste the following content into it:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  POSTGRES_PASSWORD: "bigHardPassword"
  REDIS_PASSWORD: "anotherBigHardPassword"
```

---
### Start Minikube
Initialize your local Kubernetes cluster:

```bash
minikube start
```

> [!WARNING]
> Ensure Minikube has enough resources (CPU/Memory) allocated if you are running other heavy applications.

---

## 📦 2. Resource Deployment

Apply the manifest files in the specific order below to ensure dependencies (such as Secrets and Databases) are created before the application logic.

### Apply Manifests

**1. Secrets**
*Establish secure credentials.*
```bash
kubectl apply -f Kubernetes/01-secrets.yaml
```

**2. PostgreSQL**
*Deploy the primary database.*
```bash
kubectl apply -f Kubernetes/02-postgres.yaml
```

**3. Redis**
*Deploy the caching layer.*
```bash
kubectl apply -f Kubernetes/03-redis.yaml
```

**4. Application**
*Deploy the main application logic.*
```bash
kubectl apply -f Kubernetes/04-app.yaml
```

**5. Ingress**
*Configure external access routing.*
```bash
kubectl apply -f Kubernetes/05-ingress.yaml
```

---

## 🌐 3. Network Configuration

To make the Ingress resource accessible from your localhost, you must create a network route.

### Enable Tunneling
Open a **new terminal window** and run the following command to create a network route to the cluster services.

```bash
minikube tunnel
```

> [!IMPORTANT]
> Keep this terminal window **open** while testing.

---

## ✅ 4. Verification and Access

Verify that the application is running correctly using the method appropriate for your operating system.

### Option A: Linux / macOS (Ubuntu)
If you are on a Unix-based system, you can use curl by passing the Host header defined in the Ingress.

```bash
curl -H "Host: challenge.local" http://127.0.0.1/
```

### Option B: Windows
On Windows, you may need to forward the service port directly to access the application locally.

**1. Forward the Port:**
```powershell
kubectl port-forward service/main-app 8080:80
```

**2. Access the Application:**
Open your browser to [http://localhost:8080](http://localhost:8080) or use curl:

```powershell
curl localhost:8080
```

---

## 🔧 Troubleshooting

| Issue | Suggested Fix |
| :--- | :--- |
| **Pods Pending** | Check if Minikube has sufficient resources using `kubectl describe pod <pod-name>`. |
| **Connection Refused** | Ensure `minikube tunnel` is running in a separate terminal window (requires root/admin on some OS). |











# Ansible Configuration and Verification Guide

![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)

This document details the standard operating procedures for validating the Ansible automation scripts designed to provision the infrastructure illustrated below.

> [!NOTE]
> Due to the current absence of a live AWS environment, validation is restricted to **static analysis** and **syntax checking**. This ensures codebase integrity and proper formatting without provisioning or modifying actual cloud resources.

---

### 📋 System Requirements

Ensure your environment meets the following version standards before proceeding:

* **Ansible**: Version **2.17+** is required.
    > *Note: This project was developed and validated using **Ansible Core 2.19.4**.*

---

## 🛠️ 1. Dependency Management

Before executing any playbooks, it is mandatory to ensure that all external roles and collections required by the project are installed in your local environment.

### Install Requirements
Execute the following command to download the dependencies defined in your requirements file:

```bash
ansible-galaxy install -r requirements.yml
```

---

## ✅ 2. Syntax Validation

To verify the structure and validity of the YAML files without triggering a deployment, utilize the syntax check mode. This process parses the playbook to report formatting errors, indentation issues, or missing modules.

### Run Syntax Check
Execute the playbook using the `--syntax-check` flag. You will be prompted for the Vault password to decrypt sensitive variables required for the validation process.

```bash
ansible-playbook ansible/playbook.yml --syntax-check --ask-vault-pass
```

### 🔐 Credentials
When the prompt appears, utilize the following development credential to decrypt the Vault:

> **Vault Password:** `123456`

---

## 🚩 Summary of Flags

Below is a reference guide for the command-line flags used in the validation process:

| Flag | Description |
| :--- | :--- |
| `--syntax-check` | Performs a parse of the playbook to check for syntax errors. **Does not** execute any tasks on targets. |
| `--ask-vault-pass` | Interactively prompts the user to enter the password required to decrypt Ansible Vault files (`*.enc` or encrypted strings). |