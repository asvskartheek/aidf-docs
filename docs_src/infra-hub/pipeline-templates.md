# Pipeline Templates

An AI pipeline is an automated, end-to-end workflow that transforms raw data into actionable AI outputs. A **template** is the reusable definition; an **instance** is a live execution of that template processing real data.

## Key Principles

- **Modularity:** Each component independently deployable and testable
- **Reproducibility:** Every run traceable and reproducible
- **Scalability:** Architecture handles varying data volumes and computational demands
- **Reliability:** Built-in error handling and recovery mechanisms
- **Observability:** Comprehensive monitoring and logging throughout

## Creating a Pipeline Template

1. Switch to Config view → click **Pipeline Template** → `+Pipeline`
2. Select the component and sub-component for each service
3. Drag and drop AI services onto the canvas; link from the Start point
4. Connect services to create logical flow; double-click arrows to add conditions
5. Configure each node (double-click → fill parameters → click Set)
6. Enter pipeline name, description, and category → click Create
7. Test the pipeline → submit for approval in **My Reviews**
8. After approval, the template becomes available in GenAI Studio as an instance

## Example: YOLO Video Auto-Labeling Pipeline

Identifies objects in videos and auto-annotates them, feeding undetected objects to human annotators.

```
Pipeline Flow:
  Start
    → YOLO Object Detection (Pre-processing)
        Source: Azure Blob Container
        Model: YOLOv8 / YOLOv9 / YOLOv10
        Output Format: YOLO + COCO
        Storage Type: Physical (pushes to Data Marketplace)
  End
```

## Example: VILA 15B Video Context Extraction

NVIDIA's VILA 15B model analyzes video at frame level, generating contextual insights based on a natural-language prompt.

```
Pipeline Flow:
  Start
    → Villa15-B-version2 (Pre-processing)
        Source: Azure Blob Container
        Prompt: "Identify all safety hazards and describe what is happening"
        SAS Token: Required for Azure Blob access
        Output: JSON context summary per video
  End
```

## Manager Controller & Bulk Load Pipelines

Two special system pipelines manage the Human-in-the-Loop annotation flow:

- **Manager_Controller:** Exports completed annotation task data from the annotation platform (using Task IDs) to configured storage and updates SSR status in AIDF.
- **Bulk_Load:** Imports exported annotated metadata into Data Hub, making it available for downstream consumption. Configurable by industry, usage, nature, storage type, and dataset category.
