---
name: Bankr Dev - Token Trading
description: This skill should be used when building a trading bot, implementing token swaps programmatically, creating a DCA bot, building an arbitrage system, or automating cross-chain bridges. Covers TypeScript prompt templates like `tradingPrompts.buy()`, swap response parsing, and complete TradingBot class implementation. Triggered by "trading bot code", "swap prompt template", "TypeScript trading types", "programmatic token swaps".
version: 1.0.0
---

# Token Trading - Developer Guide

Build bots and applications that trade tokens via the Bankr API.

## Overview

Token trading through Bankr supports:
- Same-chain swaps (buy/sell tokens)
- Cross-chain bridges and swaps
- ETH/WETH conversions
- Multiple amount formats (USD, percentage, exact)

**Supported Chains:** Base, Polygon, Ethereum, Unichain, Solana

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for trading operations
const tradingPrompts = {
  // Same-chain swaps
  buy: (amount: string, token: string, chain?: string) =>
    chain ? `Buy ${amount} of ${token} on ${chain}` : `Buy ${amount} of ${token}`,

  sell: (amount: string, token: string, chain?: string) =>
    chain ? `Sell ${amount} of ${token} on ${chain}` : `Sell ${amount} of ${token}`,

  swap: (fromAmount: string, fromToken: string, toToken: string, chain?: string) =>
    chain
      ? `Swap ${fromAmount} ${fromToken} for ${toToken} on ${chain}`
      : `Swap ${fromAmount} ${fromToken} for ${toToken}`,

  // Cross-chain
  bridge: (amount: string, token: string, fromChain: string, toChain: string) =>
    `Bridge ${amount} ${token} from ${fromChain} to ${toChain}`,

  crossChainSwap: (fromToken: string, fromChain: string, toToken: string, toChain: string) =>
    `Swap ${fromToken} on ${fromChain} for ${toToken} on ${toChain}`,

  // ETH/WETH conversion
  wrapEth: (amount: string) => `Convert ${amount} ETH to WETH`,
  unwrapWeth: (amount: string) => `Convert ${amount} WETH to ETH`,
};

// Amount format helpers
const formatAmount = {
  usd: (dollars: number) => `$${dollars}`,
  percentage: (pct: number) => `${pct}%`,
  exact: (amount: number, token: string) => `${amount} ${token}`,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

async function handleTradeResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Trade successful:", result.response);

    // Process executed transactions
    if (result.transactions) {
      for (const tx of result.transactions) {
        const data = tx.metadata.__ORIGINAL_TX_DATA__;
        if (data) {
          console.log(data.humanReadableMessage);
          console.log(`${data.inputTokenAmount} ${data.inputTokenTicker} → ${data.outputTokenTicker}`);
        } else if (tx.metadata.description) {
          console.log(tx.metadata.description);
        }
      }
    }
  } else if (result.status === "failed") {
    console.error("Trade failed:", result.error);
  }
}
```

## TypeScript Types

Reference `bankr-client-patterns` skill for complete types. Key trading types:

```typescript
// Transaction types for trading
type TradingTransaction =
  | SwapTransaction
  | ApprovalTransaction
  | ConvertEthToWethTransaction
  | ConvertWethToEthTransaction
  | SwapCrossChainTransaction;

// Swap transaction metadata
interface SwapMetadata {
  __ORIGINAL_TX_DATA__: {
    chain: string;
    humanReadableMessage: string;
    inputTokenAddress: string;
    inputTokenAmount: string;
    inputTokenTicker: string;
    outputTokenAddress: string;
    outputTokenAmount: string;
    outputTokenTicker: string;
    receiver: string;
  };
  approvalRequired?: boolean;
  approvalTx?: { to: string; data: string };
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| DCA Bot | Regular purchases at intervals | `Buy $100 of ETH` |
| Dip Buyer | Buy when price drops | `Buy $50 of ETH on Base` |
| Arbitrage | Cross-chain price differences | `Swap ETH on Ethereum for USDC on Polygon` |
| Rebalancer | Maintain portfolio ratios | `Sell 10% of my ETH for USDC` |
| Auto-Sell | Take profits at targets | `Sell $500 worth of BNKR` |

## Code Example: Trading Bot

```typescript
import { execute } from "./bankr-client";

interface TradeConfig {
  token: string;
  chain: string;
  buyAmount: string;
}

class TradingBot {
  constructor(private config: TradeConfig) {}

  async executeBuy(): Promise<void> {
    const prompt = tradingPrompts.buy(`$${this.config.buyAmount}`, this.config.token, this.config.chain);
    const result = await execute(prompt, (msg) => console.log("  >", msg));
    await handleTradeResponse(result);
  }

  async executeSell(percentage: number): Promise<void> {
    const prompt = tradingPrompts.sell(`${percentage}%`, this.config.token, this.config.chain);
    const result = await execute(prompt, (msg) => console.log("  >", msg));
    await handleTradeResponse(result);
  }

  async executeSwap(fromToken: string, toToken: string, amount: string): Promise<void> {
    const prompt = tradingPrompts.swap(amount, fromToken, toToken, this.config.chain);
    const result = await execute(prompt, (msg) => console.log("  >", msg));
    await handleTradeResponse(result);
  }
}

// Usage
const bot = new TradingBot({ token: "ETH", chain: "Base", buyAmount: "50" });
await bot.executeBuy();
```

## Error Handling

Common errors and how to handle them in bots:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Insufficient balance | Not enough funds | Log and skip, or adjust amount |
| Token not found | Invalid symbol | Verify token exists, use contract address |
| High slippage | Price moved too much | Retry with smaller amount |
| Network congestion | Chain is busy | Implement retry with backoff |

```typescript
async function executeTradeWithRetry(prompt: string, maxRetries = 3): Promise<JobStatusResponse | null> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const result = await execute(prompt);

    if (result.status === "completed") return result;

    const error = result.error || "";
    const isRetryable = error.includes("slippage") || error.includes("congestion");

    if (!isRetryable || attempt === maxRetries) {
      console.error(`Trade failed: ${error}`);
      return result;
    }

    console.log(`Attempt ${attempt} failed, retrying...`);
    await new Promise((r) => setTimeout(r, 5000 * attempt));
  }
  return null;
}
```

## Chain Selection

```typescript
const CHAINS = {
  base: { native: "ETH", lowFees: true },
  polygon: { native: "MATIC", lowFees: true },
  ethereum: { native: "ETH", lowFees: false },
  solana: { native: "SOL", lowFees: true },
  unichain: { native: "ETH", lowFees: true },
};

function selectChain(preferLowFees = true): string {
  return preferLowFees ? "Base" : "Ethereum";
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-automation` - Limit orders and scheduled trades
- `bankr-market-research` - Price data for trading decisions
