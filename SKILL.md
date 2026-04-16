---
name: clawfy-workflow-execution
description: >-
  Operate Clawfy workflows via API. Use when the user wants to list,
  execute, monitor, or get results from Clawfy workflows.
  Triggers: clawfy, workflow, execution, automation, DAG, image generation,
  video generation, audio generation, workflow studio, ejecutar workflow,
  automatizacion, generar imagen, generar video, estudio de workflows,
  ejecutar automatizacion, studio de/workflows, style system, brand style,
  colors brand, typography brand, crear style system, definir marca,
  configurar estilo, estilo de marca, brand identity, mi marca
metadata:
  tags: clawfy, workflow, automation, execution, DAG, content generation, style-system, brand
---

## When to use

Use this skill whenever you need to:
- List existing workflows in a Clawfy account
- Get details of a specific workflow
- Execute a workflow and monitor its progress
- Cancel a running workflow
- Get the results (outputs) of a completed execution
- Estimate the cost of running a workflow

## Configuration

Before using the Clawfy API, set up your credentials:

```bash
# Required
export CLAWFY_API_KEY="clawfy_test_xxx"

# Optional (defaults to http://localhost:3001)
export CLAWFY_BASE_URL="http://localhost:3001"
```

**Getting an API key:**
1. Open Clawfy Web at http://localhost:3000
2. Go to Settings → API Keys
3. Create a new key and copy it

**Authentication:**
All requests require the `x-api-key` header:
```
x-api-key: YOUR_API_KEY
```

---

## Quick Reference

Read these files for detailed coverage:

- [rules/api-workflows.md](rules/api-workflows.md) — Complete API reference
- [rules/style-systems.md](rules/style-systems.md) — Style System API reference
- [rules/best-practices.md](rules/best-practices.md) — Security and error handling

---

## Common Scenarios

Step-by-step guides:

- [rules/scenarios/scenario-list-workflows.md](rules/scenarios/scenario-list-workflows.md) — List all workflows
- [rules/scenarios/scenario-execute-workflow.md](rules/scenarios/scenario-execute-workflow.md) — Execute and monitor
- [rules/scenarios/scenario-manage-style-systems.md](rules/scenarios/scenario-manage-style-systems.md) — Manage brand styles

---

## Key Concepts

### Workflow Structure

A Clawfy workflow is defined in DSL format:

```
WORKFLOW my_pipeline
  STEP.1.TEXT_PROMPT prompt="A sunset"
  STEP.2.IMAGE_GEN model=minimax prompt="A sunset" aspect_ratio="16:9"
  STEP.3.OUTPUT type=image input=2
```

**Dynamic variables:** Use `{{varName}}` or `${varName}` placeholders in DSL, then pass `variables` at execution time:

```
WORKFLOW refine_image
  STEP.1.TEXT_PROMPT prompt={{prompt}}
  STEP.2.SAMPLER prompt=${prompt} denoise=0.7 init_image=${prev_image} aspect_ratio=${aspect_ratio}
  STEP.3.OUTPUT type=image input=2
```

Execute with: `{ "variables": { "prompt": "sunset over Tokyo", "prev_image": "https://...", "aspect_ratio": "16:9" } }`

### Node Types

| Node Type | Purpose |
|-----------|---------|
| `TEXT_PROMPT` | Text input |
| `SAMPLER` | Unified image generation (txt2img, img2img, inpaint) — **preferred over IMAGE_GEN** |
| `IMAGE_GEN` | Image generation (txt2img) — backward compatible wrapper around SAMPLER |
| `IMAGE_EDIT` | Image editing (img2img) — backward compatible wrapper |
| `UPSCALE` | Resolution upscaling — backward compatible wrapper |
| `STYLE_TRANSFER` | Style transfer — backward compatible wrapper |
| `VIDEO_GEN` | Video generation (Kling/Veo stubs) |
| `AUDIO_GEN` | Audio generation (ElevenLabs stub) |
| `PASSTHROUGH` | Data passthrough |
| `OUTPUT` | Final output |

### SAMPLER — Unified Image Node

SAMPLER handles all image generation types. Parameters determine behavior:

| Parameters | Operation |
|-----------|-----------|
| `denoise=1.0`, no `init_image` | **txt2img** — generate from noise |
| `denoise<1.0` + `init_image` | **img2img** — transform existing image |
| `mask` + `init_image` | **inpaint** — regenerate masked regions only |
| `denoise=0` + `init_image` | **passthrough** — return image unchanged |

