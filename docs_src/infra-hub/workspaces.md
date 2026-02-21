# Workspaces

A **workspace** is the top-level logical boundary in AIDF. It defines both the cloud account/region and the environment stage (Dev, QA, Staging, Prod). All projects, resources, and AI services live inside a workspace.

## When to Create Workspaces

- One workspace per **business unit** (e.g., MUFG GCMD team, Legal team, Marketing)
- One workspace per **country/region** (e.g., Singapore, International)
- One workspace per **major AI initiative** (e.g., Customer Insight Generation)

## Creating a Workspace

1. **Navigate** to Workspaces tab → click `+Workspace`
2. **Choose provider:** Amazon Web Services, Microsoft Azure, Google Cloud Platform, NVIDIA NGC
3. **Enter Identity Details:** Admin user email *(triggers approval workflow — only authorized personnel can create workspaces)*
4. **Workspace Details:** Name, description, cloud account, region
5. **Add Environment(s):** Must add at least one. Choose type (Dev, QA, Stage, Prod) and size (Small/Medium/Large)
    - *Small:* Light development, minimal data, low-end Kubernetes pod
    - *Medium:* QA/staging, mirrors production performance without full scale
    - *Large:* Production, high-throughput, multi-region HA clusters
6. **Submit** for provisioning

!!! info "Dynamic resource allocation"
    Creating an environment defines a quota (max GPU units, CPU cores, memory) without committing physical hardware. Resources are allocated from the shared pool only when a workload actively runs — optimizing cost by paying only for active computation.

## Workspace Lifecycle

- **Edit:** Change description, add/modify environments
- **Delete:** Automatically deallocates ALL child projects, infrastructure (clusters, VMs, storage), and Kubernetes namespaces — preventing "zombie" resource costs
