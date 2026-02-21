# Managing AI Services in Infra Hub

AI Services are the **fundamental building blocks** of AI intelligence in AIDF. They are categorized into three types:

- **Data Services:** Data processing, storage, and transformation
- **Model Services:** ML/AI model capabilities (inference, fine-tuning)
- **Application Services:** End-to-end AI applications

## Service Visibility

| Visibility | Description |
|-----------|-------------|
| **Public** | Available as a shared library across all projects in the workspace. Added to the common library. Can be licensed. |
| **Private** | Available only within the specific project. Used for proofs-of-concept or proprietary services. |

## Adding an AI Service

1. Switch to **Config view**
2. Click **AI Services** → `+Add`
3. Select visibility (Public/Private)
4. Select the service ZIP file *(must be uploaded to Azure Public Artifactory first)*
5. Click Submit — the Manifest Engine automatically builds and deploys the service
6. A reviewer must **approve** the service before it can be used in pipelines

## AI Service Management Options

| Action | Effect |
|--------|--------|
| **Test** | Run a validation test with a sample payload before approving |
| **Deploy** | Deploy a public service to the current project |
| **Update** | Replace current version; old version is automatically undeployed |
| **Undeploy** | Stop the service and release all allocated resources |
| **Delete** | Completely remove the service and perform full resource cleanup |
