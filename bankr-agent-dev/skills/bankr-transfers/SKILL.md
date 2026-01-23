---
name: Bankr Dev - Transfers
description: This skill should be used when building a payment bot, implementing batch transfers, creating a tip bot, automating payroll with ENS resolution, or building a distribution system. Covers TypeScript prompt templates like `transferPrompts.toEns()`, recipient validation patterns, and PaymentBot class with batch processing. Triggered by "payment bot code", "transfer prompt template", "ENS resolution TypeScript", "batch transfer implementation".
version: 1.0.0
---

# Transfers - Developer Guide

Build bots and applications that transfer tokens via the Bankr API.

## Overview

Token transfers through Bankr support:
- Direct wallet addresses (0x...)
- ENS name resolution (vitalik.eth)
- Social handle resolution (@username on Twitter, Farcaster, Telegram)
- Multiple chains and token types

**Supported Chains:** Base, Polygon, Ethereum, Unichain, Solana

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for transfer operations
const transferPrompts = {
  // To address
  toAddress: (amount: string, token: string, address: string, chain?: string) =>
    chain
      ? `Send ${amount} ${token} to ${address} on ${chain}`
      : `Send ${amount} ${token} to ${address}`,

  // To ENS name
  toEns: (amount: string, token: string, ens: string, chain?: string) =>
    chain
      ? `Send ${amount} ${token} to ${ens} on ${chain}`
      : `Send ${amount} ${token} to ${ens}`,

  // To social handle
  toSocial: (amount: string, token: string, handle: string, platform: string) =>
    `Send ${amount} ${token} to ${handle} on ${platform}`,
};

// Recipient format helpers
const formatRecipient = {
  address: (addr: string) => addr,
  ens: (name: string) => name,
  social: (handle: string, platform: string) => `@${handle.replace("@", "")} on ${platform}`,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

async function handleTransferResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Transfer successful:", result.response);

    // Process executed transactions
    if (result.transactions) {
      for (const tx of result.transactions) {
        const data = tx.metadata.__ORIGINAL_TX_DATA__;
        if (data) {
          console.log(`Sent ${data.inputTokenAmount} ${data.inputTokenTicker}`);
          console.log(`To: ${data.receiver}`);
          console.log(`Message: ${data.humanReadableMessage}`);
        }
      }
    }
  } else if (result.status === "failed") {
    console.error("Transfer failed:", result.error);
  }
}

// Extract resolved address from response
function extractResolvedAddress(result: JobStatusResponse): string | null {
  if (result.transactions?.[0]) {
    return result.transactions[0].metadata.__ORIGINAL_TX_DATA__.receiver;
  }
  return null;
}
```

## TypeScript Types

Reference `bankr-client-patterns` skill for complete types. Key transfer types:

```typescript
// Transaction types for transfers
type TransferTransaction =
  | TransferEthTransaction
  | TransferErc20Transaction
  | TransferNftTransaction;

