# Clawfy Skills

Agent skill for operating Clawfy workflows via API. Enables listing, executing, monitoring, and retrieving results from Clawfy workflows.

## What is this?

A **skill** is a set of instructions that helps an AI agent (like Claude, Codex, or OpenClaw) operate the Clawfy API correctly. It contains documentation, best practices, and step-by-step scenarios.

## Installation

### Via GitHub (recommended)

```bash
npx skills add clawfy/clawfy-skills
```

Or for a specific agent:
```bash
npx skills add clawfy/clawfy-skills -a your-agent-name
```

### Via OpenClaw / ClawHub

```bash
openclaw skills install clawfy-workflow-execution
clawhub install clawfy-workflow-execution
```

### Manual Installation

Clone or copy this repository to your agent's skills directory:
```
~/.openclaw/skills/clawfy-skills/
```

## Configuration

Set up your Clawfy credentials as environment variables:

```bash
# Required
export CLAWFY_API_KEY="your-api-key"

# Optional (defaults to http://localhost:3001)
export CLAWFY_BASE_URL="http://localhost:3001"
```

**Getting an API key:**
1. Open Clawfy Web (http://localhost:3000)
2. Go to **Settings → API Keys**
3. Create a new key

## Directory Structure

```
clawfy-skills/
├── SKILL.md                      # Entry point — read this first
└── rules/
    ├── api-workflows.md          # Complete API reference
    ├── best-practices.md          # Security and error handling
    └── scenarios/
        ├── scenario-list-workflows.md
        └── scenario-execute-workflow.md
```

## Quick Start

After installation, the agent can:

- **List workflows**: "show my workflows"
- **Execute workflows**: "run the product photo workflow"
- **Monitor progress**: automatic polling until completion
- **Get results**: deliver generated media directly

## Usage Example

```
User: Generate a product photo using the ecommerce workflow

Agent: I'll help you run the ecommerce workflow.

First, let me see what workflows are available...

📋 Your Workflows

1. **Product Photography** (active)
2. **Social Media Generator** (draft)
3. **Email Header Maker** (active)

Which would you like to run?

User: 1

Agent: For "Product Photography", I need a few inputs:

1. **Product description**: What should appear?
2. **Aspect ratio**: 1:1, 16:9, or 9:16?
3. **Style**: natural, studio, or dramatic?

User: "golden necklace", "1:1", "studio"

Agent: ✅ Executing! Tracking progress...

Status: queued → running → completed

✅ Generated successfully!

[IMAGE DISPLAYED DIRECTLY]

Cost: $0.025
```

## Requirements

- Clawfy API running (local or remote)
- Valid API key

## Topics Covered

### API Reference (`rules/api-workflows.md`)

All 10 endpoints documented:
- Health check
- List / get workflows
- Create workflows
- Execute / cancel / poll executions
- List templates
- Estimate cost
- WebSocket real-time updates

### Best Practices (`rules/best-practices.md`)

- Security rules (never expose API keys)
- Error handling (401, 404, 422, 429, 500)
- Polling tips (frequency, timeouts)
- Rate limiting
- WebSocket vs polling

### Scenarios (`rules/scenarios/`)

Step-by-step guides for common tasks:
- List workflows and let user choose
- Execute and monitor workflow to completion

## About

- **Version**: 1.0.0
- **Maintained by**: Clawfy team
- **License**: MIT

---

For questions or issues, contact the Clawfy team or open an issue on GitHub.