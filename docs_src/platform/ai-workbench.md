# AI Workbench

AI Workbench is the platform's **model catalog and development environment**. It holds every AI model onboarded as an "AI Service of type model" — LLMs, CNNs, custom algorithms, ASR engines, and more.

## 1. Model Benchmarking

Before choosing a model for a use case, AI engineers run benchmarking to identify the best performer:

- **Generic benchmarking:** Use publicly available or domain-specific ground truth datasets. Compare multiple models (e.g., Claude Sonnet, Gemini, GPT-4o) on metrics like accuracy, latency, and cost.
- **Use-case specific benchmarking:** Create a custom ground truth dataset in Data Hub specific to your use case (e.g., legal covenant extraction samples). Run a new benchmark to see which model performs best on your actual data.
- Results produce comparison reports showing metric scores across all selected models.

!!! tip "Best practice"
    Use generic benchmarking to eliminate clearly underperforming models, then run use-case specific benchmarking with your curated ground truth to make the final selection.

## 2. Model Fine-Tuning

When a model's off-the-shelf accuracy is insufficient:

1. Select the model to fine-tune (e.g., Whisper Large for ASR)
2. Choose the compute platform and cluster (RunPod, Denvr, Azure)
3. Select fine-tuning **aspects**:
    - *Core performance* — general accuracy improvement
    - *Robustness* — performance across acoustic noise levels, environments
    - *Ethical alignment* — bias mitigation, demographic fairness
4. Upload datasets per aspect to Data Hub; specify train/test split (e.g., 70/30)
5. Run the fine-tuning job; view iterative reports with metrics like Word Error Rate (WER), Character Error Rate (CER), BLEU scores
6. Compare multiple fine-tuning run results side-by-side including infrastructure consumption

## 3. AI Governance

Before deploying a model, generate governance reports against organizational policies:

- Upload governance policy documents (e.g., Singapore AI regulations, MUFG corporate AI policy)
- Select frameworks: **Atlas**, **NIST AI RMF** (more to be added)
- Create one or more policies (country-level, industry-level, corporate-level)
- Run an assessment — generates a scorecard showing model compliance with each policy
- All governance reports are stored centrally per model version

## The AI Workbench Workflow

```
Benchmark → Fine-Tune → Evaluate → Governance → Baseline & Promote
```

1. **Benchmark** — Identify the best candidate model for your domain
2. **Fine-Tune** — Improve accuracy on your specific use case data
3. **Evaluate** — Compare metrics across fine-tuning iterations
4. **Governance** — Run policy assessments and generate compliance scorecards
5. **Baseline & Promote** — Lock the validated model version for inferencing/production
