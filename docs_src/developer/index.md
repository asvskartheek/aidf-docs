# What is an AI Service?

An **AI Service** in the AIDF ecosystem is a self-contained, deployable component that exposes a well-defined HTTP endpoint to perform one or more AI or data-processing tasks. It is the fundamental unit of all intelligence in the platform.

## Core Properties

1. **Follows a Standard Folder Structure** — ensures consistency across all services and teams
2. **Implements a Design-Pattern Backbone** — Abstract Class + Concrete Class + Context/Strategy Class
3. **Exposes a Consistent Endpoint** — pattern: `/datafactory/{module}/{serviceName}`

## Endpoint Pattern

```
# Standard AIDF AI Service endpoint:
/api/aiservice/centific/product/aidatafoundary/frontcontroller/
  {workspace-name}/{project-name}/{serviceName}/{aiservice_id}/{deploy_env_id}/test
```

## Developer Guide Navigation

- [Folder Structure](folder-structure.md) — Standard directory layout for every AI Service
- [Strategy Design Pattern](strategy-pattern.md) — Abstract Class → Concrete Class → Context
- [Logging](logging.md) — Verbose and console log types
- [Onboarding](onboarding.md) — Step-by-step deployment process
- [Docker & Deployment](docker.md) — Containerization and the Manifest Engine
- [Manifest File](manifest.md) — Deployment blueprint reference
