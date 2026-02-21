# AI Service Onboarding

After building and testing your AI Service locally, follow these steps to onboard it into AIDF:

## Onboarding Steps

1. **Role Switch** — Assume the **Platform Admin** role in Infra Hub to manage service packaging and deployment
2. **Code Packaging** — Organize code per AIDF folder structure → Create a ZIP archive of the entire service directory
3. **Publish to Artifactory** — Use the **Agile Workbench VS Code plugin** to upload the ZIP to the Public Artifactory (Azure Blob container)
4. **Set Visibility** — Choose **Private** (project-only) or **Public** (shared across workspace)
5. **Deploy via Manifest Engine** — Click Deploy → Manifest Engine builds Docker image → pushes to ACR → deploys to AKS
6. **Test** — Run a preliminary test with a sample payload to verify the service works correctly
7. **Admin Approval** — A reviewer approves the service in **My Reviews** — only then is it available for pipeline use

## Deployment Flow Detail

When you click Deploy in Infra Hub:

1. **Build Image:** Converts the ZIP from Artifactory into a Docker image using the Dockerfile in `scripts/`
2. **Push to ACR:** Uploads the image to Azure Container Registry under a predictable naming scheme
3. **Deploy to AKS:** Spins up the container on Azure Kubernetes Service, wiring in the correct HTTP route (`/datafactory/{component}/{serviceName}`) from the manifest's component field
