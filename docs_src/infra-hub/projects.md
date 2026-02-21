# Projects & Environments

A **project** is a scoped container for AI service development, pipelines, and related artifacts. Projects must be created inside an existing workspace, inheriting its region, environment tag, and resource boundaries.

## Creating a Project

1. In Infra Hub, click **Projects** → `+ Project`
2. Enter project name and description
3. Select the parent workspace
4. Assign users and roles (Platform Admin, AI Engineer, QA, Architect, Auditor)
5. Select environment(s) to link to the project
6. Submit

## Environment Types

| Environment | Purpose | Characteristics |
|------------|---------|-----------------|
| **Development (Dev)** | Feature development, experiments, initial debugging | Smallest compute footprint. Rapid, unchecked deployments permitted. |
| **Quality Assurance (QA)** | Systematic testing, validation, integration checks | Medium instances. Functional, regression, and integration test suites. |
| **Staging** | Near-production replica, load testing, user acceptance | Matches production architecture. Same container images as Prod but isolated from live traffic. |
| **Production (Prod)** | Live environment serving real users | Largest compute allocation. Full disaster-recovery and scaling policies. Only approved service versions deployed. |

!!! info
    The environment within a workspace is **shared across all projects** in that workspace. If a workspace has a Medium environment, all projects under it share that compute pool.

## Switching Projects

Use the project switcher (top-left dropdown) → select Config view → click the switch icon → select the desired project and environment → Submit.
