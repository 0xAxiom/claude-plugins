---
name: Bankr Dev - Leverage Trading
description: This skill should be used when building a leverage trading bot, implementing perpetual position management, creating risk management systems with stop-loss automation, or building an Avantis integration. Covers TypeScript prompt templates like `leveragePrompts.openLongWithRisk()`, liquidation price calculations, position PnL parsing, and LeverageTradingBot class with risk parameters. Triggered by "leverage bot code", "perpetual trading TypeScript", "Avantis integration", "stop loss automation".
version: 1.0.0
---

# Leverage Trading - Developer Guide

Build bots and applications that trade perpetuals via Avantis and the Bankr API.

## Overview

Leverage trading through Bankr (via Avantis) supports:
- Long and short positions
- Leverage from 1x to 100x+
- Crypto, forex, and commodities markets
- Stop loss and take profit orders

**Chain:** Base

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for leverage trading operations
const leveragePrompts = {
  // Open positions
  openLong: (asset: string, leverage: number, collateral: string) =>
    `Open a ${leverage}x long on ${asset} with ${collateral}`,

  openShort: (asset: string, leverage: number, collateral: string) =>
    `Open a ${leverage}x short on ${asset} with ${collateral}`,

  // With risk management
  openLongWithSL: (asset: string, leverage: number, collateral: string, stopLoss: string) =>
    `Open a ${leverage}x long on ${asset} with ${collateral}, stop loss at ${stopLoss}`,

  openLongWithTP: (asset: string, leverage: number, collateral: string, takeProfit: string) =>
    `Open a ${leverage}x long on ${asset} with ${collateral}, take profit at ${takeProfit}`,

  openLongWithRisk: (
    asset: string,
    leverage: number,
    collateral: string,
    stopLoss: string,
    takeProfit: string
  ) =>
    `Open a ${leverage}x long on ${asset} with ${collateral}, stop loss at ${stopLoss}, take profit at ${takeProfit}`,

  // View/close positions
  viewPositions: () => "Show my Avantis positions",
  closePosition: (asset: string, direction: "long" | "short") =>
    `Close my ${asset} ${direction} position`,
  closeAllPositions: () => "Close all my Avantis positions",

  // Check PnL
  checkPnl: () => "Check my PnL on Avantis",
};

