# Compute Offloading Architecture

For computationally intensive workloads (model training, large-scale inference), AIDF services offload execution to external GPU infrastructure while keeping the main service container lightweight.

## How It Works

1. The AI service receives a training/inference request via its HTTP endpoint in AKS
2. The `platform_cluster_manager.py` selects the appropriate compute provider based on configuration
3. The selected deployment module (RunPod/Denvr/on-prem/Azure) provisions a GPU cluster or pod
4. The Docker image (from ACR) is deployed to the GPU infrastructure
5. The heavy computation runs on GPU hardware
6. Results are stored in the central model repository (Azure Blob) or database
7. The main service polls for completion and returns results to the caller

## Supported Platforms

| Platform | Best For | Billing |
|----------|---------|---------|
| **RunPod** | Burst fine-tuning jobs | Pay-per-use GPU pods |
| **Denvr** | Large-scale training with consistent performance | High-performance dedicated GPU clusters |
| **On-Premises** | Data-sovereign workloads where data cannot leave the organization's network | Internal GPU servers |
| **Azure** | Integrated Azure-native deployments | AKS node pools with GPU-enabled VMs |
