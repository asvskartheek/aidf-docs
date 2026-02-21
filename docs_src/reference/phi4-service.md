# Microsoft Phi-4 Fine-Tuning Service

A complete AI Service implementation for fine-tuning Microsoft Phi-4 language models using distributed GPU computing. Demonstrates the full AIDF AI Service pattern for model-type services.

!!! info "Location"
    **Code directory:** `Code/microsoftphi4reference/`
    **Main service:** `src/microsoftphi4.py` (36 KB)
    **Fine-tuning factory:** `src/ModelFinetuningFactory.py` (239 KB)

## Features

- **Distributed GPU Computing:** RunPod integration for on-demand GPU infrastructure. Pay-per-use model. Parallel fine-tuning jobs across multiple GPUs.
- **Advanced Model Training:** HuggingFace Transformers integration. Transfer learning with custom datasets. Memory optimization strategies. Periodic checkpointing for recovery.
- **Comprehensive Monitoring:** Real-time metrics (training loss, validation loss, accuracy, learning rate). MLFlow integration for experiment logging and visualization. WER, CER, BLEU scores, and perplexity measurements.
- **Automated Data Pipeline:** DataMarket integration for dataset acquisition. Container deployment via ACR to RunPod. Central model repository with versioning.

## Service Workflow

```
1. Container Image Deployment
   Pull Docker image from Azure Container Registry → Deploy to RunPod GPU infrastructure

2. Dataset Acquisition
   Integrate with DataMarket platform → Download dataset by unique identifier

3. Model Training Pipeline
   Load Phi-4 model → Configure hyperparameters and GPU utilization → Run distributed training

4. Monitoring & Evaluation
   Real-time metrics capture → MLFlow logging → Multi-metric performance analysis

5. Storage & Reporting
   Database integration → Version-controlled model storage → Automated report generation
```

## Configuration Fields

```python
Config.HOST_URL                        # Service host URL
Config.ACR_NAME                        # Azure Container Registry name
Config.RUNPOD_API_KEY                  # RunPod API Key
Config.REGISTRY_AUTH_ID_RUNPOD         # Registry auth for RunPod
Config.TENANT_ID / CLIENT_ID / CLIENT_SECRET  # Azure AD credentials
Config.DB_CONNECTION_STRING            # Primary database
Config.CENTRAL_MODEL_REPO_CONNECTION_STRING   # Model artifact storage
Config.ML_FLOW_AZURE_CONNECTION_STRING # MLFlow Azure backend
```

## Key Source Files

| File | Purpose | Size |
|------|---------|------|
| `src/microsoftphi4.py` | Main service entry point, HTTP route, context orchestration | 36 KB |
| `src/ModelFinetuningFactory.py` | Complete fine-tuning pipeline — data loading, training loops, checkpointing, evaluation | 239 KB |
| `src/HuggingFaceModelFactory.py` | Model download, configuration, and initialization from HuggingFace Hub | 25 KB |
| `src/rlhf.py` | Reinforcement Learning from Human Feedback implementation | 27 KB |
| `src/ollama_infer.py` | Local inference via Ollama runtime | 26 KB |
| `src/datasheet_info_fetch_and_insert.py` | Dataset metadata management and insertion | 47 KB |
| `src/api_data_access_layer.py` | Database CRUD operations for training runs and results | 17 KB |
| `resources/model_queries.py` | SQL query templates for model metadata operations | 12 KB |
| `config/ai_llm_model_finetuning.json` | Full fine-tuning request schema with hyperparameters, compute config, evaluation settings | 15 KB |
