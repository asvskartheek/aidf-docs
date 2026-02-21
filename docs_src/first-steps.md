# First Steps on AIDF

A hands-on walkthrough that takes you from zero to running your first AI pipeline on the Centific AI Data Foundry platform.

!!! warning "Before you begin"
    You need a Centific employee account. Ask your workspace admin to add you to the relevant project. For workspace creation you need the **Platform Admin** role.

**Choose your path:**

=== "Business User / Analyst"
    You want to understand what AIDF does, explore existing pipelines, and trigger annotation tasks — without writing code.

    Follow **Steps 1 → 2 → 4 → 7**

=== "AI Engineer / Developer"
    You'll build AI services, deploy them to the platform, wire them into pipelines, and run experiments.

    Follow **all 7 steps**

---

## Step 1 — Understand the Platform

AIDF is made up of **six modules**, each handling a different stage of the AI lifecycle. Think of it as an assembly line: data flows in, gets processed, models get trained, then AI workflows get deployed to automate business processes.

```
Data Hub → AI Workbench → GenAI Studio → Agent Workbench
        (supported by: Infra Hub + GenBI)
```

**Three things you need to know:**

**Everything is an "AI Service"**
Every capability on the platform — an OCR engine, a speech transcriber, a YOLO object detector — is packaged as a self-contained microservice with a standard HTTP endpoint. You build services, deploy them, and chain them together into pipelines.

**Workspaces → Projects → Services**
The hierarchy is: a **Workspace** maps to a cloud account/region (one per business unit or department), a **Project** is a specific initiative inside that workspace, and **AI Services & Pipelines** live inside a project. You always work within a project context.

**Data flows from Hub → Canvas → Hub**
Raw data enters **Data Hub**. If it needs human labeling, it goes to **Data Canvas** (annotation platform). Labeled data returns to Data Hub. That labeled data then trains/fine-tunes models in **AI Workbench**, and the trained model powers an AI Service used in a **GenAI Studio** pipeline.

!!! tip
    AIDF ships with **100+ pre-built pipeline templates** covering common AI tasks. You don't need to build everything from scratch — explore the template library in GenAI Studio first.

---

## Step 2 — Log In & Orient Yourself

AIDF uses your **Centific employee credentials** (SSO). No separate account needed.

- [ ] Navigate to the AIDF platform URL provided by your admin — you'll be redirected to Microsoft SSO login
- [ ] Sign in with your Centific email and password
- [ ] Check that your name and role appear in the top-left dropdown (Platform Admin or your assigned role)

After login you'll see tiles for all six modules. Click any tile to enter that module. The most important one to start with is **Infra Hub** — it's where all workspace and project setup happens.

**Two views in Infra Hub:** Use the top-left dropdown to switch:

- **Admin View** — manage workspaces & projects
- **Config View** — deploy AI services & pipelines

!!! info
    If you can't see a module or get a permission error, you haven't been added to a project yet. Contact your workspace admin — they need to add your email with an appropriate role.

---

## Step 3 — Set Up Your Workspace

!!! warning "Platform Admin role required"
    Only admins can create workspaces. If you're an AI engineer or business user, skip to [Step 4](#step-4-create-a-project) — your admin will have already set this up.

