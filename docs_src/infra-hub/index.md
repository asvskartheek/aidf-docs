# Infra Hub

Infra Hub is the **infrastructure provisioning and management engine** of AIDF. It abstracts the complexity of setting up cloud resources — virtual machines, Kubernetes clusters, storage accounts — so every AI service can acquire the right compute at deployment time.

**Currently supports:** Azure, Databricks, RunPod, Denvr, on-premises data centers.

**Roadmap:** AWS, GCP, Alibaba Cloud, NVIDIA DGX appliances.

## Two Views

| View | Purpose | Key Sections |
|------|---------|--------------|
| **Admin View** | Manage workspaces, projects, HIL tasks | Dashboard, Workspaces, Projects, HIL Tasks |
| **Config View** | Manage AI services, pipelines, approvals | Dashboard, HIL Tasks, AI Services, Pipeline Template, My Reviews |

Use the top-left dropdown to switch between views.

## Navigation

- [Workspaces](workspaces.md) — Top-level cloud account/region containers
- [Projects & Environments](projects.md) — Scoped containers for AI service development
- [Managing AI Services](ai-services.md) — Deploy, test, approve, and manage AI Services
- [Pipeline Templates](pipeline-templates.md) — Build and promote reusable workflow templates
- [HIL Tasks](hil-tasks.md) — Human-in-the-Loop task management
- [Roles & Permissions](rbac.md) — User role definitions
