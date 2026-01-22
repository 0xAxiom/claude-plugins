---
name: Bankr Error Handling
description: This skill should be used when encountering authentication errors, API key errors, 401 errors, "invalid API key", "BANKR_API_KEY not set", job failures, or any Bankr API errors. Provides setup instructions and troubleshooting guidance for resolving Bankr configuration issues.
version: 1.0.0
---

# Bankr Error Handling

Resolve Bankr API errors and authentication issues.

## Authentication Errors (401)

The most common error is an authentication failure due to missing or invalid API key.

### Symptoms

- HTTP 401 status code
- Error message: "Invalid API key" or "Unauthorized"
- Job submission fails immediately

### Resolution Steps

Present these setup instructions clearly to the user:

**Step 1: Create an API Key**
```
Visit https://bankr.bot/api to create a new API key
```

**Step 2: Set Environment Variable**
```bash
# Add to shell profile (~/.zshrc or ~/.bashrc)
export BANKR_API_KEY=bk_your_api_key_here

# Or create .env file in project root
BANKR_API_KEY=bk_your_api_key_here
```

**Step 3: Restart Claude Code**
```
Close and reopen the terminal/Claude Code session
```

### Important

- **DO NOT retry** when authentication fails
- The user must fix their API key configuration first
- Do not attempt alternative methods or workarounds

## Job Failure Errors

When a job completes with `status: "failed"`:

### Check the Error Field

The `error` field contains the failure reason:

```json
{
  "status": "failed",
  "error": "Insufficient balance for transaction"
}
```

### Common Job Failures

| Error | Cause | Resolution |
|-------|-------|------------|
| Insufficient balance | Not enough tokens for trade | Check balance, reduce amount |
| Token not found | Invalid token symbol/address | Verify token exists on chain |
| Slippage exceeded | Price moved too much | Retry or increase slippage |
| Transaction reverted | On-chain execution failed | Check transaction details |
| Rate limit exceeded | Too many requests | Wait and retry |
| Unsupported operation | Feature not available | Try alternative approach |

### Suggesting Alternatives

When a job fails, consider:
- Reducing the trade amount
- Trying a different chain
- Splitting into smaller transactions
- Checking token liquidity

## API Request Errors

Errors that occur before job creation:

### HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 400 | Bad request | Check prompt format |
| 401 | Unauthorized | Fix API key (see above) |
| 402 | Payment required | x402 payment failed |
| 404 | Not found | Invalid endpoint |
| 429 | Rate limited | Wait and retry |
| 500 | Server error | Retry after delay |

### x402 Payment Errors (402)

Bankr uses x402 micropayments on Base chain:

```json
{
  "x402Version": 1,
  "error": "Payment required",
  "accepts": [{
    "scheme": "exact",
    "network": "base",
    "asset": "0x22af33fe49fd1fa80c7149773dde5890d3c76f3b"
  }]
}
```

**Resolution**:
- Ensure wallet has BNKR tokens on Base
- Check privateKey is configured for payments
- Typical cost: ~$0.01 USD per request

## Troubleshooting Checklist

When errors occur, verify:

1. **API Key Configuration**
   - [ ] BANKR_API_KEY environment variable is set
   - [ ] Key starts with `bk_`
   - [ ] Key is not expired or revoked
   - [ ] Claude Code was restarted after setting

2. **Network/Connectivity**
   - [ ] Internet connection is working
   - [ ] api.bankr.bot is reachable
   - [ ] No firewall blocking requests

3. **Request Format**
   - [ ] Prompt is not empty
   - [ ] Prompt is under 10,000 characters
   - [ ] No invalid characters

4. **For Trading Operations**
   - [ ] Wallet has sufficient balance
   - [ ] Token exists on specified chain
   - [ ] Recipient address is valid (for transfers)

## Error Response Format

All errors follow this structure:

```json
{
  "success": false,
  "error": "Error message describing the issue",
  "message": "Additional context or instructions"
}
```

For job failures:

```json
{
  "success": true,
  "jobId": "job_ABC123",
  "status": "failed",
  "error": "Specific failure reason",
  "prompt": "Original request"
}
```

## Reporting Errors to Users

When presenting errors:

1. **Be clear**: State what went wrong simply
2. **Be actionable**: Provide specific fix steps
3. **Don't over-explain**: Avoid technical jargon unless needed
4. **Offer alternatives**: Suggest what they can do instead

### Example Error Responses

**Authentication Error**:
```
Your Bankr API key is not configured. To set it up:
1. Visit https://bankr.bot/api to create an API key
2. Set BANKR_API_KEY in your environment
3. Restart Claude Code
```

**Insufficient Balance**:
```
You don't have enough ETH for this trade. Your current balance
is 0.05 ETH but the trade requires 0.1 ETH. Try a smaller amount
or add more ETH to your wallet.
```

**Token Not Found**:
```
I couldn't find a token called "XYZ" on Base. Did you mean one of these?
- XYZ Protocol (XYZ) on Ethereum
- XYZ Token (XYZT) on Polygon
```

## Recovery Strategies

### For Transient Errors

Network issues, rate limits, server errors:
- Wait 5-10 seconds
- Retry the request once
- If still failing, inform user

### For Configuration Errors

API key, environment issues:
- Guide user through setup
- Do not retry until fixed
- Verify fix before continuing

### For Business Logic Errors

Insufficient balance, unsupported operations:
- Explain the limitation
- Suggest alternatives
- Help user adjust their request
