---
name: Bankr Guard
description: Security middleware for Bankr agents. Use when checking transaction safety, sanitizing on-chain data for LLM input, detecting prompt injection attempts, or auditing agent actions. Provides spending limits, contract allowlists, secret redaction, and audit logging.
version: 1.0.0
---

# Bankr Guard

Security middleware that wraps Bankr operations with safety checks.

## When to Use

- Before executing any transaction (check spending limits)
- Before passing on-chain data to LLM (sanitize for injection)
- Before logging or displaying agent output (redact secrets)
- When auditing what an agent did (review audit trail)

## Quick Usage

```typescript
import { BankrGuard } from 'bankr-guard';

const guard = BankrGuard.strict();

// Check transaction
const result = await guard.checkTransaction(tx);
if (!result.allowed) {
  throw new Error(`Blocked: ${result.reason}`);
}

// Sanitize input
const safe = await guard.sanitizeInput(untrustedData);

// Redact secrets
const clean = await guard.redactOutput(response);
```

## Presets

| Preset | Daily Limit | Per-TX Limit | Mode |
|--------|-------------|--------------|------|
| `strict()` | 0.5 ETH | 0.05 ETH | Reject suspicious |
| `standard()` | 1 ETH | 0.1 ETH | Warn on suspicious |
| `permissive()` | 10 ETH | 1 ETH | Log only |

## Prompt Injection Patterns Detected

- Instruction overrides ("ignore previous", "new task:")
- Jailbreak attempts (DAN, roleplay hijacking)
- Crypto-specific attacks ("drain wallet", "approve unlimited")
- Hidden unicode (zero-width chars, RTL override)
- Base64 encoded payloads

## Secrets Redacted

- Private keys (Base58 88-char, hex 64-char)
- Seed phrases (BIP39 12/24 words)
- API keys (common patterns)
- Environment variable leaks

## Integration with Bankr

```typescript
// Wrap Bankr trading with security
async function safeTrade(prompt: string) {
  const guard = BankrGuard.standard();
  
  // Sanitize the prompt
  const sanitized = await guard.sanitizeInput(prompt);
  if (sanitized.threats > 0) {
    return { error: 'Suspicious input detected' };
  }
  
  // Execute via Bankr
  const result = await bankr.prompt(sanitized.clean);
  
  // Redact any secrets from response
  const safe = await guard.redactOutput(result.response);
  
  return safe;
}
```
