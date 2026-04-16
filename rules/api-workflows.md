# Clawfy API — Complete Workflow Reference

> Base URL: `$CLAWFY_BASE_URL` (default: `http://localhost:3001`)
> All requests require: `x-api-key: $CLAWFY_API_KEY`
> Content-Type: `application/json` for all requests
> **IMPORTANT:** This API uses `/api/v1/` prefix (NOT `/api/`)

---

## 1. Health Check

### GET /api/v1/health

Verify the API is running and healthy.

**Request:**
```bash
curl -s http://localhost:3001/api/v1/health \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "status": "ok",
  "db": "connected",
  "timestamp": "2026-04-10T12:00:00.000Z",
  "version": "1.0.0"
}
```

---

## 2. List Workflows

### GET /api/v1/workflows

Returns all workflows for the authenticated user.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `limit` | number | No | Max results (default: 20) |
| `offset` | number | No | Skip for pagination |

**Request:**
```bash
curl -s "http://localhost:3001/api/v1/workflows?limit=10" \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "workflows": [
    {
      "id": "wf_abc123",
      "name": "Product Photography",
      "dsl": "WORKFLOW product_photo\n  STEP.1.TEXT_PROMPT prompt=\"product shot\"\n  STEP.2.IMAGE_GEN model=minimax\n  STEP.3.OUTPUT type=image input=2",
      "status": "active",
      "tags": ["ecommerce", "product"],
      "createdAt": "2026-04-01T10:00:00.000Z",
      "updatedAt": "2026-04-05T15:30:00.000Z"
    }
  ]
}
```

---

## 3. Get Workflow Details

### GET /api/v1/workflows/:id

Get full details of a specific workflow including its graph structure.

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Workflow ID |

**Request:**
```bash
curl -s http://localhost:3001/api/v1/workflows/wf_abc123 \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "workflow": {
    "id": "wf_abc123",
    "name": "Product Photography",
    "dsl": "WORKFLOW product_photo\n  STEP.1.TEXT_PROMPT prompt=\"product shot\"\n  STEP.2.IMAGE_GEN model=minimax\n  STEP.3.OUTPUT type=image input=2",
    "version": 1,
    "graph": {
      "nodes": [
        { "id": "1", "type": "TEXT_PROMPT", "data": { "prompt": "product shot" } },
        { "id": "2", "type": "IMAGE_GEN", "data": { "model": "minimax" } },
        { "id": "3", "type": "OUTPUT", "data": { "type": "image", "input": "2" } }
      ],
      "edges": [
        { "id": "e1", "source": "1", "target": "2" },
        { "id": "e2", "source": "2", "target": "3" }
      ]
    },
    "status": "active",
    "createdAt": "2026-04-01T10:00:00.000Z",
    "updatedAt": "2026-04-05T15:30:00.000Z"
  }
}
```

---

## 4. Create Workflow

### POST /api/v1/workflows

Create a new workflow from DSL string.

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Workflow name |
| `dsl` | string | Yes | Workflow DSL definition |
| `tags` | string[] | No | Tags for organization |

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/workflows \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My New Workflow",
    "dsl": "WORKFLOW my_workflow\n  STEP.1.TEXT_PROMPT prompt=\"hello\"\n  STEP.2.IMAGE_GEN model=minimax\n  STEP.3.OUTPUT type=image input=2",
    "tags": ["test"]
  }'
```

**Response:**
```json
{
  "workflow": {
    "id": "wf_new123",
    "name": "My New Workflow",
    "dsl": "WORKFLOW my_workflow...",
    "status": "draft",
    "createdAt": "2026-04-10T12:00:00.000Z"
  }
}
```

---

## 5. Execute Workflow

### POST /api/v1/workflows/:id/execute

Start a workflow execution by workflow ID. Returns immediately with an `executionId` for polling.

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Workflow ID to execute |

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/workflows/wf_abc123/execute \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "executionId": "D8IaCuxM2feVFl2GRXTZW",
  "status": "queued",
  "priority": 2,
  "websocketUrl": "ws://localhost:3001/ws?executionId=D8IaCuxM2feVFl2GRXTZW"
}
```

**Important fields:**
- `executionId` — use this for polling status (GET /api/v1/executions/:id)
- `websocketUrl` — optional WebSocket for real-time updates (see WebSocket section)
- `priority` — execution priority (0=low, 1=normal, 2=high)