A workspace ties together a **cloud account** (e.g., your team's Azure subscription), a **region** (e.g., East US), and one or more **environments** (Dev, QA, Staging, Prod).

```
Workspace: MUFG-GCMD-Singapore (Azure East Asia)
  └─ Environment: Dev (Small)
  └─ Environment: QA (Medium)
  └─ Environment: Prod (Large)
       ├─ Project: Customer-Onboarding
       └─ Project: Legal-Document-Analysis
```

**Creating a workspace — step by step:**

- [ ] Go to **Infra Hub** → click the **Workspaces** tab → click `+ Workspace`
- [ ] Choose your cloud provider: **Microsoft Azure** (most common), AWS, GCP, or NVIDIA NGC
- [ ] Enter the admin email → click **Continue** *(triggers an approval email)*
- [ ] Fill in workspace name, description, Azure subscription account, and region
- [ ] Add at least one environment. Click **Add** → choose type and size *(start with Dev/Small)*
- [ ] Click **Submit** → **Proceed**

**Choosing an environment size:**

| Size | Use Case |
|------|----------|
| **Small** | Exploration and early dev. Single small VM or low-end Kubernetes pod. Cheapest option. |
| **Medium** | QA and staging. Mirrors production performance without full scale. |
| **Large** | Production. High-throughput clusters, HA setup, disaster recovery. |

!!! tip "Key insight"
    Creating an environment only sets a **quota** — no real compute is reserved until a workload runs. You only pay for what you use.

---

## Step 4 — Create a Project

A project is scoped to a workspace and environment. Every pipeline you build, every AI service you deploy, and every data experiment you run belongs to a project.

- [ ] In Infra Hub, click the **Projects** tab → `+ Project`
- [ ] Enter a project name (e.g., `customer-onboarding-v1`) and a short description *(you can't rename projects later)*
- [ ] Select the parent **Workspace** from the dropdown
- [ ] Assign team members: click `+ Assign` → search by name → select role *(Platform Admin, AI Engineer, QA, Architect, Auditor)*
- [ ] Select the environment(s) to attach *(start with Dev)*
- [ ] Click **Submit** → **OK**

**Switching into a project:** All work in Config view happens within a project context.

1. Click the role/view dropdown (top-left)
2. Select **Config** to enter Config View
3. Click the project-switcher icon (↔) next to the project name
4. Select your project and environment → click **Submit**

!!! tip
    Always verify which project and environment you're in before deploying a service or running a pipeline. Deploying to production when you meant to use Dev is a common mistake.

---

## Step 5 — Deploy an AI Service

An AI service is the unit of intelligence on AIDF — every pipeline node is an AI service.

!!! info
    If you just want to *use* existing services, you can build a pipeline using services that are already deployed and public in your workspace. This step is for **AI Engineers** building a new service.

### Option A — Use an existing public service

- [ ] Switch to **Config View** in Infra Hub → click **AI Services**
- [ ] Use the **Components** dropdown to browse by category (Ingestion, Pre-processing, Storage, etc.)
- [ ] Click **Action → Deploy** on any public service you want to use in your project

### Option B — Deploy a new service you've built

- [ ] Zip your entire service directory: `zip -r my_service.zip my_service/` *(ZIP name must match the `<Name>` field in manifest.xml)*
- [ ] Upload the ZIP to the **Public Artifactory** using the Agile Workbench VS Code plugin
- [ ] In Infra Hub Config View → **AI Services** → click `+ Add`
- [ ] Select visibility: **Private** (this project only) or **Public** (all projects in workspace)
- [ ] Select your service ZIP from the list → click **Submit** *(Manifest Engine builds image → pushes to ACR → deploys to AKS)*
- [ ] Click **Action → Test** with a sample payload to verify output
- [ ] A Platform Admin reviews and **approves** your service in **My Reviews**

!!! tip
    While you wait for approval, write your `auto_qa` unit tests and verify `input.json` schema matches your service's actual payload.

---

## Step 6 — Build Your First Pipeline

A pipeline template is an ordered set of AI service nodes. Once built and approved, a template becomes a live instance in GenAI Studio.

- [ ] In Infra Hub Config View → click **Pipeline Template** → `+ Pipeline`
- [ ] Browse services by component. Drag your first service onto the canvas
- [ ] Draw a line from the **Start** node to your first service box
- [ ] Add more services and connect them in sequence
- [ ] **Configure each node**: double-click a service box → fill in parameters → click **Set**
- [ ] *(Optional)* Add **conditional branching**: double-click an arrow → write the condition using output KPI variables (e.g., `pii_detected == true`)
- [ ] Enter a **Pipeline Name**, description, and category → click **Create**

**Minimal example — Video object-detection pipeline:**

```
Start → Video Ingestion (Azure Blob → AIDF) → YOLO Detection → Output to Data Hub
```

**Getting the pipeline approved:**

- [ ] Click **Action → Test** with a sample input. Check status: Initiated → Success
- [ ] Ask a Platform Admin to review it in **My Reviews** → approve
- [ ] After approval, your pipeline template becomes available as an instance in **GenAI Studio**

---

## Step 7 — Run & Inspect Results

### Running the pipeline in GenAI Studio

- [ ] Navigate to **GenAI Studio** from the app dashboard
- [ ] Find your approved pipeline in the pipeline list
- [ ] Click the pipeline → click **Run** (or schedule it)
- [ ] Confirm the input payload → click **Execute**
- [ ] Click **View Result** once status shows Success (see per-service output, KPIs, execution time)
- [ ] Click **View Execution Trajectory** to trace data flow through each node

### Verifying output in Data Hub

If your pipeline registers output in Data Hub (`Storage Type: Physical`):

- [ ] Navigate to **Data Hub** → select your Industry section
- [ ] Find the dataset tile matching your pipeline output → click **Get Details**
- [ ] Click the **Data** tab → verify new files with expected format and IDs
- [ ] Check the **Lineage** tab to confirm the pipeline as the source

### Troubleshooting

| Symptom | Action |
|---------|--------|
| **Pipeline Failed** | Click the failed node in the execution trajectory. Check **Service Output** for error message. Look at container console logs for stack trace. |
| **Service not available** | Go to Infra Hub → Config View → AI Services. Check the Deploy Status column — it may not be approved yet. |
| **Empty output** | Check that the source container name and folder path match the actual input data location in Azure Blob Storage. |
| **Wrong results** | Verify `config/input.json` payload. Run the service individually via HTTP endpoint using Postman before placing it in a pipeline. |

---

## You're up and running!

**Your first-steps checklist:**

- [ ] I understand the six core AIDF modules and how data flows between them
- [ ] I can log in and navigate between Admin View and Config View in Infra Hub
- [ ] I know what a workspace is and how to create one (or find mine)
- [ ] I have been added to a project and can switch into it
- [ ] I can browse the AI services catalog and deploy/use existing services
- [ ] I can build a simple pipeline by dragging services onto the canvas and configuring each node
- [ ] I can run a pipeline, monitor its status, and inspect results in Data Hub

**Where to go next:**

- [Data Hub →](platform/data-hub.md) — Ingest datasets, set up HIL annotation tasks, use synthetic data generation
- [AI Workbench →](platform/ai-workbench.md) — Benchmark models, fine-tune with your data, run governance assessments
- [Agent Workbench →](platform/agent-workbench.md) — Automate a business process end-to-end using LangGraph
- [Developer Guide →](developer/index.md) — Write a new AI service from scratch

!!! tip "Questions?"
    Post them to the AIDF community forum (link shared by Ravi during training) — the team commits to responding by the next business day. For code-level questions, reach out to `ai-services-team@centific.com`.
