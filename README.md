# Docker LLM Environment

A Docker-based setup for running local Large Language Models using Ollama with a web interface via Open WebUI.

## Prerequisites

- Docker and Docker Compose installed on your system
- NVIDIA GPU with Docker GPU support (optional, for hardware acceleration)
- At least 8GB of free disk space for models

## System Components

- **Ollama**: Local LLM inference server
- **Open WebUI**: Web-based chat interface for interacting with models
- **Persistent Storage**: Data persistence for models and chat history

## Quick Start

### 1. Clone and Navigate to the Repository

```bash
git clone <repository-url>
cd Docker_LLM_environment
```

### 2. Prepare Your Models

Place your GGUF model files in the `LLM_models/` directory. For example:
```
LLM_models/
├── llama-3.2-3b-instruct-q4_k_m.gguf
└── other-model.gguf
```

### 3. Start the Services

```bash
docker-compose up -d
```

This will:
- Pull the necessary Docker images
- Start Ollama on port 11434
- Start Open WebUI on port 3000
- Mount persistent volumes for data storage

### 4. Create Custom Models (Optional)

If you have custom Modelfiles, you can create models using them:

```bash
# Copy the Modelfile to the container
docker cp ./your-model.Modelfile ollama:/tmp/Modelfile

# Create the model
docker exec ollama ollama create your-model-name -f /tmp/Modelfile
```

Example for the included Llama 3.2 3B model:
```bash
docker cp .\llama3-3b-manual.Modelfile ollama:/tmp/Modelfile
docker exec ollama ollama create llama3-3b-manual-chat -f /tmp/Modelfile
```

### 5. Access the Web Interface

Open your browser and navigate to:
```
http://localhost:3000
```

## Usage

### Downloading Models via Ollama

You can download models directly through Ollama:

```bash
# Download a model (example: Llama 2 7B)
docker exec ollama ollama pull llama2:7b

# List available models
docker exec ollama ollama list

# Run a model directly via CLI
docker exec -it ollama ollama run llama2:7b
```

### Managing the Environment

```bash
# Stop all services
docker-compose down

# View logs
docker-compose logs

# Restart services
docker-compose restart

# Update images
docker-compose pull
docker-compose up -d
```

## Directory Structure

```
Docker_LLM_environment/
├── docker-compose.yml          # Main configuration
├── llama3-3b-manual.Modelfile  # Example Modelfile
├── LLM_models/                 # Your local model files
├── ollama-data/                # Ollama persistent data (auto-created)
└── open-webui-data/           # Open WebUI persistent data (auto-created)
```

## Configuration

### GPU Support

The configuration includes NVIDIA GPU support. If you don't have an NVIDIA GPU, you can remove or comment out the GPU-related sections in `docker-compose.yml`:

```yaml
# Remove or comment these lines if no GPU
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: all
          capabilities: [gpu]
```

### Port Configuration

- **Ollama API**: Port 11434 (can be changed in docker-compose.yml)
- **Open WebUI**: Port 3000 (maps to container port 8080)

### Environment Variables

The Open WebUI connects to Ollama using:
```yaml
environment:
  - OLLAMA_BASE_URL=http://ollama:11434
```

## Troubleshooting

### Common Issues

1. **Model not found**: Ensure your model files are in the `LLM_models/` directory and the container has been restarted after adding them.

2. **GPU not recognized**: Verify NVIDIA Docker runtime is installed:
   ```bash
   docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
   ```

3. **Permission issues**: Ensure the mounted directories have proper permissions.

4. **Port conflicts**: If ports 3000 or 11434 are in use, modify the port mappings in `docker-compose.yml`.

### Logs and Debugging

```bash
# View all service logs
docker-compose logs

# View specific service logs
docker-compose logs ollama
docker-compose logs open-webui

# Check container status
docker-compose ps
```

## Creating Custom Models

### Modelfile Format

Create a `.Modelfile` with your model configuration:

```dockerfile
FROM /LLM_models_host/your-model.gguf
TEMPLATE """{{ if .System }}<|start_header_id|>system<|end_header_id|>\n{{ .System }}<|eot_id|>\n{{ end }}<|start_header_id|>user<|end_header_id|>\n{{ .Prompt }}<|eot_id|>\n<|start_header_id|>assistant<|end_header_id|>\n{{ .Response }}<|eot_id|>"""
PARAMETER stop "<|start_header_id|>"
PARAMETER stop "<|end_header_id|>"
PARAMETER stop "<|eot_id|>"
```

### Loading the Model

```bash
docker cp ./your-custom.Modelfile ollama:/tmp/Modelfile
docker exec ollama ollama create your-model-name -f /tmp/Modelfile
```

## Data Persistence

All data is persisted in local directories:
- `./ollama-data/` - Downloaded models and Ollama configuration
- `./open-webui-data/` - Chat history and Open WebUI settings
- `./LLM_models/` - Your custom model files

These directories are excluded from git via `.gitignore`.

## Security Considerations

- The services are exposed only on localhost by default
- Consider using reverse proxy with SSL for production deployments
- Regularly update Docker images for security patches

## Support

For issues and questions:
1. Check the logs using `docker-compose logs`
2. Verify all prerequisites are met
3. Ensure sufficient disk space and memory
4. Check the official documentation for [Ollama](https://ollama.ai/) and [Open WebUI](https://github.com/open-webui/open-webui)