**Optional body params** (JSON body):
| Field | Type | Description |
|-------|------|-------------|
| `seed` | number | Random seed for reproducibility |
| `priority` | string | `high`, `normal`, or `low` |
| `quantization` | string | `fp32`, `fp16`, `int8`, `int4` |
| `vramBudget` | number | VRAM budget in MB |
| `variables` | object | Record<string, string> — replaces `{{var}}` or `${var}` placeholders in DSL at execution time |
| `styleSystemId` | string | Style System ID to enrich prompts with brand identity |

**Variables — Iterative Refinement:**

Use `variables` to pass dynamic values at execution time. This lets you reuse the same workflow for different inputs without creating new workflows:

```bash
# Workflow with {{prompt}} placeholder in DSL
# Execute with different variable values to refine results
curl -s -X POST http://localhost:3001/api/v1/workflows/wf_abc123/execute \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "prompt": "sunset over Tokyo with warm orange colors",
      "aspect_ratio": "16:9"
    }
  }'
```

The DSL can use both `{{varName}}` and `${varName}` syntax:

```
WORKFLOW refine_image
  STEP.1.TEXT_PROMPT prompt={{prompt}}
  STEP.2.IMAGE_GEN model=minimax prompt=${prompt} aspect_ratio=${aspect_ratio}
  STEP.3.OUTPUT type=image input=2
```

Execute with `variables: { "prompt": "...", "aspect_ratio": "16:9" }` — the placeholders get replaced at runtime. Re-execute with different variables to iterate without creating new workflows.

---

## 5b. SAMPLER Node — Unified Image Generation

The `SAMPLER` node is a unified node that handles all image generation types (txt2img, img2img, inpaint, upscale). IMAGE_GEN, IMAGE_EDIT, UPSCALE, and STYLE_TRANSFER are backward-compatible wrappers around SAMPLER.

### SAMPLER Parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model` | string | `minimax` | Provider model: `minimax`, `replicate` |
| `prompt` | string | — | **Required.** Text prompt |
| `negative` | string | — | Negative prompt |
| `init_image` | string | — | URL of image to transform (img2img/inpaint) |
| `denoise` | number | `1.0` | 0.0-1.0, strength of transformation |
| `mask` | string | — | URL of mask image (white=keep, black=regenerate) for inpaint |
| `steps` | number | — | Denoising steps |
| `cfg` | number | — | Guidance scale |
| `seed` | number | — | Random seed |
| `aspect_ratio` | string | `1:1` | `1:1`, `16:9`, `9:16`, `3:2`, `2:3` |

### Operation Routing:

| Parameters | Operation |
|-----------|-----------|
| `denoise=1.0`, no `init_image` | **txt2img** — generate from pure noise |
| `denoise<1.0` + `init_image` | **img2img** — transform existing image |
| `mask` + `init_image` | **inpaint** — regenerate masked regions only |
| `denoise=0` + `init_image` | **passthrough** — return init_image unchanged |

### DSL Examples:

```bash
# txt2img (generate from scratch)
WORKFLOW generate
  STEP.1.SAMPLER prompt="a red car" denoise=1.0 model=minimax
  STEP.2.OUTPUT type=image input=1

# img2img (transform existing image)
WORKFLOW refine
  STEP.1.SAMPLER prompt="more vibrant colors" denoise=0.7 init_image="${last.1.image_url}" model=minimax
  STEP.2.OUTPUT type=image input=1

# inpaint (edit specific regions)
WORKFLOW inpaint_face
  STEP.1.SAMPLER prompt="blue eyes" denoise=0.8 init_image="${last.1.image_url}" mask="https://example.com/mask.png" model=replicate
  STEP.2.OUTPUT type=image input=1
```

### Backward Compatibility:

IMAGE_GEN, IMAGE_EDIT, UPSCALE, and STYLE_TRANSFER still work exactly as before — they are SAMPLER with different defaults:

- `IMAGE_GEN` = SAMPLER with `denoise=1.0` (txt2img)
- `IMAGE_EDIT` = SAMPLER with `denoise=0.7` (img2img)
- `UPSCALE` = SAMPLER with `denoise=0` (passthrough)
- `STYLE_TRANSFER` = SAMPLER with `denoise=0.8`

---

## 5c. Execution History DSL — Reference Previous Executions

Use the `executionHistory` DSL patterns to reference outputs from previous workflow executions:

### History Patterns:

| Pattern | Description |
|---------|-------------|
| `${last.nodeId.field}` | Last execution's node output (shorthand) |
| `${history[-1].nodeId.field}` | Last execution (explicit) |
| `${history[-2].nodeId.field}` | Second-to-last execution |
| `${history[-n].nodeId.field}` | Nth-from-last execution |
| `${exec.EXEC_ID.nodeId.field}` | Specific execution by ID |

### Examples:

