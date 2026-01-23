---
name: Bankr Dev - Polymarket
description: This skill should be used when building a prediction market bot, implementing odds monitoring, creating an auto-betting system, or tracking Polymarket positions programmatically. Covers TypeScript prompt templates like `polymarketPrompts.betYes()`, odds parsing, expected value calculations, and PolymarketBot class with alert systems. Triggered by "Polymarket bot code", "betting prompt template", "odds tracking TypeScript", "prediction market automation".
version: 1.0.0
---

# Polymarket - Developer Guide

Build bots and applications that interact with Polymarket via the Bankr API.

## Overview

Polymarket operations through Bankr support:
- Searching and discovering markets
- Checking odds and probabilities
- Placing bets (buying shares)
- Viewing and redeeming positions

**Chain:** Polygon (uses USDC.e for betting)

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for Polymarket operations
const polymarketPrompts = {
  // Search markets
  search: (query: string) => `Search Polymarket for ${query}`,
  trending: () => "What prediction markets are trending on Polymarket?",

  // Check odds
  checkOdds: (event: string) => `What are the odds ${event}?`,
  marketDetails: (market: string) => `Show me details for the ${market} market on Polymarket`,

  // Place bets
  bet: (amount: string, outcome: string, market: string) =>
    `Bet ${amount} on ${outcome} for ${market}`,
  betYes: (amount: string, market: string) => `Bet ${amount} on Yes for ${market}`,
  betNo: (amount: string, market: string) => `Bet ${amount} on No for ${market}`,

  // Manage positions
  viewPositions: () => "Show my Polymarket positions",
  redeemPositions: () => "Redeem my Polymarket positions",
  checkPosition: (market: string) => `Check my position on ${market}`,
};

