# Reference Template

The `reference_template` project is the **canonical starting point** for building any new AI Service on AIDF. Copy this template, rename it, and implement your service-specific logic in the concrete class.

## Structure

```
reference_template/
├── auto_qa/
│   └── template_unit_test.py    # 7.7 KB — unit test scaffold
├── config/
│   ├── input.json               # Input schema template
│   ├── output.json              # Output schema template
│   ├── feedback.json            # HIL feedback template
│   └── env.config               # Environment configuration
├── lib/
│   └── requirements.txt         # Base dependencies
├── scripts/
│   └── manifest.xml             # Deployment blueprint template
├── src/
│   ├── Abstract_Class.py        # Abstract base class skeleton
│   ├── aiservice.py             # Context class + HTTP route (23 KB)
│   ├── environment.py           # Config management
│   ├── constants.py             # Shared constants
│   ├── api_data_access_layer.py # Database access layer
│   └── compute_offloading/     # Multi-platform compute
│       ├── abstract_class.py    # Compute offloading contract
│       ├── azure_deploy.py      # Azure deployment
│       ├── denvr_deploy.py      # Denvr GPU cluster deployment (13 KB)
│       ├── on_prem.py           # On-premises deployment (41 KB)
│       ├── runpod_deploy.py     # RunPod GPU deployment (11 KB)
│       └── platform_cluster_manager.py  # Cluster orchestration
└── Dockerfile
```

## Compute Offloading

The `compute_offloading` module enables heavy compute jobs (training, inference on large models) to run on external GPU infrastructure while the main service container remains lightweight:

| Platform | Use Case | File |
|----------|---------|------|
| **RunPod** | On-demand GPU pods, pay-per-use. Ideal for fine-tuning jobs. | `runpod_deploy.py` |
| **Denvr** | High-performance GPU clusters. Suitable for large-scale training. | `denvr_deploy.py` |
| **On-Premises** | Internal GPU servers for data-sovereign workloads. | `on_prem.py` |
| **Azure** | Azure VMs with GPU support via AKS node pools. | `azure_deploy.py` |
