---
name: Bankr Dev - Portfolio
description: This skill should be used when building a portfolio dashboard, implementing balance monitoring across chains, creating allocation tracking, or building a rebalancing bot. Covers TypeScript prompt templates like `portfolioPrompts.chainBalance()`, portfolio parsing utilities, change detection algorithms, and PortfolioTracker class with analytics. Triggered by "portfolio tracker code", "balance monitoring TypeScript", "multi-chain dashboard", "rebalancing automation".
version: 1.0.0
---

# Portfolio - Developer Guide

Build bots and applications that query portfolio data via the Bankr API.

## Overview

Portfolio operations through Bankr support:
- Total portfolio value across all chains
- Chain-specific balances
- Token-specific holdings
- Real-time USD valuations

**Supported Chains:** Base, Polygon, Ethereum, Unichain, Solana

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for portfolio operations
const portfolioPrompts = {
  // Full portfolio
  totalPortfolio: () => "Show my portfolio",
  totalBalance: () => "What's my total balance?",
  allHoldings: () => "List all my crypto holdings",
  portfolioValue: () => "How much is my wallet worth?",

  // Chain-specific
  chainBalance: (chain: string) => `Show my ${chain} balance`,
  chainHoldings: (chain: string) => `What tokens do I have on ${chain}?`,

  // Token-specific
  tokenBalance: (token: string) => `How much ${token} do I have?`,
  tokenAcrossChains: (token: string) => `Show my ${token} on all chains`,

  // Comparisons
  largestHolding: () => "What's my largest holding?",
  tokenLocation: (token: string) => `Where do I have the most ${token}?`,
};

// Chain constants
const CHAINS = ["Base", "Polygon", "Ethereum", "Unichain", "Solana"] as const;
type Chain = (typeof CHAINS)[number];
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface TokenBalance {
  token: string;
  amount: number;
  valueUsd: number;
  chain: string;
}

interface ChainBalance {
  chain: string;
  totalValueUsd: number;
  tokens: TokenBalance[];
}

interface Portfolio {
  totalValueUsd: number;
  chains: ChainBalance[];
  lastUpdated: Date;
}

// Parse full portfolio response
function parsePortfolio(response: string): Portfolio {
  const portfolio: Portfolio = {
    totalValueUsd: 0,
    chains: [],
    lastUpdated: new Date(),
  };

  // Example format:
  // "Your portfolio ($12,345.67 total):
  //
  // Base:
  // - ETH: 2.5 ($8,125.00)
  // - USDC: 1,500.00 ($1,500.00)"

  // Extract total
  const totalMatch = response.match(/\(\$([0-9,.]+)\s+total\)/);
  if (totalMatch) {
    portfolio.totalValueUsd = parseFloat(totalMatch[1].replace(",", ""));
  }

  // Parse by chain
  const chainSections = response.split(/\n\n(?=[A-Za-z]+:)/);

  for (const section of chainSections) {
    const chainMatch = section.match(/^([A-Za-z]+):/);
    if (!chainMatch) continue;

    const chainBalance: ChainBalance = {
      chain: chainMatch[1],
      totalValueUsd: 0,
      tokens: [],
    };

    // Parse token lines
    const tokenLines = section.split("\n").filter((l) => l.startsWith("-"));
    for (const line of tokenLines) {
      const match = line.match(/-\s+(\w+):\s+([0-9,.]+)\s+\(\$([0-9,.]+)\)/);
      if (match) {
        const valueUsd = parseFloat(match[3].replace(",", ""));
        chainBalance.tokens.push({
          token: match[1],
          amount: parseFloat(match[2].replace(",", "")),
          valueUsd,
          chain: chainBalance.chain,
        });
        chainBalance.totalValueUsd += valueUsd;
      }
    }

    if (chainBalance.tokens.length > 0) {
      portfolio.chains.push(chainBalance);
    }
  }

  return portfolio;
}

// Parse single token balance
function parseTokenBalance(response: string, token: string): TokenBalance[] {
  const balances: TokenBalance[] = [];

  // Example: "Your ETH balance:
  // - Base: 2.5 ETH ($8,125.00)
  // - Ethereum: 0.5 ETH ($1,625.00)"

  const lines = response.split("\n").filter((l) => l.startsWith("-"));
  for (const line of lines) {
    const match = line.match(/-\s+(\w+):\s+([0-9,.]+)\s+\w+\s+\(\$([0-9,.]+)\)/);
    if (match) {
      balances.push({
        token,
        chain: match[1],
        amount: parseFloat(match[2].replace(",", "")),
        valueUsd: parseFloat(match[3].replace(",", "")),
      });
    }
  }

  return balances;
}

