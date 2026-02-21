# Data Hub

Data Hub acts as the **enterprise data catalog and metadata store**. Every time an AI service processes or generates a data artifact, the service registers that output in Data Hub. It is the single source of truth for all data within AIDF.

## Key Capabilities

- **Data Dictionary:** Type, size, format, and schema for each data element.
- **Metadata Management:** Rich metadata including tags, categories, and custom attributes.
- **Lineage Tracking:** Full provenance — where the data came from (Data Marketplace, One Data Platform, external upload), what transformations were applied (PII suppression, encoding, cleansing), and where it is being consumed (AI Workbench training, GenAI Studio pipeline).
- **Human-in-the-Loop (HIL) Integration:** Trigger annotation tasks directly from Data Hub; completed annotations flow back automatically from Data Canvas.
- **Synthetic Data Generation:** Create synthetic datasets using LLMs with configurable prompts, model selection, quality-check models, and generation guidelines.
- **Self-Service Requests:** Select data elements and create annotation tasks (e.g., Audio Transcription, Video Annotation) that are published to the Data Canvas annotation platform.

## Data Ingestion Sources

| Source | Description |
|--------|-------------|
| **Data Marketplace** | 3,000+ curated, labeled datasets. |
| **One Data Platform** | Collect new data via crowd resources (images, video, audio). |
| **External Upload** | Upload from Azure Blob Store via an automated pipeline. *(Direct upload from local machine is a planned feature pending data compliance review.)* |
| **Pipeline Output** | Any AI service output can be registered directly into Data Hub. |

## Annotation Workflow

```
1. Select data elements in Data Hub
   └─ Choose individual files or a batch that require annotation

2. Create a Self-Service Request (SSR)
   └─ Select annotation template (Audio Transcription, Custom App from Data Canvas)

3. Task auto-published to Data Canvas
   └─ Human annotators receive the task with annotation guidelines

4. Annotator completes the task
   └─ Annotated data (ground truth labels) is submitted back

5. Run Manager Controller Pipeline
   └─ Exports completed task data from annotation platform back to AIDF storage

6. Run Bulk Load Pipeline
   └─ Imports annotated metadata into Data Hub; lineage updated
```

## Pre-Annotation with GenAI Studio Pipelines

Rather than sending raw data to humans, you can first run a GenAI Studio pipeline for **automated pre-annotation**, dramatically reducing annotator workload.

**Example — Isaac Snapdragon Pre-Annotation Pipeline** (built for Qualcomm):

1. Pull 800+ dark video files from Azure Blob Store
2. Apply brightness/contrast preprocessing
3. Run people detection, upper body, and head detection services
4. Detect PII, sensitive topics, and perform redaction
5. Push pre-annotated results to human annotators via HIL task

!!! success "Result"
    Humans only need to verify or correct pre-existing labels rather than annotating from scratch, cutting annotation time significantly.
