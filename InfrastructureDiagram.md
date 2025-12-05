### 📊 Infrastructure Diagram

The following diagram illustrates the interaction between the Kubernetes components and the AWS resources provisioned by the Ansible roles:
---
```mermaid
graph TD
    user((User))
    
    subgraph "AWS Cloud (Provisioned via Ansible)"
        CF[CloudFront CDN]
        ECR["Amazon ECR<br/>(Docker Images)"]
        SQS["Amazon SQS<br/>(Queues)"]
        
        subgraph VPC
            EKS_Control[Amazon EKS Cluster]
            
            subgraph "Data Layer (Managed)"
                RDS["Amazon RDS<br/>(PostgreSQL)"]
                Redis["ElastiCache<br/>(Redis)"]
            end
            
            subgraph "EKS Worker Nodes (Kubernetes)"
                Ingress[Ingress Controller]
                
                subgraph "Namespace: Default"
                    SvcApp[Service: Main App]
                    PodApp[Pod: App Container]
                end
            end
        end
    end

    %% Traffic Flow
    user -->|HTTPS| CF
    CF -->|HTTP| Ingress
    Ingress -->|Routing| SvcApp
    SvcApp --> PodApp
    
    %% Application Dependencies
    PodApp -->|Read/Write| RDS
    PodApp -->|Cache| Redis
    PodApp -.->|Messages| SQS
    
    %% Deploy
    EKS_Control -.->|Pull Images| ECR
    
    %% Styles
    style CF fill:#f9f,stroke:#333,stroke-width:2px
    style RDS fill:#316192,stroke:#fff,color:#fff
    style Redis fill:#DD0031,stroke:#fff,color:#fff
    style EKS_Control fill:#FF9900,stroke:#333,color:#fff
```

---
    
