# Clawfy API — Complete Workflow Reference

> Base URL: `$CLAWFY_BASE_URL` (default: `http://localhost:3001`)
> All requests require: `x-api-key: $CLAWFY_API_KEY`
> Content-Type: `application/json` for all requests

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

### POST /api/v1/executions

Start a workflow execution. Returns immediately with an `executionId` for polling.

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workflowId` | string | Yes | ID of the workflow to execute |
| `input` | object | No | Input values for workflow nodes (key = nodeId, value = input) |
| `seed` | number | No | Random seed for reproducibility |
| `priority` | string | No | `high`, `normal`, or `low` (defaults to `normal` or workflow default) |

**Request:**
```bash
curl -s -X POST http://localhost:3001/api/v1/executions \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflowId": "wf_abc123",
    "input": {
      "1": "A beautiful sunset over mountains"
    },
    "priority": "normal"
  }'
```

**Response:**
```json
{
  "executionId": "exec_xyz789",
  "websocketUrl": "ws://localhost:3001/ws?executionId=exec_xyz789",
  "execution": {
    "id": "exec_xyz789",
    "workflowId": "wf_abc123",
    "status": "queued",
    "queuedAt": "2026-04-10T12:05:00.000Z"
  }
}
```

**Important fields:**
- `executionId` — use this for polling status
- `websocketUrl` — optional WebSocket for real-time updates (see WebSocket section)

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

## 9. Estimate Cost

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

## 10. WebSocket Real-Time Updates

Connect to `ws://localhost:3001/ws?executionId=YOUR_EXECUTION_ID` for live updates.

**Authentication:** Pass the API key as a query parameter in the URL:

```
ws://localhost:3001/ws?executionId=YOUR_EXECUTION_ID&apiKey=YOUR_API_KEY
```

**Why this method:** The WebSocket upgrade happens over HTTP, so the `x-api-key` header is not automatically forwarded. Adding `apiKey` as a query param is the supported method for authenticating WebSocket connections to Clawfy.

**Connection example (JavaScript):**
```javascript
const apiKey = process.env.CLAWFY_API_KEY; // Never hardcode!
const executionId = "exec_xyz789";
const wsUrl = `ws://localhost:3001/ws?executionId=${executionId}&apiKey=${apiKey}`;
const ws = new WebSocket(wsUrl);
```

**Important security notes:**
- **NEVER** hardcode the API key in the URL string. Always use `process.env.CLAWFY_API_KEY`
- **NEVER** log the WebSocket URL — it contains the credentials
- The API key in the query param is sent over the wire, so always use HTTPS in production

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
1. POST /api/v1/executions → get executionId
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

**Timeout handling:** If status stays "queued" for more than 5 minutes, check if inputs are correct.

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
| 422 | `VALIDATION_ERROR` | Request validates but is semantically wrong |
| 429 | `RATE_LIMITED` | Too many requests, wait and retry |
| 500 | `INTERNAL_ERROR` | Server error, try again later |