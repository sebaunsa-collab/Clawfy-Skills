# Clawfy Best Practices

## Security Rules

### NEVER expose credentials

- **Never** hardcode API keys in logs, error messages, or responses
- **Never** print `$CLAWFY_API_KEY` value
- **Always** use environment variables: `$CLAWFY_API_KEY`

**Wrong:**
```
console.log("Using API key: " + apiKey);
```

**Correct:**
```
console.log("Using API key: [REDACTED]");
```

### Use environment variables

```bash
# Always reference via env var, never hardcode
export CLAWFY_API_KEY="clawfy_test_xxx"
```

### HTTPS in production

When `CLAWFY_BASE_URL` points to a production server, always use HTTPS:
- `https://api.clawfy.com` ✅
- `http://api.clawfy.com` ❌ (credentials sent in plain text)

---

## Error Handling

### 401 — Unauthorized

**Cause:** API key is invalid, expired, or missing.

**Action:**
1. Verify `CLAWFY_API_KEY` is set correctly
2. Get a fresh key from Clawfy Web → Settings → API Keys
3. Retry the request

### 404 — Not Found

**Cause:** Workflow or execution ID does not exist.

**Action:**
1. List workflows to confirm the ID exists: `GET /api/workflows`
2. Check if the execution has already completed or been cancelled
3. Verify the ID is correct (no typos)

### 422 — Validation Error

**Cause:** Request is syntactically correct but semantically wrong.

**Action:**
1. Check the `message` field in the error response
2. Common causes:
   - Missing required fields
   - Invalid field values (e.g., non-existent model name)
   - Malformed DSL in workflow creation

### 429 — Rate Limited

**Cause:** Too many requests in a short period.

**Action:**
1. Wait before retrying (check `Retry-After` header if present)
2. Implement exponential backoff:
   - 1st retry: wait 1 second
   - 2nd retry: wait 2 seconds
   - 3rd retry: wait 4 seconds
   - Max 5 retries

### 500 — Internal Server Error

**Cause:** Something went wrong on the server side.

**Action:**
1. Wait 5 seconds
2. Retry once
3. If still failing, report the issue to the user and suggest trying again later

---

## Input Validation

### Validate request size before sending

Large payloads can be rejected by the server or cause timeouts. Always validate input sizes:

| Field | Max Recommended Size | Why |
|-------|---------------------|-----|
| `input` (workflow execution) | < 10 KB per value | Node inputs are processed in memory |
| `dsl` (workflow definition) | < 50 KB | Large DSL is a code smell |
| `prompt` strings | < 8000 chars | Downstream AI models have token limits |

**Rule of thumb:** If the user provides input that feels "too large", ask them to simplify or chunk it.

### Never expose internal error details

When the API returns 500 errors, you may see stack traces or database messages in the response. **Never** show these to the user.

**Internal error (from server):**
```json
{ "error": "ECONNREFUSED", "stack": "Error: connect ECONNREFUSED 127.0.0.1:5432\n    at Database.connect..." }
```

**What you say to the user:**
```
"The workflow execution failed. This is usually a temporary issue — please try again in a few moments."
```

**Why:** Stack traces and internal paths reveal infrastructure details that could aid attackers.

---

## Polling Tips

### Polling Frequency

| Workflow Type | Polling Interval | Max Duration |
|---------------|------------------|-------------|
| Text generation | 3 seconds | 2 minutes |
| Image generation | 5 seconds | 5 minutes |
| Video generation | 10 seconds | 30 minutes |

### Never give up early

After executing a workflow, you MUST continue polling until terminal state. Do NOT stop when status is `queued` or `running` — your obligation is to follow through to completion.

### Polling loop example

```
Execution submitted: exec_xyz789
Status: queued | Waiting...
Status: running | 25% complete
Status: running | 50% complete
Status: running | 75% complete
Status: completed | 100% complete
Fetching results... Done!
```

### Timeout handling

If status stays `queued` for more than 5 minutes:
1. Check if the workflow inputs are correct
2. Verify the API is healthy with `/api/health`
3. Consider cancelling and retrying

