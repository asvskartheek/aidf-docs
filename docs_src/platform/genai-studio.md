# GenAI Studio

GenAI Studio is the **central hub for managing AI services and building pipelines**. It reads each service's manifest to register API endpoints, allocate compute, and enforce visibility settings. It streamlines the entire deployment lifecycle from smoke test to production.

## Pipeline Builder

The drag-and-drop canvas lets you chain multiple AI services into end-to-end workflows:

- Select AI services from the component library (Data Factory, Feature Extraction, Content Generation, etc.)
- Drop services onto the canvas and connect them with arrows
- Configure each node: input/output parameters, model to use, storage configuration
- Add **conditional branching**: define conditions on output KPIs — e.g., "only proceed to PII redaction if sensitive topics are detected"
- Connect a Human-in-the-Loop node to push results to Data Canvas for expert review

## Pipeline Types

| Type | Purpose | Example |
|------|---------|---------|
| Pre-annotation | Automate label creation before human review | YOLO object detection → upper body detection → HIL task |
| Data processing | Transform, clean, and enrich data | Audio ingestion → noise reduction → speaker diarization → PII redaction |
| Post-annotation | Validate and consolidate annotated data | Quality check → Data Hub registration |
| RAG/Inference | Production AI inference pipelines | Document extraction → LLM extraction → structured output |

## Real-World Example: Isaac Snapdragon Pre-Annotation Pipeline

Built for Qualcomm's Isaac camera project, this pipeline processes dark security footage:

1. Pull 800+ video files from Azure Blob Store
2. Brightness/contrast preprocessing (making dark footage legible)
3. People detection service
4. Upper body and head detection
5. Plant and object detection
6. Aggregate detection outputs
7. PII detection and redaction (replace with beeps)
8. Sensitive topic detection
9. Push to HIL task for human verification

!!! success "Result"
    Humans only verify/correct pre-existing detections rather than labeling from scratch on near-invisible footage.

## Example: YOLO Video Auto-Labeling Pipeline

```
Start
  → YOLO Object Detection (Pre-processing)
      Source: Azure Blob Container
      Model: YOLOv8 / YOLOv9 / YOLOv10
      Output Format: YOLO + COCO
      Storage Type: Physical (pushes to Data Marketplace)
End
```

## Example: VILA 15B Video Context Extraction

NVIDIA's VILA 15B model analyzes video at frame level, generating contextual insights based on a natural-language prompt. Detects objects, actions, hazards, and interprets scenes.

```
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