Key parameters: `prompt`, `init_image`, `denoise` (0.0-1.0), `mask`, `steps`, `cfg`, `seed`, `model`

### Execution States

| State | Meaning | Next Action |
|-------|---------|-------------|
| `queued` | Waiting in queue | Continue polling |
| `running` | Currently executing | Continue polling |
| `completed` | Successfully finished | Get results |
| `failed` | Error occurred | Report error to user |
| `cancelled` | Manually cancelled | Notify user |

---

## Style System (Brand Identity)

The Style System lets you define brand identities — colors, typography, logo, mood, and layout preferences — and reuse them across all image generations for consistent brand output.

### When to use

Use the Style System when:
- User wants to define or manage brand identity (colors, fonts, logo)
- User references "my brand", "brand style", "company colors", "estilo de marca"
- User wants to create content with consistent brand look-and-feel
- Setting up image generation for a specific client or business
- User says "crear style system", "definir marca", "configurar estilo"

### What you can do

- List all Style Systems owned by the user
- Create a new Style System with brand parameters
- Update an existing Style System
- Delete a Style System

### Style System Data Model

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable name (e.g. "Mercury Hub Brand") |
| `brandName` | string | Brand/business name |
| `industry` | string | e.g. tech, beauty, food, fashion |
| `colors` | object | `{ primary, secondary, accent, background, text }` — hex values |
| `typography` | object | `{ headlineFont, headlineWeight, bodyFont, bodyWeight }` |
| `logo` | object | `{ url, position, maxWidth }` |
| `layoutPreference` | string | e.g. product_hero, split_50_50, text_heavy |
| `mood` | string | e.g. premium, casual, bold, minimal, friendly |
| `stylePreset` | string | Visual family: clean_tech, clean_beauty, premium_luxury, streetwear, wellness_natur, bold_color |
| `customInstructions` | string | Free-form design notes for the agent |
| `examples` | string[] | URLs to reference images the brand likes |

### Style Presets

| Preset | Description |
|--------|-------------|
| `clean_tech` | Minimalist tech aesthetic, soft studio light, white/bold accents |
| `clean_beauty` | Skincare/cosmetic aesthetic, white backgrounds, elegant serif |
| `premium_luxury` | Dark backgrounds, gold accents, dramatic lighting |
| `streetwear` | Bold colors, dynamic angles, high contrast, urban feel |
| `wellness_natur` | Earth tones, natural lighting, organic layouts |
| `bold_color` | Vibrant solid backgrounds, strong color blocks, high impact |

### How it works

1. **Create** a Style System with brand parameters via `POST /api/style-systems`
2. **Reference** it by ID when executing image workflows (future integration)
3. The system **enriches** the generation prompt with style parameters
4. Result: consistent brand output without repeating design specs every time

### Example: Creating a Style System

```
User: "Quiero crear un estilo de marca para Mercury Hub"

Agent: "Claro. Necesito los detalles de tu marca:

1. **Nombre del style system**: ¿Cómo lo querés llamar?
2. **Nombre de la marca**: ¿Cuál es el nombre del negocio?
3. **Industria**: ¿Tech, belleza, food, fashion?
4. **Colores principales**: Dame los hex (ej: #1A7F37)
5. **Tipografía**: ¿Qué fuentes usás?
6. **Mood**: ¿Premium, casual, bold, minimal?
7. **Preset** (opcional): ¿Alguna familia visual?

Agent luego hace:
POST /api/v1/style-systems
{
  "name": "Mercury Hub Brand Style",
  "brandName": "Mercury Hub",
  "industry": "tech",
  "colors": { "primary": "#1A7F37", "secondary": "#2D3748", "accent": "#38A169", "background": "#FFFFFF", "text": "#1A202C" },
  "mood": "premium",
  "stylePreset": "clean_tech"
}
```

---

## Important Rules

1. **Always use polling** — after executing, poll every 5 seconds until terminal state
2. **Never stop polling early** — your obligation is to follow through until completion
3. **Deliver results directly** — when completed, present the outputs to the user, not just URLs
4. **Use environment variables** — never hardcode API keys in logs or responses
5. **HTTPS in production** — always use HTTPS when `CLAWFY_BASE_URL` points to production

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `CLAWFY_API_KEY` | Yes | — | Your Clawfy API key |
| `CLAWFY_BASE_URL` | No | `http://localhost:3001` | Clawfy API base URL |