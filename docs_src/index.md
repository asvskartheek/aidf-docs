# AIDF Platform Documentation

Comprehensive reference for the **Centific AI Data Foundry (AIDF)** — an end-to-end platform for developing, deploying, and managing enterprise AI solutions. Covers all six sessions of the February 2026 training.

<div class="grid cards" markdown>

-   :material-walk: **New here?**

    Start with the hands-on walkthrough that takes you from zero to a running pipeline.

    [First Steps →](first-steps.md)

-   :material-wrench: **Building an AI Service?**

    Full code tutorial: from the abstract contract to a deployed classifier.

    [Tutorial →](tutorial.md)

-   :material-server: **Managing infrastructure?**

    Workspaces, projects, service deployment, and pipeline templates.

    [Infra Hub →](infra-hub/index.md)

-   :material-code-braces: **Developer reference?**

    Folder structure, strategy pattern, manifest, Docker, and onboarding.

    [Developer Guide →](developer/index.md)

</div>

---

## Platform Overview

The **AI Data Foundry (AIDF)** is Centific's end-to-end platform for accelerating the development, deployment, and management of AI solutions. It serves as a comprehensive AI workflow orchestration environment that integrates data, models, and applications into one unified framework.

AIDF's core purpose is to streamline every step of AI development — from high-quality data curation and preparation to model training, fine-tuning, and deployment — all under a single platform.

### Modular Architecture

AIDF is designed with a **modular, plugin-based architecture**. Each component (called a "studio") is responsible for a different aspect of the AI lifecycle. The design allows teams to mix and match tools for specific use cases and scale independently.

The platform leverages **Kubernetes-based orchestration and containerization** to deploy AI services at scale. Each AI function is packaged as an **"AI Service"** — a self-contained, deployable microservice exposing a standard HTTP endpoint.

### Six Core Modules

| Module | Role |
|--------|------|
| **Data Hub** | Enterprise data catalog and metadata store. Manages data lineage, annotations, and human-in-the-loop tasks. |
| **AI Workbench** | Model catalog and development environment for benchmarking, fine-tuning, training, and governance. |
| **GenAI Studio** | Pipeline builder and AI services management hub. Orchestrates multi-step workflows with drag-and-drop. |
| **Agent Workbench** | Build, import, validate, and monitor agentic LLM workflows for business process automation. |
| **Infra Hub** | Infrastructure provisioning engine — manages workspaces, projects, compute, and AI service deployment. |
| **GenBI** | Natural-language cognitive analytics layer. Ask plain-English questions against structured data. |

### Business Value

- **End-to-end efficiency:** Covers every stage (ingestion, prep, training, evaluation, deployment, monitoring) under one platform. Includes 100+ pre-configured AI workflow templates.
- **Quality of outcomes:** Emphasizes high-quality data, model validation, RLHF feedback loops, integrated benchmarking, and drift monitoring.
- **Scalability & flexibility:** Cloud-native microservice architecture handles multimodal AI workloads (vision, language, speech) in real-time. Infrastructure-agnostic across public cloud, private data centers, and edge devices.
- **Reuse & collaboration:** Pipeline templates created by one team can be reused and scheduled for automated execution by others.

---

## Training Sessions

The AIDF platform training consisted of 6 sessions held February 3–11, 2026. Sessions 1–3 covered functional aspects; Sessions 4–6 covered technical deep-dives.

| Session | Date | Theme | Summary |
|---------|------|-------|---------|
| **1** | Feb 3 | Functional | Platform Introduction & Data Hub. Presenter Avinash Ganesh introduced all AIDF modules. Deep-dive into Data Hub, GenAI Studio pipeline builder with the Isaac Snapdragon pre-annotation pipeline example. |
| **2** | Feb 4 | Functional | AI Workbench in depth: model benchmarking, fine-tuning jobs, evaluation reports, AI Governance (Atlas/NIST AI RMF frameworks). |
| **3** | Feb 5 | Functional | Agent Workbench: live demo of MUFG corporate customer onboarding LangGraph workflow. Infra Hub: workspace/project management, compute sizing. |
| **4** | Feb 9 | Technical | AI Service creation: folder structure, Strategy Design Pattern, config management, manifest.xml, Docker basics. |
| **5** | Feb 10 | Technical | Model onboarding & fine-tuning: training vs. inference services, compute offloading (RunPod/Denvr/Azure), MLFlow. Microsoft Phi-4 reference walkthrough. |
| **6** | Feb 11 | Technical | Pipeline template creation in GenAI Studio, conditional branching, environment promotion (Dev → QA → Staging → Prod). |
