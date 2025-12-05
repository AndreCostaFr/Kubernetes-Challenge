readme_file:
  filename: "README.md"
  encoding: "utf-8"
  description: "Official documentation for the Kubernetes Challenge deployment"
  content: |
    # Kubernetes Challenge Deployment Guide

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
    git clone [https://github.com/AndreCostaFr/Kubernetes-Challenge](https://github.com/AndreCostaFr/Kubernetes-Challenge)
    cd Kubernetes-Challenge
    ```

    ### Start Minikube
    Initialize your local Kubernetes cluster:

    ```bash
    minikube start
    ```

    > **⚠️ Note:** Ensure Minikube has enough resources (CPU/Memory) allocated if you are running other heavy applications.

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

    > **Important:** Keep this terminal window **open** while testing.

    ---

    ## ✅ 4. Verification and Access

    Verify that the application is running correctly using the method appropriate for your operating system.

    ### Option A: Linux / macOS (Ubuntu)
    If you are on a Unix-based system, you can use `curl` by passing the Host header defined in the Ingress.

    ```bash
    curl -H "Host: challenge.local" [http://127.0.0.1/](http://127.0.0.1/)
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





    ansible_docs:
  filename: "ANSIBLE_README.md"
  encoding: "utf-8"
  description: "Documentation for Ansible configuration and static verification"
  content: |
    # Ansible Configuration and Verification Guide

    

[Image of Ansible architecture diagram]


    This document details the standard operating procedures for validating the Ansible automation scripts.

    > **ℹ️ Note:** Due to the current absence of a live AWS environment, validation is restricted to **static analysis** and **syntax checking**. This ensures codebase integrity and proper formatting without provisioning or modifying actual cloud resources.

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







---


    graph LR
    subgraph "Automação Ansible"
        Playbook[Playbook: aws_infra]
    end

    subgraph "AWS Cloud (us-east-1)"
        direction TB
        
        %% Compute & Container
        Playbook -- "Cria" --> EKS[EKS Cluster: cloudops-challenge-cluster]
        Playbook -- "Cria" --> ECR[ECR Repo: cloudops-challenge-repo]
        
        %% Database & Cache
        Playbook -- "Cria" --> RDS[RDS Postgres: cloudops-challenge-db]
        Playbook -- "Cria" --> ElastiCache[Redis: cloudops-challenge-cache]
        
        %% Messaging & CDN
        Playbook -- "Cria" --> SQS[SQS Queue: cloudops-challenge-queue]
        Playbook -- "Cria" --> CloudFront[CloudFront Distro]
        
        %% Relações Implícitas (Lógicas)
        CloudFront -.->|"Origem (HTTP)"| LB[Load Balancer (Placeholder)]
    end
    
    style Playbook fill:#ee5,stroke:#333,stroke-width:2px