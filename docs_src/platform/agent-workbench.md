# Agent Workbench

Agent Workbench is the most recent module in AIDF. Its objective is to enable building, importing, validating, and monitoring **agentic LLM workflows** (LangGraph, custom multi-agent systems) for business process automation.

## Creating Agent Workflows

There are multiple entry points:

- **Natural language description:** A business user describes the workflow in plain English; the platform generates the agent workflow graph.
- **Import existing workflow:** AI engineers who have built LangGraph or similar workflows outside AIDF can import them.
- **Map to AI services:** Each workflow node is mapped to an onboarded AI service; engineers wire up the code for each node.

## Why Import an Already-Built Workflow?

Even if a workflow was built outside AIDF, importing it enables:

1. **Validation runs** — measure accuracy against ground truth datasets
2. **Safety evaluation** — test robustness against malicious inputs (prompt injection, adversarial data)
3. **Governance compliance** — check adherence to corporate/country AI governance policies
4. **Iterative improvement tracking** — history of all validation runs with logs and metrics
5. **Production promotion** — only workflows with acceptable metrics are promoted to production

## Case Study: MUFG Corporate Onboarding Workflow

**Business Problem:** MUFG relationship managers must manually produce 45+ KYC/onboarding documents after interviewing a corporate customer, then process signed documents back into the banking system.

**The Agent Workflow:**

1. Relationship manager uploads interview sheet (Excel/CSV) to the web app
2. **Node 1 (Completeness Check):** AI validates whether all required interview questions were answered
3. **Node 2 (Form Generation):** Generates 45 compliance and KYC documents from interview data
4. **Node 3 (Document Dispatch):** Sends documents to the end customer for digital/wet signature
5. Customer returns signed documents; relationship manager uploads them
6. **Node 4 (Internet Form Generation):** Produces internal banking forms from signed documents
7. **Node 5 (API Submission):** Submits forms to MUFG banking system via API to onboard the customer

## Validation & Evaluation

In Agent Workbench, AI engineers can run multi-pipeline evaluation reports:

- **Accuracy report:** Compare generated form fields against expected ground truth
- **Coverage report:** Percentage of required fields correctly populated
- **Security/safety report:** Test response to injected malicious content (e.g., "How do I prepare a bomb?" embedded in the interview sheet)
- **Governance report:** Check adherence to Singapore AI governance rules, MUFG corporate AI policy
- **History:** All validation run iterations are stored with logs, execution status, and metric trends

!!! note
    This demo was shown to multiple financial sector customers. MUFG was very close to signing a pilot at the time of training.
