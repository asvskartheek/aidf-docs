# Docker & Deployment

## Key Concepts

- **Image:** A read-only template built from a Dockerfile. A snapshot of a filesystem plus metadata.
- **Container:** A running, read-write instance of an image. Start, stop, and delete without affecting the original image.
- **ACR (Azure Container Registry):** Microsoft's managed service for storing and managing private Docker images.

## Essential Docker Commands

```bash
# Build image from Dockerfile in current directory
docker build -t myservice:latest .

# Run container locally on port 8002
docker run -d -p 8002:8002 myservice:latest

# Run with environment variable
docker run -d -p 8002:8002 -e ENVIRONMENT=Development myservice:latest

# Push to Azure Container Registry
docker push myregistry.azurecr.io/myservice:latest

# Pull from registry
docker pull myregistry.azurecr.io/myservice:latest
```

## Standard Dockerfile Structure

```dockerfile
FROM python:3.11                          # Base Python 3.11 image
WORKDIR /code                             # Set working directory

COPY lib/requirements.txt /code/requirements.txt
RUN pip install --no-cache-dir -r requirements.txt \
    && pip install internal-utils==x.y.z  # Internal utility library

COPY src/loop_log.ini /code/             # Copy logging config
COPY . /code                             # Copy all service files

EXPOSE 8002                               # Document listening port

ENV FLASK_APP=src.aiservice
ENV FLASK_RUN_HOST=0.0.0.0
ENV FLASK_RUN_PORT=8002
ENV FLASK_ENV=production

CMD ["flask", "run"]                      # Start Flask server
```

## Manifest Engine Deployment Flow

When you click Deploy in Infra Hub:

1. **Build Image:** Converts the ZIP from Artifactory into a Docker image using the Dockerfile in `scripts/`
2. **Push to ACR:** Uploads the image to Azure Container Registry under a predictable naming scheme
3. **Deploy to AKS:** Spins up the container on Azure Kubernetes Service, wiring in the correct HTTP route (`/datafactory/{component}/{serviceName}`) from the manifest's component field
