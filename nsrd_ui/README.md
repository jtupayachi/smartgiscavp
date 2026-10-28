# NSRD UI - Live App Generator 🚀

A dockerized React application that generates live applications from OpenAPI specifications using AI models.

## Features

- 📤 **File Upload**: Drag & drop or browse OpenAPI specifications (JSON, YAML, TXT)
- 🤖 **Model Selection**: Choose from multiple AI models (GPT-4, Claude, Gemini, Llama, DeepSeek, etc.)
- 🖥️ **Live Preview**: View generated applications in real-time through an iframe
- 🐳 **Dockerized**: Fully containerized for easy deployment
- ⚡ **Development Mode**: Hot-reload enabled for rapid development

## Local deployment with vLLM via LiteLLM

Release 2 calls a vLLM server through a LiteLLM proxy (OpenAI-compatible `/v1/chat/completions`) instead of Ollama. `docker-compose.yml` joins the external docker network of the LiteLLM container (`local-gateway_default`; change the name to match your gateway). Default model: `ornl-qwen3-8-27b` (Qwen3.8-27B served by vLLM):

```bash
cp .env.litellm.example .env.litellm   # set OPENAI_API_KEY to the LiteLLM key (chmod 600, never commit)
docker compose up -d --build           # UI + API on http://localhost:8432
docker compose restart nsrd-ui         # after editing server.js / pipeline.js (bind-mounted)
docker compose up -d --build -V        # after dependency changes (refreshes the node_modules volumes)
```

Generated projects use the named volume `projects_data`. Set `NSRD_PROJECTS_DIR=/host/path` in `.env` to bind-mount a host directory instead. Container logs are capped at 5 × 50 MB.

Relevant variables: `LLM_PROVIDER=openai` (default; `ollama` restores the release-1 Ollama backend), `OPENAI_BASE_URL=http://litellm:4000/v1`, `LLM_DEFAULT_MODEL=ornl-qwen3-8-27b`,
`LLM_THINKER_ENABLE_THINKING` and `LLM_CODER_ENABLE_THINKING` (Qwen3 `enable_thinking`).

Experiment support (scaled-up evaluation, https://huggingface.co/datasets/jtupayac/smartgiscavp-scaled-experiment): the pipeline accepts an OpenAPI spec as context, and generated apps may import `@turf/turf`. Each repair round is snapshotted to `projects/_rounds/<id>/r<k>` and served at `/preview/<id>--r<k>/`. Telemetry is available at `GET /api/pipeline/telemetry/<id>`, round sources at `GET /api/pipeline/rounds/<id>/<k>/src`, and a fixture-driven mock REST API at `/mock/<fixture>/*` (fixtures are registered via `/api/mock/fixtures`).

## Project Structure

```
nsrd_ui/
├── public/                 # Static files
│   ├── index.html
│   └── manifest.json
├── src/
│   ├── components/        # React components
│   │   ├── FileUpload.tsx      # File upload with drag & drop
│   │   ├── FileUpload.css
│   │   ├── ModelSelector.tsx   # AI model selector
│   │   ├── ModelSelector.css
│   │   ├── AppPreview.tsx      # Iframe preview component
│   │   └── AppPreview.css
│   ├── App.tsx            # Main application
│   ├── App.css
│   ├── index.tsx          # Entry point
│   └── index.css
├── Dockerfile             # Production build
├── Dockerfile.dev         # Development build
├── docker-compose.yml     # Docker orchestration
├── nginx.conf            # Nginx configuration
├── package.json          # Dependencies
└── tsconfig.json         # TypeScript configuration
```

## Prerequisites

- Docker and Docker Compose
- (Optional) Node.js 18+ for local development

## Quick Start with Docker

### Production Mode

1. **Build and start the container:**
   ```bash
   docker-compose up --build
   ```

2. **Access the application:**
   Open your browser at `http://localhost:3000`

3. **Stop the container:**
   ```bash
   docker-compose down
   ```

### Development Mode (with hot reload)

1. **Start development server:**
   ```bash
   docker-compose --profile dev up nsrd-ui-dev
   ```

2. **Access the application:**
   Open your browser at `http://localhost:3001`

3. **Make changes:**
   Edit files in `src/` and see changes reflected instantly!

## Local Development (without Docker)

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start development server:**
   ```bash
   npm start
   ```

3. **Build for production:**
   ```bash
   npm run build
   ```

## Usage

### 1. Upload OpenAPI Specification

- Drag and drop your OpenAPI spec file (JSON/YAML/TXT)
- Or click to browse and select a file
- The spec will be validated automatically

### 2. Select AI Model

Choose from available models:
- **GPT-4**: Most capable, best for complex tasks
- **GPT-3.5 Turbo**: Fast and efficient
- **Claude 3**: Anthropic's advanced model
- **Gemini Pro**: Google's multimodal AI
- **Llama 3**: Open source, powerful
- **DeepSeek Coder**: Specialized in code generation

### 3. Configure & Generate

- Set the port where your app will run (default: 8000)
- Click "Generate App" to create your application
- View the live preview in the iframe

## Docker Commands

```bash
cd /home/jose/nsrd_ornl/nsrd_ui && docker compose down && docker volume rm nsrd_ui_build_cache && docker compose up -d

```

## Configuration

### Environment Variables

Create a `.env` file in the root directory:

```env
REACT_APP_API_URL=http://localhost:8000
REACT_APP_DEFAULT_PORT=8000
```

### Port Configuration

Edit `docker-compose.yml` to change exposed ports:

```yaml
services:
  nsrd-ui:
    ports:
      - "3000:80"  # Change 3000 to your desired port
```

### Nginx Configuration

Modify `nginx.conf` to customize:
- Proxy settings
- Cache policies
- CORS headers
- API routing

## API Integration

To integrate with your backend API, modify the `handleGenerateApp` function in `src/App.tsx`:

```typescript
const handleGenerateApp = async () => {
  const response = await axios.post('/api/generate', {
    spec: uploadedSpec,
    model: selectedModel,
    port: appPort
  });
  
  setAppUrl(response.data.appUrl);
};
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose logs nsrd-ui

# Remove old containers and volumes
docker-compose down -v
docker-compose up --build
```

### Port already in use

```bash
# Find process using port 3000
lsof -ti:3000

# Kill the process
kill -9 $(lsof -ti:3000)
```

### Changes not reflecting in development

```bash
# Restart development container
docker-compose --profile dev restart nsrd-ui-dev

# Or rebuild
docker-compose --profile dev up --build nsrd-ui-dev
```

## Technologies Used

- **React 18** with TypeScript
- **Swagger Parser** for OpenAPI validation
- **js-yaml** for YAML parsing
- **Docker** for containerization
- **Nginx** for production serving

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with Docker
5. Submit a pull request

## License

Copyright © 2025 ORNL NSRD. All rights reserved.

## Support

For issues or questions, contact the NSRD team at ORNL.
