# Deployment Environments

| Environment | Purpose | Compute | Deployment Policy |
|------------|---------|---------|------------------|
| **Development (Dev)** | Feature development, experiments, initial debugging | Small VMs / low-end Kubernetes pods | Rapid, unchecked deployments. Developers verify functionality. |
| **QA / Staging** | Systematic testing, integration checks, regression | Medium instances, mirrors production scale | Functional + regression + integration tests. Surfaces config issues before staging. |
| **Staging** | Near-production replica, load testing, user acceptance | Medium-to-large, same images as Prod | Isolated from live traffic. Performance benchmarking, security scans, stakeholder demos. |
| **Production (Prod)** | Live environment serving real users | Large instances, HA clusters, optional multi-region | Only thoroughly tested, approved versions deployed. Full DR and scaling policies. |