// Bet amount helpers
const formatBetAmount = {
  dollars: (amount: number) => `$${amount}`,
  shares: (amount: number) => `${amount} shares`,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface MarketInfo {
  title: string;
  yesPrice: number;
  noPrice: number;
  volume24h?: number;
  liquidity?: number;
}

interface Position {
  market: string;
  outcome: string;
  shares: number;
  avgPrice: number;
  currentValue: number;
  pnl: number;
}

// Parse market search results
function parseMarketResults(response: string): MarketInfo[] {
  // Response format varies, parse accordingly
  const markets: MarketInfo[] = [];
  // Parse the response text to extract market info
  // Example: "1. Presidential Election 2024 - 65% Yes"
  const lines = response.split("\n").filter((l) => l.match(/^\d+\./));

  for (const line of lines) {
    const match = line.match(/^(\d+)\.\s+(.+?)\s+-\s+(\d+)%\s+(Yes|No)?/);
    if (match) {
      const yesPrice = parseInt(match[3]) / 100;
      markets.push({
        title: match[2],
        yesPrice,
        noPrice: 1 - yesPrice,
      });
    }
  }
  return markets;
}

// Parse position results
function parsePositions(response: string): Position[] {
  const positions: Position[] = [];
  // Parse response text for position data
  // Example: "- Presidential Election: 20 Yes shares ($13 value)"
  const lines = response.split("\n").filter((l) => l.startsWith("-"));

  for (const line of lines) {
    const match = line.match(/-\s+(.+?):\s+(\d+)\s+(Yes|No)\s+shares\s+\(\$([0-9.]+)\s+value\)/);
    if (match) {
      positions.push({
        market: match[1],
        outcome: match[3],
        shares: parseInt(match[2]),
        avgPrice: 0, // Not always in response
        currentValue: parseFloat(match[4]),
        pnl: 0, // Calculate if needed
      });
    }
  }
  return positions;
}

async function handlePolymarketResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Polymarket operation successful");
    console.log(result.response);

    // Handle bet confirmations
    if (result.transactions?.length) {
      for (const tx of result.transactions) {
        console.log(`Transaction: ${tx.type}`);
      }
    }
  } else if (result.status === "failed") {
    console.error("Polymarket operation failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Polymarket-specific types for your bot
interface PolymarketBet {
  market: string;
  outcome: "yes" | "no";
  amount: number; // USD
  expectedShares?: number;
}

interface BetResult {
  success: boolean;
  shares: number;
  avgPrice: number;
  totalCost: number;
  market: string;
  outcome: string;
}

interface MarketOdds {
  market: string;
  yesPrice: number; // 0-1
  noPrice: number; // 0-1
  yesProbability: number; // Percentage
  noProbability: number;
  timestamp: Date;
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Odds Tracker | Monitor odds changes | `What are the odds Trump wins?` |
| Auto-Bettor | Bet when odds hit target | `Bet $10 on Yes for election` |
| Arbitrage | Find mispriced markets | Search + compare odds |
| Portfolio Tracker | Monitor all positions | `Show my Polymarket positions` |
| Auto-Redeemer | Claim winning positions | `Redeem my Polymarket positions` |

## Code Example: Prediction Market Bot

```typescript
import { execute } from "./bankr-client";

interface OddsAlert {
  market: string;
  outcome: "yes" | "no";
  targetOdds: number; // Price at which to bet
  betAmount: number;
}

class PolymarketBot {
  private alerts: OddsAlert[] = [];

  async searchMarkets(query: string): Promise<MarketInfo[]> {
    const result = await execute(polymarketPrompts.search(query));
    return result.status === "completed" && result.response ? parseMarketResults(result.response) : [];
  }

  async getOdds(event: string): Promise<MarketOdds | null> {
    const result = await execute(polymarketPrompts.checkOdds(event));
    if (result.status !== "completed" || !result.response) return null;

    const match = result.response.match(/(\d+)%\s+Yes/);
    if (!match) return null;

    const yesPct = parseInt(match[1]);
    return {
      market: event,
      yesPrice: yesPct / 100,
      noPrice: 1 - yesPct / 100,
      yesProbability: yesPct,
      noProbability: 100 - yesPct,
      timestamp: new Date(),
    };
  }

  async placeBet(bet: PolymarketBet): Promise<BetResult | null> {
    const prompt = bet.outcome === "yes"
      ? polymarketPrompts.betYes(`$${bet.amount}`, bet.market)
      : polymarketPrompts.betNo(`$${bet.amount}`, bet.market);

    const result = await execute(prompt, (msg) => console.log("  >", msg));
    if (result.status !== "completed") return null;

    const sharesMatch = result.response?.match(/(\d+\.?\d*)\s+shares/);
    const priceMatch = result.response?.match(/\$([0-9.]+)\s+each/);

    return {
      success: true,
      shares: sharesMatch ? parseFloat(sharesMatch[1]) : 0,
      avgPrice: priceMatch ? parseFloat(priceMatch[1]) : 0,
      totalCost: bet.amount,
      market: bet.market,
      outcome: bet.outcome,
    };
  }

  async getPositions(): Promise<Position[]> {
    const result = await execute(polymarketPrompts.viewPositions());
    return result.status === "completed" && result.response ? parsePositions(result.response) : [];
  }

  async redeemPositions(): Promise<boolean> {
    return (await execute(polymarketPrompts.redeemPositions())).status === "completed";
  }

  addAlert(alert: OddsAlert): void {
    this.alerts.push(alert);
  }

  async checkAlerts(): Promise<void> {
    for (const alert of this.alerts) {
      const odds = await this.getOdds(alert.market);
      if (!odds) continue;

      const currentPrice = alert.outcome === "yes" ? odds.yesPrice : odds.noPrice;
      if (currentPrice <= alert.targetOdds) {
        await this.placeBet({ market: alert.market, outcome: alert.outcome, amount: alert.betAmount });
        this.alerts = this.alerts.filter((a) => a !== alert);
      }
    }
  }
}

// Usage
const bot = new PolymarketBot();

// Search for markets
const markets = await bot.searchMarkets("election");
console.log("Found markets:", markets);

// Check odds
const odds = await bot.getOdds("Trump wins the election");
console.log("Current odds:", odds);

// Place a bet
const betResult = await bot.placeBet({
  market: "Presidential Election 2024",
  outcome: "yes",
  amount: 10,
});
console.log("Bet result:", betResult);

// Set up alerts
bot.addAlert({
  market: "Super Bowl",
  outcome: "yes",
  targetOdds: 0.45, // Bet when Yes shares are at 45 cents
  betAmount: 20,
});

// Check alerts periodically
setInterval(() => bot.checkAlerts(), 60000);
```

## Error Handling

Common errors and how to handle them in bots:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Market not found | Invalid search | Try different terms |
| Insufficient USDC | Not enough funds | Check balance first |
| Market closed | Already resolved | Skip and remove |
| Low liquidity | Thin order book | Use smaller amounts |

```typescript
async function safeBet(bot: PolymarketBot, bet: PolymarketBet): Promise<BetResult | null> {
  const odds = await bot.getOdds(bet.market);
  if (!odds) return null;

  const price = bet.outcome === "yes" ? odds.yesPrice : odds.noPrice;
  if (price < 0.01 || price > 0.99) return null; // Market likely closed

  return bot.placeBet(bet);
}
```

## Calculating Expected Value

```typescript
interface BetAnalysis {
  expectedValue: number;
  breakEvenProbability: number;
  potentialProfit: number;
  potentialLoss: number;
}

function analyzeBet(
  amount: number,
  price: number,
  yourProbability: number
): BetAnalysis {
  // Shares you get for your bet
  const shares = amount / price;

  // If you win, each share pays $1
  const potentialProfit = shares - amount;
  const potentialLoss = amount;

  // Expected value
  const expectedValue = yourProbability * potentialProfit - (1 - yourProbability) * potentialLoss;

  // Break-even probability (when EV = 0)
  const breakEvenProbability = price;

  return {
    expectedValue,
    breakEvenProbability,
    potentialProfit,
    potentialLoss,
  };
}

// Example: Should you bet $100 on Yes at 60 cents if you think probability is 70%?
const analysis = analyzeBet(100, 0.6, 0.7);
console.log(analysis);
// expectedValue: ~16.67 (positive = good bet)
// breakEvenProbability: 0.6 (60%)
// potentialProfit: ~66.67
// potentialLoss: 100
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-portfolio` - Check USDC balance for betting