// Asset categories
const SUPPORTED_ASSETS = {
  crypto: ["BTC", "ETH", "SOL", "ARB", "AVAX", "BNB", "DOGE", "LINK", "OP", "MATIC"],
  forex: ["EUR/USD", "GBP/USD", "USD/JPY", "AUD/USD", "USD/CAD"],
  commodities: ["Gold", "Silver", "Oil", "Natural Gas"],
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface PositionInfo {
  asset: string;
  direction: "long" | "short";
  leverage: number;
  collateral: number;
  entryPrice: number;
  liquidationPrice: number;
  pnl: number;
  pnlPercentage: number;
}

// Parse position opened response
function parsePositionOpened(response: string): Partial<PositionInfo> | null {
  // Example: "Opened 5x long on ETH with $100 collateral. Entry: $3,245.67. Liquidation: $2,596.54"
  const match = response.match(
    /Opened\s+(\d+)x\s+(long|short)\s+on\s+(\w+).+Entry:\s+\$([0-9,.]+).+Liquidation:\s+\$([0-9,.]+)/i
  );

  if (match) {
    return {
      leverage: parseInt(match[1]),
      direction: match[2].toLowerCase() as "long" | "short",
      asset: match[3],
      entryPrice: parseFloat(match[4].replace(",", "")),
      liquidationPrice: parseFloat(match[5].replace(",", "")),
    };
  }
  return null;
}

// Parse positions list
function parsePositions(response: string): PositionInfo[] {
  const positions: PositionInfo[] = [];
  // Example: "- ETH Long 5x: +$23.45 (7.2%)"
  const lines = response.split("\n").filter((l) => l.startsWith("-"));

  for (const line of lines) {
    const match = line.match(/-\s+(\w+)\s+(Long|Short)\s+(\d+)x:\s+([+-]?\$[0-9.]+)\s+\(([+-]?[0-9.]+)%\)/i);
    if (match) {
      positions.push({
        asset: match[1],
        direction: match[2].toLowerCase() as "long" | "short",
        leverage: parseInt(match[3]),
        collateral: 0, // Not in response
        entryPrice: 0,
        liquidationPrice: 0,
        pnl: parseFloat(match[4].replace("$", "").replace("+", "")),
        pnlPercentage: parseFloat(match[5].replace("+", "")),
      });
    }
  }
  return positions;
}

async function handleLeverageResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Leverage operation successful");
    console.log(result.response);

    // Handle transactions
    if (result.transactions) {
      for (const tx of result.transactions) {
        if (tx.metadata.description) {
          console.log(`Trade: ${tx.metadata.description}`);
        }
      }
    }
  } else if (result.status === "failed") {
    console.error("Leverage operation failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Leverage trading specific types
interface LeveragePosition {
  asset: string;
  direction: "long" | "short";
  leverage: number;
  collateral: number; // USD
  stopLoss?: number; // Price or percentage
  takeProfit?: number; // Price or percentage
}

interface PositionResult {
  success: boolean;
  asset: string;
  direction: "long" | "short";
  leverage: number;
  entryPrice: number;
  liquidationPrice: number;
  collateral: number;
}

// Risk parameters
interface RiskParams {
  maxLeverage: number;
  maxPositionSize: number; // USD
  defaultStopLoss: number; // Percentage
  defaultTakeProfit: number; // Percentage
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Trend Follower | Long/short based on trend | `Open a 5x long on ETH with $100` |
| Mean Reversion | Fade extreme moves | `Short BTC with 3x leverage` |
| Scalper | Quick in/out trades | Open/close with tight SL/TP |
| Risk Manager | Monitor liquidation risk | `Show my Avantis positions` |
| Auto-Closer | Close at PnL targets | `Close my ETH long position` |

## Code Example: Leverage Trading Bot

```typescript
import { execute } from "./bankr-client";

interface TradingSignal {
  asset: string;
  direction: "long" | "short";
  confidence: number; // 0-1
}

class LeverageTradingBot {
  private riskParams: RiskParams = {
    maxLeverage: 10,
    maxPositionSize: 500,
    defaultStopLoss: 5,
    defaultTakeProfit: 15,
  };

  calculatePositionSize(confidence: number): number {
    return Math.min(this.riskParams.maxPositionSize * confidence, this.riskParams.maxPositionSize);
  }

  calculateLeverage(confidence: number): number {
    return Math.max(1, Math.min(Math.ceil(confidence * this.riskParams.maxLeverage), this.riskParams.maxLeverage));
  }

  async openPosition(signal: TradingSignal): Promise<PositionResult | null> {
    const collateral = this.calculatePositionSize(signal.confidence);
    const leverage = this.calculateLeverage(signal.confidence);

    const prompt = leveragePrompts.openLongWithRisk(
      signal.asset,
      leverage,
      `$${collateral}`,
      `-${this.riskParams.defaultStopLoss}%`,
      `+${this.riskParams.defaultTakeProfit}%`
    );

    const result = await execute(prompt, (msg) => console.log("  >", msg));
    if (result.status !== "completed") return null;

    const parsed = parsePositionOpened(result.response || "");
    if (!parsed) return null;

    return {
      success: true,
      asset: signal.asset,
      direction: signal.direction,
      leverage: parsed.leverage || leverage,
      entryPrice: parsed.entryPrice || 0,
      liquidationPrice: parsed.liquidationPrice || 0,
      collateral,
    };
  }

  async getPositions(): Promise<PositionInfo[]> {
    const result = await execute(leveragePrompts.viewPositions());
    return result.status === "completed" && result.response ? parsePositions(result.response) : [];
  }

  async closePosition(asset: string, direction: "long" | "short"): Promise<boolean> {
    return (await execute(leveragePrompts.closePosition(asset, direction))).status === "completed";
  }

  async closeAllPositions(): Promise<boolean> {
    return (await execute(leveragePrompts.closeAllPositions())).status === "completed";
  }

  async monitorPositions(): Promise<void> {
    const positions = await this.getPositions();

    for (const position of positions) {
      const shouldClose =
        position.pnlPercentage <= -this.riskParams.defaultStopLoss ||
        position.pnlPercentage >= this.riskParams.defaultTakeProfit;

      if (shouldClose) {
        await this.closePosition(position.asset, position.direction);
      }
    }
  }
}

// Usage
const bot = new LeverageTradingBot();

// Open a position based on a signal
const signal: TradingSignal = {
  asset: "ETH",
  direction: "long",
  confidence: 0.7,
};

const position = await bot.openPosition(signal);
console.log("Opened position:", position);

// Monitor positions periodically
setInterval(() => bot.monitorPositions(), 30000);

// Get current positions
const positions = await bot.getPositions();
console.log("Current positions:", positions);
```

## Risk Management

```typescript
// Risk calculation utilities
function calculateLiquidationPrice(
  entryPrice: number,
  leverage: number,
  direction: "long" | "short",
  maintenanceMargin: number = 0.005 // 0.5%
): number {
  if (direction === "long") {
    // Liquidated when price drops enough to wipe collateral
    return entryPrice * (1 - 1 / leverage + maintenanceMargin);
  } else {
    // Liquidated when price rises enough
    return entryPrice * (1 + 1 / leverage - maintenanceMargin);
  }
}

function calculatePositionSize(
  collateral: number,
  leverage: number
): number {
  return collateral * leverage;
}

function calculatePnL(
  entryPrice: number,
  currentPrice: number,
  positionSize: number,
  direction: "long" | "short"
): number {
  const priceDiff = currentPrice - entryPrice;
  const pnl = direction === "long" ? priceDiff : -priceDiff;
  return (pnl / entryPrice) * positionSize;
}

function checkRisk(
  collateral: number,
  leverage: number,
  accountBalance: number,
  existingExposure: number
): { allowed: boolean; reason?: string } {
  const positionSize = collateral * leverage;
  const totalExposure = existingExposure + positionSize;

  if (totalExposure > accountBalance * 3) {
    return { allowed: false, reason: "Exceeds maximum total exposure" };
  }
  if (collateral > accountBalance * 0.2) {
    return { allowed: false, reason: "Position too large for account size" };
  }
  if (collateral > 100 && leverage > 10) {
    return { allowed: false, reason: "Reduce leverage for larger positions" };
  }

  return { allowed: true };
}
```

## Error Handling

Common errors and how to handle them:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Insufficient collateral | Not enough funds | Reduce position size |
| Asset not supported | Invalid market | Check supported list |
| Liquidation | Price moved against | Position closed, log loss |
| High funding rate | Expensive to hold | Consider closing |

```typescript
async function safeOpenPosition(
  position: LeveragePosition,
  accountBalance: number
): Promise<PositionResult | null> {
  // Risk check first
  const riskCheck = checkRisk(
    position.collateral,
    position.leverage,
    accountBalance,
    0 // Calculate existing exposure
  );

  if (!riskCheck.allowed) {
    console.error("Risk check failed:", riskCheck.reason);
    return null;
  }

  // Validate asset
  const allAssets = [
    ...SUPPORTED_ASSETS.crypto,
    ...SUPPORTED_ASSETS.forex,
    ...SUPPORTED_ASSETS.commodities,
  ];

  if (!allAssets.some((a) => position.asset.toUpperCase().includes(a.toUpperCase()))) {
    console.error("Asset not supported:", position.asset);
    return null;
  }

  // Open position
  const bot = new LeverageTradingBot();
  return await bot.openPosition({
    asset: position.asset,
    direction: position.direction,
    confidence: 1, // Use full specified size
  });
}
```

## Leverage Guidelines

| Risk Level | Leverage | Use Case |
|------------|----------|----------|
| Conservative | 1-3x | Long-term trend following |
| Moderate | 3-10x | Swing trading |
| Aggressive | 10-25x | Short-term scalping |
| High Risk | 25x+ | Experienced traders only |

```typescript
function getLeverageGuidance(
  timeframe: "scalp" | "swing" | "position",
  confidence: number
): number {
  const baseLeverage = {
    scalp: 15,
    swing: 5,
    position: 2,
  };

  return Math.ceil(baseLeverage[timeframe] * confidence);
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-market-research` - Price data for trading decisions
- `bankr-portfolio` - Check balances before trading