async function handlePortfolioResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Portfolio query successful");
    console.log(result.response);
  } else if (result.status === "failed") {
    console.error("Portfolio query failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Portfolio-specific types for your bot
interface PortfolioSnapshot {
  timestamp: Date;
  totalValueUsd: number;
  holdings: TokenBalance[];
}

interface BalanceChange {
  token: string;
  chain: string;
  previousAmount: number;
  currentAmount: number;
  change: number;
  changePercent: number;
}

interface PortfolioAlert {
  type: "balance_drop" | "balance_rise" | "token_received" | "token_sent";
  token: string;
  chain: string;
  amount: number;
  threshold: number;
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Dashboard | Display portfolio | `Show my portfolio` |
| Balance Check | Pre-trade validation | `How much ETH do I have?` |
| Chain Monitor | Track specific chain | `Show my Base balance` |
| Token Tracker | Follow specific token | `Show my USDC on all chains` |
| Alert System | Monitor for changes | Compare snapshots |

## Code Example: Portfolio Tracker

```typescript
import { execute } from "./bankr-client";

class PortfolioTracker {
  private snapshots: PortfolioSnapshot[] = [];

  async getPortfolio(): Promise<Portfolio | null> {
    const result = await execute(portfolioPrompts.totalPortfolio());
    return result.status === "completed" && result.response ? parsePortfolio(result.response) : null;
  }

  async getChainBalance(chain: string): Promise<ChainBalance | null> {
    const result = await execute(portfolioPrompts.chainBalance(chain));
    if (result.status !== "completed" || !result.response) return null;
    const portfolio = parsePortfolio(result.response);
    return portfolio.chains.find((c) => c.chain.toLowerCase() === chain.toLowerCase()) || null;
  }

  async getTokenBalance(token: string): Promise<TokenBalance[]> {
    const result = await execute(portfolioPrompts.tokenBalance(token));
    return result.status === "completed" && result.response ? parseTokenBalance(result.response, token) : [];
  }

  async takeSnapshot(): Promise<PortfolioSnapshot | null> {
    const portfolio = await this.getPortfolio();
    if (!portfolio) return null;

    const snapshot: PortfolioSnapshot = {
      timestamp: new Date(),
      totalValueUsd: portfolio.totalValueUsd,
      holdings: portfolio.chains.flatMap((c) => c.tokens),
    };

    this.snapshots.push(snapshot);
    if (this.snapshots.length > 100) this.snapshots.shift();

    return snapshot;
  }

  detectChanges(): BalanceChange[] {
    if (this.snapshots.length < 2) return [];

    const previous = this.snapshots[this.snapshots.length - 2];
    const current = this.snapshots[this.snapshots.length - 1];
    const changes: BalanceChange[] = [];

    for (const holding of current.holdings) {
      const prev = previous.holdings.find((h) => h.token === holding.token && h.chain === holding.chain);
      const previousAmount = prev?.amount || 0;
      const change = holding.amount - previousAmount;

      if (Math.abs(change) > 0.0001) {
        changes.push({
          token: holding.token,
          chain: holding.chain,
          previousAmount,
          currentAmount: holding.amount,
          change,
          changePercent: previousAmount > 0 ? (change / previousAmount) * 100 : 100,
        });
      }
    }

    for (const holding of previous.holdings) {
      const stillExists = current.holdings.find((h) => h.token === holding.token && h.chain === holding.chain);
      if (!stillExists && holding.amount > 0.0001) {
        changes.push({
          token: holding.token,
          chain: holding.chain,
          previousAmount: holding.amount,
          currentAmount: 0,
          change: -holding.amount,
          changePercent: -100,
        });
      }
    }

    return changes;
  }

  async hasEnoughBalance(token: string, amount: number, chain?: string): Promise<{ enough: boolean; available: number; chain?: string }> {
    const balances = await this.getTokenBalance(token);

    if (chain) {
      const chainBalance = balances.find((b) => b.chain.toLowerCase() === chain.toLowerCase());
      const available = chainBalance?.amount || 0;
      return { enough: available >= amount, available, chain };
    }

    const best = [...balances].sort((a, b) => b.amount - a.amount)[0];
    return best
      ? { enough: best.amount >= amount, available: best.amount, chain: best.chain }
      : { enough: false, available: 0 };
  }

  getValueHistory(): { timestamp: Date; value: number }[] {
    return this.snapshots.map((s) => ({ timestamp: s.timestamp, value: s.totalValueUsd }));
  }
}

// Usage
const tracker = new PortfolioTracker();

// Get full portfolio
const portfolio = await tracker.getPortfolio();
console.log("Total value:", portfolio?.totalValueUsd);
console.log("Chains:", portfolio?.chains.map((c) => c.chain));

// Check specific token
const ethBalances = await tracker.getTokenBalance("ETH");
console.log("ETH holdings:", ethBalances);

// Take periodic snapshots
setInterval(async () => {
  await tracker.takeSnapshot();
  const changes = tracker.detectChanges();
  if (changes.length > 0) {
    console.log("Balance changes detected:", changes);
  }
}, 60000);

// Check before trading
const canTrade = await tracker.hasEnoughBalance("ETH", 0.1, "Base");
console.log("Can trade 0.1 ETH on Base:", canTrade);
```

## Portfolio Analytics

```typescript
// Portfolio analysis utilities
function calculateAllocation(portfolio: Portfolio): Map<string, number> {
  const allocation = new Map<string, number>();

  for (const chain of portfolio.chains) {
    for (const token of chain.tokens) {
      const current = allocation.get(token.token) || 0;
      allocation.set(token.token, current + token.valueUsd);
    }
  }

  // Convert to percentages
  for (const [token, value] of allocation) {
    allocation.set(token, (value / portfolio.totalValueUsd) * 100);
  }

  return allocation;
}

function getTopHoldings(portfolio: Portfolio, count: number = 5): TokenBalance[] {
  const allTokens = portfolio.chains.flatMap((c) => c.tokens);
  return allTokens.sort((a, b) => b.valueUsd - a.valueUsd).slice(0, count);
}

function calculateChainExposure(portfolio: Portfolio): Map<string, number> {
  const exposure = new Map<string, number>();

  for (const chain of portfolio.chains) {
    exposure.set(chain.chain, (chain.totalValueUsd / portfolio.totalValueUsd) * 100);
  }

  return exposure;
}

// Rebalancing suggestions
interface RebalanceSuggestion {
  action: "buy" | "sell";
  token: string;
  amount: number;
  reason: string;
}

function suggestRebalance(
  portfolio: Portfolio,
  targetAllocation: Map<string, number>
): RebalanceSuggestion[] {
  const suggestions: RebalanceSuggestion[] = [];
  const currentAllocation = calculateAllocation(portfolio);

  for (const [token, targetPct] of targetAllocation) {
    const currentPct = currentAllocation.get(token) || 0;
    const diff = targetPct - currentPct;

    if (Math.abs(diff) > 2) {
      // Only suggest if >2% off
      const amount = (Math.abs(diff) / 100) * portfolio.totalValueUsd;
      suggestions.push({
        action: diff > 0 ? "buy" : "sell",
        token,
        amount,
        reason: `Currently ${currentPct.toFixed(1)}%, target ${targetPct}%`,
      });
    }
  }

  return suggestions;
}
```

## Dashboard Data Structure

```typescript
// Data structure for building dashboards
interface DashboardData {
  summary: {
    totalValueUsd: number;
    totalValueChange24h: number;
    totalValueChangePercent24h: number;
  };
  holdings: {
    token: string;
    totalAmount: number;
    totalValueUsd: number;
    priceUsd: number;
    change24h: number;
    chains: { chain: string; amount: number }[];
  }[];
  chainBreakdown: {
    chain: string;
    valueUsd: number;
    percentage: number;
    tokenCount: number;
  }[];
  recentActivity: BalanceChange[];
}

async function buildDashboardData(tracker: PortfolioTracker): Promise<DashboardData> {
  const portfolio = await tracker.getPortfolio();
  if (!portfolio) throw new Error("Failed to fetch portfolio");

  const history = tracker.getValueHistory();
  const value24hAgo = history.length > 24 ? history[history.length - 25]?.value : history[0]?.value;
  const valueChange = portfolio.totalValueUsd - (value24hAgo || portfolio.totalValueUsd);

  return {
    summary: {
      totalValueUsd: portfolio.totalValueUsd,
      totalValueChange24h: valueChange,
      totalValueChangePercent24h: value24hAgo ? (valueChange / value24hAgo) * 100 : 0,
    },
    holdings: [], // Aggregate tokens across chains
    chainBreakdown: portfolio.chains.map((c) => ({
      chain: c.chain,
      valueUsd: c.totalValueUsd,
      percentage: (c.totalValueUsd / portfolio.totalValueUsd) * 100,
      tokenCount: c.tokens.length,
    })),
    recentActivity: tracker.detectChanges(),
  };
}
```

## Error Handling

Common errors and how to handle them in bots:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Empty response | No holdings found | Return empty portfolio |
| Chain unavailable | RPC issues | Skip chain, retry later |
| Token not recognized | Unknown token in wallet | Log warning, include raw data |
| Price data unavailable | Market data gap | Use last known price or mark as stale |
| Timeout | Large portfolio | Increase timeout, paginate if possible |

```typescript
async function safeGetPortfolio(tracker: PortfolioTracker): Promise<Portfolio | null> {
  try {
    const portfolio = await tracker.getPortfolio();

    if (!portfolio || portfolio.chains.length === 0) {
      console.warn("Empty portfolio returned");
      return {
        totalValueUsd: 0,
        chains: [],
        lastUpdated: new Date(),
      };
    }

    return portfolio;
  } catch (err) {
    const error = err as Error;

    if (error.message.includes("timeout")) {
      console.error("Portfolio query timed out, retrying with longer timeout");
      // Implement retry logic
    }

    console.error("Portfolio error:", error.message);
    return null;
  }
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-token-trading` - Trade based on portfolio data
- `bankr-market-research` - Price data for valuations
