---
name: Bankr Dev - Automation
description: This skill should be used when building a limit order system, implementing DCA strategies programmatically, creating TWAP execution, or building a grid trading bot. Covers TypeScript prompt templates like `automationPrompts.dca()`, automation ID parsing, DcaStrategyBuilder and GridTradingBot classes, and retry patterns. Triggered by "limit order bot code", "DCA automation TypeScript", "TWAP implementation", "grid trading bot".
version: 1.0.0
---

# Automation - Developer Guide

Build bots and applications that create and manage automated orders via the Bankr API.

## Overview

Automation through Bankr supports:
- Limit orders (buy/sell at target price)
- Stop loss orders (sell on price drop)
- DCA (Dollar Cost Averaging)
- TWAP (Time-Weighted Average Price)
- Scheduled commands
- Solana trigger orders (Jupiter)

**Supported Chains:** Base, Polygon, Ethereum (EVM), Solana (Jupiter triggers)

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for automation operations
const automationPrompts = {
  // Limit orders
  limitBuy: (token: string, price: string, amount?: string) =>
    amount
      ? `Set a limit order to buy ${amount} of ${token} at ${price}`
      : `Set a limit order to buy ${token} at ${price}`,

  limitSell: (token: string, price: string, amount?: string) =>
    amount
      ? `Limit sell ${amount} ${token} when it hits ${price}`
      : `Limit sell ${token} when it hits ${price}`,

  // Stop loss
  stopLoss: (token: string, price: string) =>
    `Set stop loss for my ${token} at ${price}`,

  stopLossPercent: (token: string, percent: number) =>
    `Stop loss: sell ${token} if it drops ${percent}%`,

  // DCA
  dca: (amount: string, token: string, frequency: string) =>
    `DCA ${amount} into ${token} ${frequency}`,

  dcaWithDuration: (amount: string, token: string, frequency: string, duration: string) =>
    `DCA ${amount} into ${token} ${frequency} for ${duration}`,

  // TWAP
  twap: (amount: string, token: string, duration: string) =>
    `TWAP buy ${amount} of ${token} over ${duration}`,

  twapSell: (amount: string, token: string, duration: string) =>
    `TWAP sell ${amount} ${token} over ${duration}`,

  // Scheduled commands
  schedule: (command: string, schedule: string) =>
    `${schedule}, ${command}`,

  // Management
  viewAutomations: () => "Show my automations",
  viewLimitOrders: () => "What limit orders do I have?",
  viewDcaOrders: () => "Show my DCA orders",
  cancelAutomation: (id: string) => `Cancel automation ${id}`,
  cancelAll: () => "Cancel all my automations",

  // History
  automationHistory: () => "Show automation history",
  executedOrders: () => "What orders executed today?",
};

