---
name: Bankr Job Workflow
description: This skill should be used when executing Bankr requests, submitting prompts to Bankr API, polling for job status, checking job progress, using Bankr MCP tools, or understanding the submit-poll-complete workflow pattern. Provides the core asynchronous job pattern for all Bankr API operations.
version: 1.0.0
---

# Bankr Job Workflow

Execute Bankr API operations using the asynchronous job pattern via MCP tools.

## Core Concept

All Bankr operations follow a **submit-poll-complete** pattern:

1. **Submit** - Send natural language prompt, receive job ID
2. **Poll** - Check status every 2 seconds until terminal state
3. **Complete** - Report results when status is completed/failed/cancelled

## Available MCP Tools

### `bankr_agent_submit_prompt`

Submit a natural language prompt to start a job.

```
bankr_agent_submit_prompt(prompt: string) -> { jobId: string }
```

**Input**: Natural language request (e.g., "Buy $50 of ETH on Base")
**Output**: Job ID for tracking (e.g., "job_ABC123...")

### `bankr_agent_get_job_status`

Check the current status of a job.

```
bankr_agent_get_job_status(jobId: string) -> JobStatusResponse
```

**Response fields**:
- `status`: "pending" | "processing" | "completed" | "failed" | "cancelled"
- `response`: Text answer (when completed)
- `transactions`: Array of executed transactions
- `richData`: Images or charts (base64 or URL)
- `statusUpdates`: Progress messages during execution
- `error`: Error message (when failed)

### `bankr_agent_cancel_job`

Cancel a running job.

```
bankr_agent_cancel_job(jobId: string) -> JobStatusResponse
```

**Note**: Cancellation is idempotent - returns success even if already cancelled.

## Workflow Steps

### Step 1: Submit the Request

Submit the user's request as a natural language prompt:

```
1. Call bankr_agent_submit_prompt with the user's request
2. Store the returned jobId for polling
3. Inform user that request was submitted
```

### Step 2: Poll for Status

Poll every 2 seconds until a terminal state:

```
1. Call bankr_agent_get_job_status with jobId
2. Check statusUpdates array for new messages
3. Report any new status updates to the user
4. If status is terminal (completed/failed/cancelled), proceed to Step 3
5. Otherwise, wait 2 seconds and repeat
```

**Status Update Handling**:
- Track the last reported update count
- Only report NEW updates to avoid repetition
- Updates show what the agent is doing (e.g., "Analyzing market data...", "Preparing transaction...")

### Step 3: Report Results

Handle the terminal state appropriately:

**If completed**:
- Share the `response` text with the user
- If `transactions` exist, explain what was executed
- If `richData` exists (images/charts), mention it's available

**If failed**:
- Report the `error` message clearly
- Suggest alternatives if applicable
- If authentication error, load `bankr-error-handling` skill

**If cancelled**:
- Confirm cancellation to the user

## Job Status States

| Status | Meaning | Action |
|--------|---------|--------|
| `pending` | Job queued, not started | Keep polling |
| `processing` | Job running | Keep polling, report statusUpdates |
| `completed` | Finished successfully | Read response and transactions |
| `failed` | Error occurred | Check error field |
| `cancelled` | User cancelled | No further action |

## Timing Guidelines

- **Poll interval**: 2 seconds
- **Typical completion**: 30 seconds to 2 minutes
- **Maximum wait**: 4-5 minutes before suggesting cancellation
- Jobs involving complex trades or analysis take longer

## Output Guidelines

Present results based on query type:

| Query Type | Output Format |
|------------|---------------|
| Price queries | State price clearly with token symbol (e.g., "ETH is $3,245.67") |
| Trades | Confirm what was traded, amounts, transaction details |
| Market analysis | Summarize key insights concisely |
| Polymarket | State odds clearly with relevant context |
| Balances | List holdings with USD values |
| Errors | Explain clearly, suggest alternatives |

## Cancellation

When user requests cancellation:

```
1. Call bankr_agent_cancel_job with the jobId
2. Confirm cancellation to user
3. Do not continue polling
```

**When to suggest cancellation**:
- Job running longer than expected (>3 minutes for simple queries)
- User indicates they want to stop
- User wants to modify their request

## Example Workflow

```
User: "What's the price of Bitcoin?"

1. Submit: bankr_agent_submit_prompt("What's the price of Bitcoin?")
   -> jobId: "job_XYZ789"

2. Poll: bankr_agent_get_job_status("job_XYZ789")
   -> status: "processing", statusUpdates: ["Fetching price data..."]
   -> Report: "Fetching price data..."

3. Poll again (2s later):
   -> status: "completed", response: "Bitcoin (BTC) is trading at $97,245.32..."

4. Report: "Bitcoin (BTC) is currently trading at $97,245.32"
```

## Error Recovery

If polling fails or connection is lost:

1. Retry the status check after a brief delay
2. If multiple failures, inform user of connectivity issues
3. The job continues running server-side regardless
4. Can resume polling with the same jobId

## Integration with Capability Skills

This workflow skill handles execution. Load the appropriate capability skill first to understand:
- What operations are supported
- Required parameters and formats
- Chain-specific considerations

Then use this workflow to execute the request.
