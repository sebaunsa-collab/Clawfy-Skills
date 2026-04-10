# Scenario: Execute and Monitor Workflow

This scenario shows how to execute a workflow, poll for completion, and deliver results.

## When to use

- User asks to "run" or "execute" a workflow
- User asks to "generate" something using a workflow
- User asks to "create" something (image, video, etc.)

## Complete Flow

```
User wants to generate product images
  │
  ├─ 1. List workflows (let user pick)
  │
  ├─ 2. Get workflow details
  │
  ├─ 3. Ask for required inputs
  │
  ├─ 4. Execute workflow
  │
  ├─ 5. Poll until completion (every 5 seconds)
  │
  └─ 6. Deliver results
```

---

## Step 1: List and Let User Choose

First, present available workflows:

```
I found 3 workflows that can generate images:

1. **Product Photography** — Professional e-commerce shots
2. **Social Media Pack** — Multi-image bundle for posts
3. **Background Removal** — Clean product cutouts

Which would you like to run?
```

---

## Step 2: Get Workflow Details

After user selects, get full details to understand input nodes:

```bash
GET /api/workflows/{workflowId}
```

Look at the `graph.nodes` to identify required inputs:
- `TEXT_PROMPT` nodes need text input
- `IMAGE_GEN` nodes need prompt + model config
- `imageInput` nodes need image URLs

---

## Step 3: Collect Inputs

Ask user for each required input. Never use defaults.

```
To run "Product Photography", I need:

1. **Product description** — What should appear in the image?
   (e.g., "Gold necklace on white marble surface")

2. **Aspect ratio** — What dimensions?
   Options: 1:1 (square), 16:9 (wide), 9:16 (vertical)

3. **Style** — Any specific style?
   Options: natural, studio, dramatic

What values would you like?
```

---

## Step 4: Execute

Once you have all inputs, execute via the workflow's execute endpoint:

```bash
POST /api/workflows/{workflowId}/execute
```

No body required — the workflow ID is in the URL path.

Response:
```json
{
  "executionId": "exec_xyz789",
  "status": "queued",
  "websocketUrl": "ws://localhost:3001/ws?executionId=exec_xyz789"
}
```

Tell the user:
```
✅ Workflow submitted!

Execution ID: exec_xyz789
Status: queued
Tracking progress...
```

---

## Step 5: Poll Until Completion

**CRITICAL: Never stop polling until terminal state.**

Poll every 5 seconds:

```bash
GET /api/executions/exec_xyz789
```

### Polling Loop

```
Status: queued | Waiting in queue...
Status: running | ████████░░░░░░░░ 40%
Status: running | ████████████████░ 70%
Status: running | ██████████████████ 90%
Status: completed | 100%
```

### Status Handling

| Status | Action |
|--------|--------|
| `queued` | Continue polling, notify user |
| `running` | Continue polling, update progress if available |
| `completed` | → Step 6: Get and deliver results |
| `failed` | Report error to user |
| `cancelled` | Notify user |

---

## Step 6: Deliver Results

When `status: "completed"`, get the outputs:

```bash
GET /api/executions/exec_xyz789
```

Look at `execution.outputs`:

```json
{
  "outputs": {
    "3": "https://cdn.clawfy.io/outputs/img_001.png"
  }
}
```

### Deliver to User

```
✅ Generated successfully!

**Product Photography** — completed in 47 seconds

Output: [IMAGE DISPLAYED DIRECTLY]

Download: https://cdn.clawfy.io/outputs/img_001.png
Cost: $0.025

Would you like to run another workflow?
```

### Media Delivery Rules

| Type | How to deliver |
|------|---------------|
| Image | Send image directly via message tool |
| Video | Send video directly via message tool |
| Text | Output in message body |

**Always include:**
- Summary of what was generated
- The media itself (not just URL)
- Download link
- Cost

---

## Cancellation

If user asks to cancel during execution:

```bash
POST /api/executions/exec_xyz789/cancel
```

Response:
```json
{
  "cancelled": true,
  "executionId": "exec_xyz789"
}
```

Notify user:
```
❌ Execution cancelled.

Note: If nodes were already running, they will complete and incur costs.
```

---

## Error Handling

### Execution Failed

```json
{
  "execution": {
    "status": "failed",
    "error": "IMAGE_GEN model 'minimax-v4' not available"
  }
}
```

Response to user:
```
❌ Workflow failed.

Reason: Image generation model is currently unavailable.
Suggestion: Try again in a few minutes, or use a different workflow.
```

### Timeout

If polling exceeds maximum time:

| Workflow | Max Time |
|----------|----------|
| Text/Image | 5 minutes |
| Video | 30 minutes |

If exceeded:
```
⚠️ This is taking longer than expected.

The workflow is still running. You can:
1. Continue waiting (I'll keep polling)
2. Check back later with execution ID: exec_xyz789
```

---

## Full Example

```
User: "Generate a product photo of running shoes"

Assistant: I found 2 workflows for product photography:

1. **Product Photo Basic** — Simple single image
2. **Product Photo Pro** — Multiple angles + background

Which would you like?

User: "the pro one"

Assistant: For "Product Photo Pro", I need a few inputs:

1. **Product name**: running shoes
2. **Brand**: Nike Air Max
3. **Style**: dynamic, showing motion
4. **Background**: white studio

Please provide these values.

User: "running shoes, Nike Air Max, dynamic motion, white"

Assistant: *Executing...*

✅ Submitted! Tracking progress...

Status: queued | 0%
Status: running | ███░░░░░░░░ 25%
Status: running | ██████░░░░░ 50%
Status: running | █████████░░ 75%
Status: completed | 100%

✅ Done! Generated 3 product images in 52 seconds.

[IMAGES DISPLAYED DIRECTLY]

Download: https://cdn.clawfy.io/outputs/shoes_001.png
         https://cdn.clawfy.io/outputs/shoes_002.png
         https://cdn.clawfy.io/outputs/shoes_003.png

Cost: $0.035
```