```bash
# Use image from last execution
init_image="${last.2.image_url}"

# Use image from 2 executions ago
init_image="${history[-2].2.image_url}"

# Reference a specific execution
init_image="${exec.kUxmsRvf-_h7YC3q3fLZg.2.image_url}"
```

### How It Works:

- The system fetches the last 10 completed executions for this workflow
- `history[-1]` is the most recent, `history[-2]` is one before that, etc.
- If the referenced execution or node doesn't exist, the placeholder is left unchanged (no crash)
- Available after at least one successful execution

### Combined with Variables:

```bash
# Reference last execution AND use variables
STEP.1.SAMPLER prompt="{{prompt}}" denoise=0.7 init_image="${history[-1].2.image_url}" model=minimax
```

Execute with: `{ "variables": { "prompt": "add sunset colors" } }`

---

## 6. Get Execution Status

### GET /api/v1/executions/:id

Poll for execution status. Do this every 5 seconds until terminal state.

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Execution ID returned from execute |

**Request:**
```bash
curl -s http://localhost:3001/api/v1/executions/exec_xyz789 \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response (queued/running):**
```json
{
  "execution": {
    "id": "exec_xyz789",
    "workflowId": "wf_abc123",
    "status": "running",
    "progress": 45,
    "queuedAt": "2026-04-10T12:05:00.000Z",
    "startedAt": "2026-04-10T12:05:30.000Z"
  }
}
```

**Response (completed):**
```json
{
  "execution": {
    "id": "exec_xyz789",
    "workflowId": "wf_abc123",
    "status": "completed",
    "progress": 100,
    "outputs": {
      "3": "https://cdn.clawfy.io/outputs/img_001.png"
    },
    "completedAt": "2026-04-10T12:07:00.000Z"
  }
}
```

**Response (failed):**
```json
{
  "execution": {
    "id": "exec_xyz789",
    "workflowId": "wf_abc123",
    "status": "failed",
    "error": "IMAGE_GEN model unavailable",
    "failedAt": "2026-04-10T12:06:00.000Z"
  }
}
```

---

## 7. Cancel Execution

### POST /api/v1/executions/:id/cancel

Cancel a running or queued execution.

**Path Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Execution ID |

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/executions/exec_xyz789/cancel \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "cancelled": true,
  "executionId": "exec_xyz789",
  "cancelledAt": "2026-04-10T12:06:30.000Z"
}
```

---

## 8. List Templates

### GET /api/v1/templates

Get available workflow templates.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category` | string | No | Filter by category |
| `limit` | number | No | Max results |

**Request:**
```bash
curl -s "http://localhost:3001/api/v1/templates?category=ecommerce" \
  -H "x-api-key: $CLAWFY_API_KEY"
```

**Response:**
```json
{
  "templates": [
    {
      "id": "tmpl_001",
      "name": "E-commerce Product Shot",
      "description": "Generate professional product photography",
      "category": "ecommerce",
      "dsl": "WORKFLOW product_photo\n  STEP.1.TEXT_PROMPT...\n  STEP.2.IMAGE_GEN...\n  STEP.3.OUTPUT...",
      "createdAt": "2026-03-15T10:00:00.000Z"
    }
  ]
}
```

---

## 9. Export Template

### GET /api/v1/templates/:id/export

Export a template as a `.clawfy-template.json` file for sharing or backup.

**Request:**
```bash
curl -s http://localhost:3001/api/v1/templates/tmpl_001/export \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -O
```

**Response:** Downloads a JSON file with `Content-Disposition: attachment` header.

**Response body:**
```json
{
  "version": "1.0",
  "exportedAt": "2026-04-15T12:00:00.000Z",
  "template": {
    "name": "E-commerce Product Shot",
    "description": "Generate professional product photography",
    "category": "ecommerce",
    "dsl": "WORKFLOW product_photo\n  STEP.1.TEXT_PROMPT prompt={{prompt}}\n  STEP.2.IMAGE_GEN model=minimax\n  STEP.3.OUTPUT type=image input=2",
    "parameterSchema": {
      "type": "object",
      "properties": {
        "prompt": { "type": "string", "description": "Product description" }
      },
      "required": ["prompt"]
    },
    "tags": ["ecommerce", "product"],
    "version": 1
  },
  "metadata": {
    "generator": "Clawfy Studio",
    "appVersion": "0.1.0"
  }
}
```

---

## 10. Import Template

### POST /api/v1/templates/import

Import a template from a `.clawfy-template.json` file.

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/templates/import \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d @my-template.clawfy-template.json
```