---

## Rate Limiting

Clawfy API has a rate limit of **100 requests per minute** per IP.

If you receive `429`:
1. Wait for the `Retry-After` seconds (if header is present)
2. Otherwise wait 30 seconds before retrying
3. Reduce request frequency in your next polling loop

### Best practices to avoid rate limits

- Batch multiple reads when possible
- Use WebSocket for real-time updates instead of polling (reduces API calls)
- Cache workflow lists if the user requests them multiple times

---

## Workflow Design Principles

### Keep inputs minimal

Ask the user only for essential inputs. Use sensible defaults where possible, but always confirm before executing.

### Present workflow options

When the user asks to "run a workflow":
1. First list available workflows: `GET /api/workflows`
2. Present the top 5 most relevant options
3. Let the user choose before executing

### Always estimate cost first

Before running a potentially expensive workflow (video, multi-step):
1. Call `POST /api/cost` with the `workflowId`
2. Inform the user of the estimated cost
3. Proceed only after confirmation

### Handle failures gracefully

If a workflow fails:
1. Read the `error` field from the execution status
2. Provide a clear, non-technical explanation to the user
3. Suggest next steps (retry, different workflow, adjust inputs)

---

## WebSocket Usage

WebSocket is optional. Use it when you need real-time progress updates for a UI.

**When to use WebSocket:**
- Building a visual progress indicator
- User wants live updates on a long-running workflow
- Reducing polling API calls

**When to use Polling:**
- Simple CLI or script execution
- No UI required
- Reliability over real-time (WebSocket can disconnect)

### Authenticating WebSocket connections

**Always pass the API key as a query parameter:**

```javascript
// ✅ CORRECT — API key in query param
const ws = new WebSocket(`ws://localhost:3001/ws?executionId=${execId}&apiKey=${process.env.CLAWFY_API_KEY}`);

// ❌ WRONG — API key in a message after connect
// The x-api-key header is NOT forwarded to the WebSocket connection
```

The WebSocket upgrade is an HTTP request, so the standard `x-api-key` header is consumed by the HTTP upgrade mechanism and does not carry over to the WebSocket connection. Use the `apiKey` query parameter for authentication.

### WebSocket connection handling

```javascript
// Connect
const ws = new WebSocket("ws://localhost:3001/ws?executionId=exec_xyz789");

// Handle messages
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  switch (data.type) {
    case "node_progress":
      console.log(`Node ${data.nodeId}: ${data.progress}%`);
      break;
    case "execution_complete":
      console.log("Done!", data.outputs);
      ws.close();
      break;
  }
};

// Handle errors
ws.onerror = () => {
  console.log("WebSocket error - falling back to polling");
  // Revert to polling
};
```

---

## Testing Your Integration

### Quick smoke test

```bash
# 1. Verify API is up
curl http://localhost:3001/api/health

# 2. List workflows
curl http://localhost:3001/api/workflows \
  -H "x-api-key: $CLAWFY_API_KEY"

# 3. Execute a simple workflow (if you have one)
curl -X POST http://localhost:3001/api/executions \
  -H "x-api-key: $CLAWFY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"workflowId": "wf_your_id"}'
```

### Verifying credentials

```bash
# If this returns 401, your API key is invalid
curl http://localhost:3001/api/health \
  -H "x-api-key: $CLAWFY_API_KEY"
```

---

## Common Pitfalls

| Pitfall | Prevention |
|---------|------------|
| Forgetting to poll after execute | Always enter polling loop immediately |
| Exposing API key in logs | Use `[REDACTED]` or never log the value |
| Not handling rate limits | Implement exponential backoff |
| Stopping at `running` status | Remember: "running" is not terminal |
| Using WebSocket instead of polling for reliability | Polling is more reliable for automation |
| Hardcoding base URL | Always use `$CLAWFY_BASE_URL` env var |
| Sending API key via WebSocket message instead of query param | Use `?apiKey=` in the connection URL |
| Exposing 500 error stack traces to users | Show generic message, log details server-side |
| Sending oversized payloads | Validate size before sending |