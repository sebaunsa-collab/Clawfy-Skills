# Scenario: List Workflows

This scenario shows how to list all workflows and present them to the user.

## When to use

- User asks "what workflows do I have?"
- User asks to "see my workflows"
- User wants to choose which workflow to run

## Steps

### 1. Call List Workflows API

```bash
GET /api/workflows?limit=10
```

### 2. Parse the response

Extract the `workflows` array from the response.

### 3. Present to user

Format each workflow as a numbered list:

```
📋 Your Workflows (10 total)

1. **Product Photography**
   ID: wf_abc123
   Status: active
   Tags: ecommerce, product
   Last updated: 2 days ago

2. **Social Media Generator**
   ID: wf_def456
   Status: draft
   Tags: social, marketing
   Last updated: 1 week ago

...

Which workflow would you like to run?
```

### 4. Wait for user selection

If the list is long, ask the user which one they want (by number or name).

## Example Code

```
User: "What workflows do I have?"

Assistant:
*Call GET /api/workflows with limit=10*

📋 Your Workflows

1. **Product Photography** (active)
   Tags: ecommerce, product | Updated: Apr 8

2. **Instagram Reel Creator** (draft)
   Tags: social, video | Updated: Apr 5

3. **Email Header Generator** (active)
   Tags: marketing, email | Updated: Apr 1

Enter the number of the workflow you want to execute, or "cancel" to abort.
```

## Filtering

If the user specifies a preference (e.g., "ecommerce workflows"), filter the list:

```bash
GET /api/workflows
# Then filter client-side by tags
```

## Sorting

Display workflows by `updatedAt` descending (most recent first).

## No Workflows?

If the user has no workflows:

```
You don't have any workflows yet.

To create one:
1. Open Clawfy Web at http://localhost:3000
2. Go to Canvas and design your workflow
3. Save and it will appear here
```

## Error Handling

| Error | Response |
|-------|----------|
| 401 Unauthorized | "Your API key is invalid. Please check CLAWFY_API_KEY." |
| 500 Internal Error | "Failed to load workflows. Please try again." |