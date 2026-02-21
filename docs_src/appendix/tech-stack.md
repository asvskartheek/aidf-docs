# Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Front-End** | Angular | Web application portal for browsing, configuring, and invoking AI services |
| **Back-End** | Python Flask | Powers each AI service HTTP endpoint; orchestration logic; Azure component integration |
| **Data Storage** | PostgreSQL | Primary relational database for metadata, execution logs, and application state |
| **Container Orchestration** | Azure Kubernetes Service (AKS) | All AI services run in Docker containers, ensuring portability and scalability |
| **Container Registry** | Azure Container Registry (ACR) | Stores private Docker images for all AI services |
| **Artifact Storage** | Azure Blob Storage | Public Artifactory for service ZIPs; model artifacts; pipeline outputs |
| **Secret Management** | Azure Key Vault + HashiCorp Vault | All connection strings, API keys, credentials stored and retrieved at runtime |
| **ML Experiment Tracking** | MLFlow | Automatic experiment logging, comparison, and visualization for model training |
| **GPU Compute (Cloud)** | RunPod, Denvr, Azure GPU VMs | On-demand GPU nodes for training and heavy inference |
| **Distributed Processing** | Ray | Parallel task execution for large-scale video/data processing |
| **Logging** | ELK Stack (Elasticsearch) | Structured execution logs indexed for search and monitoring |
| **IDE Integration** | Agile Workbench (VS Code plugin) | Upload service ZIPs to Public Artifactory directly from VS Code |