// Transfer transaction metadata
interface TransferMetadata {
  __ORIGINAL_TX_DATA__: {
    chain: string;
    humanReadableMessage: string;
    inputTokenAddress: string;
    inputTokenAmount: string;
    inputTokenTicker: string;
    receiver: string; // Resolved address
  };
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Payroll Bot | Scheduled payments to team | `Send $500 USDC to 0x...` |
| Tip Bot | Reward users via social | `Send $5 ETH to @user on Twitter` |
| Distribution Bot | Airdrop to multiple addresses | `Send 100 BNKR to vitalik.eth` |
| Allowance Bot | Regular transfers to wallets | `Send 10% of my ETH to 0x...` |

## Code Example: Payment Bot

```typescript
import { execute } from "./bankr-client";

interface Payment {
  recipient: string;
  recipientType: "address" | "ens" | "twitter" | "farcaster";
  amount: string;
  token: string;
  chain?: string;
}

class PaymentBot {
  async processPayment(payment: Payment): Promise<boolean> {
    const prompt = this.buildPrompt(payment);
    const result = await execute(prompt, (msg) => console.log("  >", msg));

    if (result.status === "completed") {
      console.log("Payment sent to:", extractResolvedAddress(result));
      return true;
    }

    console.error("Payment failed:", result.error);
    return false;
  }

  private buildPrompt(payment: Payment): string {
    const { recipient, recipientType, amount, token, chain } = payment;

    switch (recipientType) {
      case "address":
      case "ens":
        return transferPrompts.toAddress(amount, token, recipient, chain);
      case "twitter":
      case "farcaster":
        return transferPrompts.toSocial(amount, token, formatRecipient.social(recipient, recipientType), recipientType);
      default:
        return transferPrompts.toAddress(amount, token, recipient, chain);
    }
  }

  async processBatch(payments: Payment[]): Promise<{ success: number; failed: number }> {
    let success = 0;
    let failed = 0;

    for (const payment of payments) {
      const result = await this.processPayment(payment);
      if (result) {
        success++;
      } else {
        failed++;
      }

      // Add delay between transfers to avoid rate limiting
      await new Promise((r) => setTimeout(r, 3000));
    }

    return { success, failed };
  }
}

// Usage
const bot = new PaymentBot();

// Single payment
await bot.processPayment({
  recipient: "vitalik.eth",
  recipientType: "ens",
  amount: "0.01",
  token: "ETH",
  chain: "Base",
});

// Batch payments
const payments: Payment[] = [
  { recipient: "0x1234...", recipientType: "address", amount: "100", token: "USDC" },
  { recipient: "alice.eth", recipientType: "ens", amount: "0.05", token: "ETH" },
  { recipient: "bob", recipientType: "twitter", amount: "$10", token: "ETH" },
];

const results = await bot.processBatch(payments);
console.log(`Completed: ${results.success} success, ${results.failed} failed`);
```

## Recipient Resolution

Bankr resolves recipients automatically. Here's how to validate before sending:

```typescript
// Validate recipient format
function validateRecipient(recipient: string): {
  type: "address" | "ens" | "social";
  valid: boolean;
  normalized: string;
} {
  // Ethereum address
  if (/^0x[a-fA-F0-9]{40}$/.test(recipient)) {
    return { type: "address", valid: true, normalized: recipient.toLowerCase() };
  }

  // ENS name
  if (recipient.endsWith(".eth")) {
    return { type: "ens", valid: true, normalized: recipient.toLowerCase() };
  }

  // Social handle (starts with @)
  if (recipient.startsWith("@")) {
    return { type: "social", valid: true, normalized: recipient };
  }

  return { type: "address", valid: false, normalized: recipient };
}

// Resolution may fail for these reasons:
const RESOLUTION_ERRORS = {
  ENS_NOT_FOUND: "ENS name does not exist",
  NO_LINKED_WALLET: "Social handle has no linked wallet",
  INVALID_ADDRESS: "Address format is invalid",
};
```

## Error Handling

Common errors and how to handle them in bots:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| ENS not found | Invalid ENS name | Verify ENS exists |
| No linked wallet | Social user hasn't linked | Skip or notify user |
| Insufficient balance | Not enough funds | Check balance first |
| Invalid address | Malformed address | Validate format |

```typescript
async function safeTransfer(
  recipient: string,
  amount: string,
  token: string
): Promise<{ success: boolean; error?: string }> {
  const validation = validateRecipient(recipient);
  if (!validation.valid) {
    return { success: false, error: "Invalid recipient format" };
  }

  const result = await execute(`Send ${amount} ${token} to ${recipient}`);

  if (result.status === "completed") {
    return { success: true };
  }

  const error = result.error || "Unknown error";
  if (error.includes("not found") || error.includes("no linked wallet")) {
    return { success: false, error: "Recipient could not be resolved" };
  }
  if (error.includes("Insufficient")) {
    return { success: false, error: "Insufficient balance" };
  }

  return { success: false, error };
}
```

## Security Considerations

```typescript
// Best practices for transfer bots
const TRANSFER_LIMITS = {
  maxSingleTransfer: 1000, // USD
  maxDailyTotal: 10000, // USD
  requireConfirmation: 500, // Above this, require extra verification
};

function checkTransferLimits(
  amountUsd: number,
  dailyTotalUsd: number
): { allowed: boolean; reason?: string } {
  if (amountUsd > TRANSFER_LIMITS.maxSingleTransfer) {
    return { allowed: false, reason: "Exceeds single transfer limit" };
  }
  if (dailyTotalUsd + amountUsd > TRANSFER_LIMITS.maxDailyTotal) {
    return { allowed: false, reason: "Exceeds daily limit" };
  }
  return { allowed: true };
}

// Always log transfers for audit
function logTransfer(payment: Payment, result: JobStatusResponse) {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    recipient: payment.recipient,
    amount: payment.amount,
    token: payment.token,
    status: result.status,
    jobId: result.jobId,
    resolvedAddress: result.transactions?.[0]?.metadata.__ORIGINAL_TX_DATA__.receiver,
  }));
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-portfolio` - Check balances before transfers