// Frequency helpers
const FREQUENCIES = {
  hourly: "every hour",
  daily: "every day",
  weekly: "every week",
  monthly: "every month",
  custom: (cron: string) => cron,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface Automation {
  id: string;
  type: "limit" | "stop_loss" | "dca" | "twap" | "scheduled";
  token: string;
  status: "active" | "triggered" | "cancelled" | "expired";
  params: Record<string, any>;
  createdAt: Date;
  nextExecution?: Date;
}

function parseAutomationCreated(response: string): { id: string; type: string } | null {
  const idMatch = response.match(/Order ID:\s*(\w+)/i) || response.match(/ID:\s*(\w+)/i);
  if (!idMatch) return null;

  const lower = response.toLowerCase();
  let type = "scheduled";
  if (lower.includes("limit")) type = "limit";
  else if (lower.includes("dca")) type = "dca";
  else if (lower.includes("stop")) type = "stop_loss";
  else if (lower.includes("twap")) type = "twap";

  return { id: idMatch[1], type };
}

function parseAutomations(response: string): Automation[] {
  const automations: Automation[] = [];
  const sections = response.split(/\n\n?\d+\./);

  for (const section of sections) {
    const idMatch = section.match(/\((\w+)\)/);
    const statusMatch = section.match(/Status:\s*(\w+)/i);
    if (!idMatch) continue;

    const lower = section.toLowerCase();
    let type: Automation["type"] = "scheduled";
    if (lower.includes("limit")) type = "limit";
    else if (lower.includes("dca")) type = "dca";
    else if (lower.includes("stop")) type = "stop_loss";
    else if (lower.includes("twap")) type = "twap";

    automations.push({
      id: idMatch[1],
      type,
      token: "",
      status: (statusMatch?.[1].toLowerCase() as Automation["status"]) || "active",
      params: {},
      createdAt: new Date(),
    });
  }

  return automations;
}

async function handleAutomationResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Automation operation successful");
    console.log(result.response);

    const created = parseAutomationCreated(result.response || "");
    if (created) {
      console.log(`Created ${created.type} automation: ${created.id}`);
    }
  } else if (result.status === "failed") {
    console.error("Automation operation failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Automation-specific types
interface LimitOrderConfig {
  token: string;
  side: "buy" | "sell";
  price: number;
  amount?: number;
  amountUsd?: number;
  expiresIn?: string; // e.g., "24h", "7d"
}

interface StopLossConfig {
  token: string;
  triggerPrice?: number;
  triggerPercent?: number;
  sellAmount?: number; // Percentage of holdings
}

interface DcaConfig {
  token: string;
  amountUsd: number;
  frequency: "hourly" | "daily" | "weekly" | "monthly";
  duration?: string; // e.g., "1 month", "indefinite"
  startDate?: Date;
}

interface TwapConfig {
  token: string;
  side: "buy" | "sell";
  totalAmount: number;
  duration: string; // e.g., "24 hours", "4 hours"
  intervals?: number;
}

interface AutomationResult {
  success: boolean;
  automationId?: string;
  type: string;
  message: string;
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Limit Order Bot | Entry/exit at prices | `Set limit order to buy ETH at $3,000` |
| Risk Manager | Auto stop losses | `Set stop loss for my ETH at $2,800` |
| DCA Bot | Regular accumulation | `DCA $100 into ETH every week` |
| Large Order Bot | Execute with TWAP | `TWAP buy $5000 of ETH over 24 hours` |
| Portfolio Bot | Scheduled rebalancing | Schedule rebalance commands |

## Code Example: Automation Manager

```typescript
import { execute } from "./bankr-client";

interface ManagedAutomation {
  id: string;
  type: string;
  config: LimitOrderConfig | StopLossConfig | DcaConfig | TwapConfig;
  createdAt: Date;
  status: "active" | "triggered" | "cancelled";
}

class AutomationManager {
  private automations: Map<string, ManagedAutomation> = new Map();

  private async createAutomation(
    type: string,
    prompt: string,
    config: LimitOrderConfig | StopLossConfig | DcaConfig | TwapConfig
  ): Promise<AutomationResult> {
    const result = await execute(prompt, (msg) => console.log("  >", msg));
    if (result.status !== "completed") {
      return { success: false, type, message: result.error || `Failed to create ${type}` };
    }

    const created = parseAutomationCreated(result.response || "");
    if (!created) {
      return { success: false, type, message: `Failed to parse ${type} response` };
    }

    this.automations.set(created.id, { id: created.id, type, config, createdAt: new Date(), status: "active" });
    return { success: true, automationId: created.id, type, message: result.response || `${type} created` };
  }

  async createLimitOrder(config: LimitOrderConfig): Promise<AutomationResult> {
    const prompt = config.side === "buy"
      ? automationPrompts.limitBuy(config.token, `$${config.price}`, config.amountUsd ? `$${config.amountUsd}` : undefined)
      : automationPrompts.limitSell(config.token, `$${config.price}`, config.amount ? `${config.amount}` : undefined);
    return this.createAutomation("limit", prompt, config);
  }

  async createStopLoss(config: StopLossConfig): Promise<AutomationResult> {
    const prompt = config.triggerPrice
      ? automationPrompts.stopLoss(config.token, `$${config.triggerPrice}`)
      : automationPrompts.stopLossPercent(config.token, config.triggerPercent || 10);
    return this.createAutomation("stop_loss", prompt, config);
  }

  async createDca(config: DcaConfig): Promise<AutomationResult> {
    const prompt = config.duration
      ? automationPrompts.dcaWithDuration(`$${config.amountUsd}`, config.token, FREQUENCIES[config.frequency], config.duration)
      : automationPrompts.dca(`$${config.amountUsd}`, config.token, FREQUENCIES[config.frequency]);
    return this.createAutomation("dca", prompt, config);
  }

  async createTwap(config: TwapConfig): Promise<AutomationResult> {
    const prompt = config.side === "buy"
      ? automationPrompts.twap(`$${config.totalAmount}`, config.token, config.duration)
      : automationPrompts.twapSell(`${config.totalAmount}`, config.token, config.duration);
    return this.createAutomation("twap", prompt, config);
  }

  async refreshAutomations(): Promise<Automation[]> {
    const result = await execute(automationPrompts.viewAutomations());
    return result.status === "completed" && result.response ? parseAutomations(result.response) : [];
  }

  async cancelAutomation(id: string): Promise<boolean> {
    const result = await execute(automationPrompts.cancelAutomation(id));
    if (result.status === "completed") {
      const automation = this.automations.get(id);
      if (automation) automation.status = "cancelled";
      return true;
    }
    return false;
  }

  getAutomation(id: string): ManagedAutomation | undefined {
    return this.automations.get(id);
  }

  // Get all local automations
  getAllAutomations(): ManagedAutomation[] {
    return Array.from(this.automations.values());
  }
}

// Usage
const manager = new AutomationManager();

// Create limit order
const limitResult = await manager.createLimitOrder({
  token: "ETH",
  side: "buy",
  price: 3000,
  amountUsd: 500,
});
console.log("Limit order:", limitResult);

// Create stop loss
const stopResult = await manager.createStopLoss({
  token: "ETH",
  triggerPercent: 10,
});
console.log("Stop loss:", stopResult);

// Create DCA
const dcaResult = await manager.createDca({
  token: "ETH",
  amountUsd: 100,
  frequency: "weekly",
  duration: "3 months",
});
console.log("DCA:", dcaResult);

// Create TWAP
const twapResult = await manager.createTwap({
  token: "ETH",
  side: "buy",
  totalAmount: 5000,
  duration: "24 hours",
});
console.log("TWAP:", twapResult);

// Check all automations
const automations = await manager.refreshAutomations();
console.log("Active automations:", automations);
```

## DCA Strategy Builder

```typescript
interface DcaStrategy {
  name: string;
  tokens: { token: string; allocation: number }[]; // allocation as percentage
  totalAmountUsd: number;
  frequency: "daily" | "weekly" | "monthly";
}

class DcaStrategyBuilder {
  async executeStrategy(
    strategy: DcaStrategy,
    manager: AutomationManager
  ): Promise<AutomationResult[]> {
    const results: AutomationResult[] = [];

    for (const { token, allocation } of strategy.tokens) {
      const amountUsd = (strategy.totalAmountUsd * allocation) / 100;

      const result = await manager.createDca({
        token,
        amountUsd,
        frequency: strategy.frequency,
      });

      results.push(result);
    }

    return results;
  }
}

// Example: Create a diversified DCA strategy
const strategy: DcaStrategy = {
  name: "Balanced Crypto DCA",
  tokens: [
    { token: "BTC", allocation: 50 },
    { token: "ETH", allocation: 30 },
    { token: "SOL", allocation: 20 },
  ],
  totalAmountUsd: 500,
  frequency: "weekly",
};

const builder = new DcaStrategyBuilder();
const results = await builder.executeStrategy(strategy, manager);
console.log("DCA strategy created:", results);
```

## Grid Trading Bot

```typescript
interface GridConfig {
  token: string;
  lowerPrice: number;
  upperPrice: number;
  gridLines: number;
  amountPerGrid: number;
}

class GridTradingBot {
  async setupGrid(
    config: GridConfig,
    manager: AutomationManager
  ): Promise<AutomationResult[]> {
    const results: AutomationResult[] = [];
    const priceStep = (config.upperPrice - config.lowerPrice) / config.gridLines;

    // Create buy orders below current price
    for (let i = 0; i < config.gridLines / 2; i++) {
      const buyPrice = config.lowerPrice + priceStep * i;

      const result = await manager.createLimitOrder({
        token: config.token,
        side: "buy",
        price: buyPrice,
        amountUsd: config.amountPerGrid,
      });

      results.push(result);
    }

    // Create sell orders above current price
    for (let i = config.gridLines / 2; i < config.gridLines; i++) {
      const sellPrice = config.lowerPrice + priceStep * i;

      const result = await manager.createLimitOrder({
        token: config.token,
        side: "sell",
        price: sellPrice,
        amountUsd: config.amountPerGrid,
      });

      results.push(result);
    }

    return results;
  }
}

// Example: Set up grid trading
const gridBot = new GridTradingBot();
const gridResults = await gridBot.setupGrid(
  {
    token: "ETH",
    lowerPrice: 2800,
    upperPrice: 3200,
    gridLines: 10,
    amountPerGrid: 50,
  },
  manager
);
```

## Error Handling

Common errors and how to handle them:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Order not triggering | Price never reached | Check threshold |
| Insufficient balance | Funds depleted | Check before creating |
| Order cancelled | Expired or conflict | Re-create if needed |
| Rate limit | Too many orders | Space out creation |

```typescript
async function createWithRetry(
  createFn: () => Promise<AutomationResult>,
  maxRetries: number = 3
): Promise<AutomationResult> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const result = await createFn();

    if (result.success) {
      return result;
    }

    // Check for retryable errors
    if (result.message.includes("rate limit") || result.message.includes("timeout")) {
      console.log(`Attempt ${attempt} failed, retrying...`);
      await new Promise((r) => setTimeout(r, 5000 * attempt));
      continue;
    }

    // Non-retryable error
    return result;
  }

  return { success: false, type: "unknown", message: "Max retries exceeded" };
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-token-trading` - Immediate trades
- `bankr-market-research` - Price data for setting triggers
