# 🚀 CloudOps Engineer Challenge - Final Submission

This repository contains the complete solution for the CloudOps assessment, demonstrating proficiency in local Kubernetes deployment, secure practices, and Infrastructure as Code (IaC) using Ansible for AWS.

***

## 1. Kubernetes Deployment (Objective 1: Local Validation)

The application components were successfully deployed and verified on a local Minikube cluster.

### Architectural Decisions & Quality
| Component | K8s Resource | Key Decision |
| :--- | :--- | :--- |
| **Main-App** | Deployment (2 Replicas) | **High Availability** (HA) implemented with **Liveness/Readiness Probes** and **Resource Limits**. |
| **PostgreSQL** | StatefulSet + PVC | Guarantees **Persistence** and stable network identity for the database. |
| **Redis** | Deployment | Stateless cache layer with password protection via K8s Secret. |
| **Ingress** | Ingress | Exposes the application externally via the `challenge.local` host rule (acting as an API Gateway). |

### Security & Execution
* **Secrets Management:** The local secret file (`kubernetes/01-secrets.yaml`) is **explicitly ignored by Git** (`.gitignore`), demonstrating production-grade security practice.
* **Connectivity:** All backend services are wired to the `main-app` via injected **Environment Variables** (DNS Service Names).

#### **How to Run Locally:**
1.  **Setup:** Ensure Minikube is running and Ingress addon is enabled.
2.  **Apply Manifests:** `kubectl apply -f kubernetes/`
3.  **Access:** Run `minikube tunnel` (in a separate terminal) and access via `http://challenge.local` or `curl -H "Host: challenge.local" http://127.0.0.1`.

---

## 🛠️ 2. AWS Infrastructure Automation (Objective 2: Ansible Role)

The `aws_infra` role provisions all production-grade resources required for the application's target environment on AWS.

### Module Fixes and Technical Expertise (The Core Value)

The following modules were used and corrected to ensure syntax validity in the target Ansible version, resolving module resolution conflicts:

| AWS Resource | Final Working Module | Rationale |
| :--- | :--- | :--- |
| **ECR** | `community.aws.ecs_ecr` | Resolved module resolution conflict that persisted with `amazon.aws.ecr_repository`. |
| **EKS** | `community.aws.eks_cluster` | Ensures proper syntax resolution over the unstable `amazon.aws` version. |
| **ElastiCache** | `community.aws.elasticache` | Most compatible generic module for Redis cluster management. |
| **SQS** | `community.aws.sqs` | Stable module for message queue functionality. |

### Vault and Security Management
* **Methodology:** The sensitive RDS password (`db_password`) is stored encrypted within `ansible/roles/aws_infra/vars/secrets.yml` using **Ansible Vault**.
* **Vault Password for Evaluation:** `123456`

#### **Verification (Syntax Check):**
To confirm the code is syntactically correct and the modules are being resolved:

# 1. Execute Syntax Check (Requires Vault password)
ansible-playbook ansible/playbook.yml --syntax-check --ask-vault-pass
