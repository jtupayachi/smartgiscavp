<div align="center">

# SmartGIScavp

### AI-Powered Multi-Page React App Generator

**Write your requirements → LLM thinks + codes → Live React app deployed in seconds**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![React](https://img.shields.io/badge/React-18-61dafb?logo=react)](https://reactjs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-4.9-3178c6?logo=typescript)](https://typescriptlang.org)
[![vLLM](https://img.shields.io/badge/vLLM-OpenAI--compatible-30a2ff)](https://github.com/vllm-project/vllm)
[![Release](https://img.shields.io/badge/release-2.0-success)](#release-2-vllm-backend)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-jtupayac%2Fsmartgiscavp-2496ed?logo=docker)](https://hub.docker.com/r/jtupayac/smartgiscavp)
[![ORNL](https://img.shields.io/badge/Oak%20Ridge-National%20Laboratory-00629b)](https://ornl.gov)

</div>

---

![SmartGIScavp UI](docs/screenshots/ui-main.png)

---

## What is SmartGIScavp?

SmartGIScavp is a full-stack app builder that lets you **describe** what you want, pick an LLM, and watch a production-quality multi-page React app get generated, built, and served — live — in your browser.

It uses a **Thinker → Coder** dual-model pipeline:

```
Your description
      ↓
  [Thinker LLM]  ← plans architecture, reasons about structure
      ↓
  [Coder LLM]   ← writes all React/JSX files
      ↓
  Vite build    ← compiles & bundles
      ↓
  Runtime review ← LLM checks for crashes, fixes them
      ↓
  Live iframe preview + download
```

Page types supported:
- **Home** — hero banners, metric cards, CTAs
- **Base** — data tables, charts, dashboards  
- **Geo / Map** — Leaflet maps with CSV data, heatmaps, polylines

---

## Release 2: vLLM backend

Release 2 replaces the Ollama calls of release 1 with an **OpenAI-compatible backend**: every Thinker, Coder, fix, and review call goes to `/v1/chat/completions`, served by **vLLM** behind a **LiteLLM** proxy.

* Default model `ornl-qwen3-8-27b` (Qwen3.8-27B on vLLM) for all roles; Qwen3 `enable_thinking` is on for the Thinker and off for the Coder (`LLM_THINKER_ENABLE_THINKING`, `LLM_CODER_ENABLE_THINKING`).
* Per-run sampling overrides (seed, temperature, top-p) for reproducible experiments.
* Pages can carry an **OpenAPI 3.x spec** instead of a CSV; generated apps call the documented endpoints with `fetch`, and a fixture-driven mock REST API is served at `/mock/<fixture>/*`.
* **Turf.js 7.4.0** is pinned in the project template for client-side spatial analysis.
* Every Vite build is snapshotted as a repair round (`/preview/<id>--r<k>/`), with telemetry at `GET /api/pipeline/telemetry/<id>`.
* Ollama is still available as a legacy option: `LLM_PROVIDER=ollama`.

The scaled-up evaluation of this release (840 runs, design, scripts, and results) is at https://huggingface.co/datasets/jtupayac/smartgiscavp-scaled-experiment. See [`nsrd_ui/README.md`](nsrd_ui/README.md) for the vLLM/LiteLLM deployment.

---

## Quick Start

```bash
git clone https://github.com/jtupayachi/smartgiscavp_ui.git
cd smartgiscavp/smartgiscavp
cp .env.litellm.example .env.litellm   # set OPENAI_API_KEY (LiteLLM key), never commit
docker compose up --build
```

Open **http://localhost:8432** in your browser.

> **Port 8432** is exposed. Change it in `docker-compose.yml` → `ports: "XXXX:80"` if needed.

---

## Connecting the LLM backend (vLLM via LiteLLM, default)

```ini
# docker-compose.yml environment / .env
LLM_PROVIDER=openai                      # default in release 2
OPENAI_BASE_URL=http://litellm:4000/v1   # LiteLLM proxy in front of vLLM (any OpenAI-compatible server works)
LLM_DEFAULT_MODEL=ornl-qwen3-8-27b
# .env.litellm
OPENAI_API_KEY=sk-...                    # LiteLLM key
```

> **Verify the connection:** `curl -H "Authorization: Bearer $OPENAI_API_KEY" $OPENAI_BASE_URL/models`

### Legacy: Ollama server (release 1)

Set `LLM_PROVIDER=ollama` and configure the Ollama variables below.

All credentials live in **`.env`** — never hardcoded. Copy the template and fill in your values:

```bash
cp smartgiscavp/.env.example smartgiscavp/.env
```

```ini
# .env
OLLAMA_HOST=http://localhost:11434   # or https://your-remote-ollama.example.com
OLLAMA_USER=                         # leave blank for no-auth local installs
OLLAMA_PASSWORD=                     # or set OLLAMA_API_KEY for Bearer token auth
ANTHROPIC_API_KEY=                   # optional — enables Claude models
```

> **Verify your Ollama connection:**
> ```bash
> curl $OLLAMA_HOST/api/tags   # should return JSON with your models
> ollama pull qwen3-coder:30b  # download a model if needed
> ```

---

## Selecting Models in the UI

SmartGIScavp uses two roles:

| Role | Purpose | Recommended |
|---|---|---|
| **Thinker** | Plans architecture, reasons step-by-step | `qwen3-coder-next` · `qwen3:30b` · `llama3.2` |
| **Coder** | Writes all JSX/React files | `qwen3-coder:30b` · `codellama:34b` · `deepseek-coder:33b` |

**Thinker = bigger/smarter model.** Coder = fast code-focused model.

You can use the **same model for both** if compute is limited.

---


## Full Demo Walkthrough

1. **Open** http://localhost:8432
2. **Add a page** — click `+ Add Page` in the sidebar
3. **Name it** and pick a type (Home / Base / Geo / Map)
4. **Describe it** — write natural language requirements, e.g.:
   > *"A dashboard page with a bar chart of species counts by site, a filterable data table, and a header with the project title EcoWatch"*
5. **Upload an SVG mockup** (optional) — draw a wireframe and the LLM will follow it
6. **Upload CSV data** (optional, for Geo pages) — lat/lng columns are detected automatically
7. **Select models** — pick your Thinker and Coder models from the dropdowns
8. **Click Generate & Deploy** — watch the pipeline run in the right panel
9. **Live preview** — the app appears in the iframe when complete
10. **Download** — get a `.tar.gz` of the full source (`npm install && npm run dev` to run it)

---

## Project Structure

```
smartgiscavp/
├── smartgiscavp/                   ← Main application (git root)
│   ├── src/                   ← React frontend (TypeScript)
│   │   ├── App.tsx            ← Main UI component
│   │   ├── App.css            ← Swiss brutalist theme
│   │   └── components/        ← FileUpload, SVGEditor, ModelSelector, TabHealthBadges
│   ├── server.js              ← Express backend + LLM proxy (vLLM/LiteLLM or Ollama)
│   ├── pipeline.js            ← Thinker→Coder→Build pipeline
│   ├── sanitizer.js           ← LLM output validation & repair
│   ├── referenceAnalyzer_rag.js ← RAG retrieval from golden examples
│   ├── rag_server.py          ← Python FAISS + sentence-transformers
│   ├── templates/react-app/   ← Vite template for generated apps
│   ├── reference-codebases/   ← Golden example components for RAG
│   ├── Dockerfile
│   └── docker-compose.yml
├── docs/
│   ├── screenshots/
│   └── deployment/
└── README.md
```

---

## RAG-Assisted Code Generation

SmartGIScavp includes a **FAISS-based RAG engine** that retrieves relevant golden-example components before generating code. This dramatically improves code quality for charts, maps, and data tables.

The Python microservice runs inside Docker automatically:

```bash
# Manually rebuild the RAG index:
docker exec smartgiscavp python3 rag_server.py
```

Golden examples live in `reference-codebases/golden-examples/src/` — add your own `.jsx` files there to extend what the LLM can reference.

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `LLM_PROVIDER` | `openai` | `openai` (vLLM/LiteLLM, release 2) or `ollama` (legacy) |
| `OPENAI_BASE_URL` | `http://litellm:4000/v1` | OpenAI-compatible endpoint |
| `OPENAI_API_KEY` | *(blank)* | Key of the OpenAI-compatible proxy |
| `LLM_DEFAULT_MODEL` | `ornl-qwen3-8-27b` | Model used for all roles |
| `LLM_THINKER_ENABLE_THINKING` / `LLM_CODER_ENABLE_THINKING` | `true` / `false` | Qwen3 thinking mode per role |
| `NSRD_PROJECTS_DIR` | named volume | Host directory for generated projects |
| `OLLAMA_HOST` | `http://localhost:11434` | Ollama server URL (legacy) |
| `OLLAMA_USER` | *(blank)* | Basic auth username |
| `OLLAMA_PASSWORD` | *(blank)* | Basic auth password |
| `OLLAMA_API_KEY` | *(blank)* | Bearer token (alternative to basic auth) |
| `ANTHROPIC_API_KEY` | *(blank)* | Enable Anthropic Claude |
| `PORT` | `80` | Express backend port |
| `VIRTUAL_HOST` | *(blank)* | nginx-proxy reverse-proxy hostname |

See `smartgiscavp/.env.example` for a full annotated template.

---

## Deployment

### Behind a reverse proxy (nginx-proxy)

```yaml
# docker-compose.yml
environment:
  - VIRTUAL_HOST=your-domain.example.com
  - LETSENCRYPT_HOST=your-domain.example.com
```

### Pull from Docker Hub

```bash
docker pull jtupayac/smartgiscavp
docker run -p 8432:80 \
  -e OPENAI_BASE_URL=http://your-litellm:4000/v1 \
  -e OPENAI_API_KEY=your_key \
  -e LLM_DEFAULT_MODEL=ornl-qwen3-8-27b \
  jtupayac/smartgiscavp
```

---

## Architecture

```
Browser
  │
  ├─ React 18 (TypeScript) — SmartGIScavp UI
  │     └─ src/App.tsx · components/
  │
  └─ Express.js server (server.js)
        ├─ POST /api/pipeline/run     → start generation job
        ├─ GET  /api/pipeline/stream/:id → SSE event stream
        ├─ GET  /api/models           → OpenAI-compatible /v1/models (Ollama /api/tags in legacy mode)
        ├─ POST /api/pipeline/clarify → answer model questions
        └─ GET  /preview/:id/         → serve generated app
              │
              ├─ pipeline.js
              │     ├─ [Thinker LLM] architectural reasoning
              │     ├─ [Coder LLM]   JSX code generation
              │     ├─ Vite build
              │     └─ Runtime review + auto-fix
              │
              ├─ referenceAnalyzer_rag.js ← RAG lookups
              │
              └─ rag_server.py (FastAPI + FAISS)
                    └─ sentence-transformers embeddings
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 · TypeScript · IBM Plex Mono · Playfair Display |
| Backend | Node.js · Express.js · SSE streaming |
| LLM | vLLM + LiteLLM (OpenAI-compatible) · Ollama (legacy) · Anthropic Claude · LangChain.js |
| Build | Vite · Tailwind CSS (generated apps) |
| RAG | Python · FastAPI · FAISS · sentence-transformers |
| Deploy | Docker · nginx |

---

## Sponsorship & Collaboration

SmartGIScavp is an active R&D project at **Oak Ridge National Laboratory** exploring the intersection of large language models, geospatial computing, and scientific app generation.

We are seeking **DOE program sponsors and research partners** to scale this work. Potential collaboration areas include:

- 🏛️ **DOE Office of Science** — integrating SmartGIScavp into scientific data workflows and user facilities
- 🗺️ **Geospatial intelligence** — extending multi-page app generation to real-time sensor and satellite data
- 🤖 **LLM infrastructure** — co-developing domain-specific fine-tuned models for scientific code generation
- 🔒 **Secure deployments** — air-gapped, on-premise builds for classified or sensitive environments

If your program could benefit from AI-assisted rapid application development for scientific data, we would love to connect.

| Contact | Link |
|---|---|
| **Xiao-Ying Yu, Ph.D.** — PI, ORNL | [Staff Profile](https://www.ornl.gov/staff-profile/xiao-ying-yu) |
| **Jose Tupayachi** — Developer | [jtupayachi.github.io](https://jtupayachi.github.io/) |
| **Repository** | [github.com/jtupayachi/smartgiscavp](https://github.com/jtupayachi/smartgiscavp) |
| **Container** | [hub.docker.com/r/jtupayac/smartgiscavp](https://hub.docker.com/r/jtupayac/smartgiscavp) |

---

## License

MIT © Oak Ridge National Laboratory

---

<div align="center">

| | |
|:---:|:---:|
| **PI** | [Xiao-Ying Yu, Ph.D.](https://www.ornl.gov/staff-profile/xiao-ying-yu) · Oak Ridge National Laboratory |
| **Dev** | [Jose Tupayachi](https://jtupayachi.github.io/) · UTK / ORNL |
| **Container** | [`docker pull jtupayac/smartgiscavp`](https://hub.docker.com/r/jtupayac/smartgiscavp) |

</div>