**Response (201 Created):**
```json
{
  "template": {
    "id": "tmpl_new123",
    "name": "My Imported Template",
    "description": "...",
    "category": "ecommerce",
    "dsl": "...",
    "parameterSchema": {...},
    "tags": [...],
    "version": 1
  }
}
```

**Error responses:**
| Status | Error | Meaning |
|--------|-------|---------|
| 400 | `VALIDATION_ERROR` | Invalid template structure, DSL syntax, or parameterSchema |
| 409 | `CONFLICT` | Template name already exists (auto-renamed with suffix) |
| 413 | `PAYLOAD_TOO_LARGE` | DSL > 64KB or parameterSchema > 16KB |

---

## 11. Estimate Cost

### POST /api/v1/cost

Estimate the cost of running a workflow before executing.

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workflowId` | string | Yes | Workflow to estimate |

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/cost \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workflowId": "wf_abc123"}'
```

**Response:**
```json
{
  "estimatedCost": 0.025,
  "stepCount": 3,
  "maxCapApplied": false,
  "breakdown": {
    "base": 0.015,
    "perStep": 0.005,
    "stepCount": 2,
    "maxCapApplied": false
  }
}
```

**Pricing formula:** `$0.015 + ($0.005 × STEP_count)`, max `$0.50`

---

## 12. WebSocket Real-Time Updates

Connect to `ws://localhost:3001/ws?executionId=YOUR_EXECUTION_ID&apiKey=YOUR_API_KEY` for live updates.

**Connection example (JavaScript):**
```javascript
const apiKey = process.env.CLAWFY_API_KEY; // Never hardcode!
const executionId = "exec_xyz789";
const wsUrl = `ws://localhost:3001/ws?executionId=${executionId}&apiKey=${apiKey}`;
const ws = new WebSocket(wsUrl);
```

**WebSocket Events:**

| Event Type | Payload | Description |
|------------|---------|-------------|
| `node_start` | `{ "nodeId": "1", "nodeType": "IMAGE_GEN" }` | Node execution started |
| `node_progress` | `{ "nodeId": "1", "progress": 50 }` | Node progress update |
| `node_complete` | `{ "nodeId": "1", "output": {...} }` | Node finished |
| `execution_complete` | `{ "outputs": {...} }` | All nodes done |
| `execution_error` | `{ "error": "..." }` | Execution failed |

**Example WebSocket message:**
```json
{
  "type": "node_progress",
  "nodeId": "2",
  "progress": 75
}
```

**Note:** Polling is recommended for simplicity. WebSocket is optional for real-time UI.

---

## Polling Protocol

After executing a workflow, you MUST poll until terminal state:

```
1. POST /api/v1/workflows/:id/execute → get executionId
2. Loop every 5 seconds:
   GET /api/v1/executions/:id
3. Check status:
   - "queued" → continue polling
   - "running" → continue polling
   - "completed" → GET /api/v1/executions/:id → get outputs
   - "failed" → report error
   - "cancelled" → notify user
```

**Maximum polling time:** 30 minutes for video workflows, 5 minutes for image/text.

---

## Results Delivery

When execution completes, present outputs to the user directly (not just URLs):

- **Images**: Use message tool to send image directly
- **Videos**: Use message tool to send video directly
- **Text**: Output in message body

**Always include:**
1. Brief summary of what was generated
2. The media file directly
3. Download link if applicable

**Example completion message:**
```
✅ Workflow completed successfully!

Generated: Product photography image (1024x1024)
Model: MiniMax Image Gen v2
Cost: $0.025

[IMAGE DISPLAYED DIRECTLY]
Download: https://cdn.clawfy.io/outputs/img_001.png
```

---

## Error Codes

| HTTP Status | Error Code | Meaning |
|-------------|------------|---------|
| 400 | `INVALID_REQUEST` | Malformed request body |
| 401 | `UNAUTHORIZED` | Invalid or missing API key |
| 404 | `WORKFLOW_NOT_FOUND` | Workflow ID does not exist |
| 404 | `EXECUTION_NOT_FOUND` | Execution ID does not exist |
| 404 | `TEMPLATE_NOT_FOUND` | Template ID does not exist |
| 409 | `CONFLICT` | Name already exists (for create operations) |
| 413 | `PAYLOAD_TOO_LARGE` | DSL or parameterSchema exceeds size limit |
| 422 | `VALIDATION_ERROR` | Request validates but is semantically wrong |
| 429 | `RATE_LIMITED` | Too many requests, wait and retry |
| 500 | `INTERNAL_ERROR` | Server error, try again later |

---

## Style Systems (Brand Identity)

For managing brand identities — colors, typography, logo, mood, and style presets — see:

**[rules/style-systems.md](style-systems.md)** — Complete Style System API reference
