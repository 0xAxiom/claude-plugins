---
name: Bankr Market Research (Developer Guide)
description: This skill should be used when building a price alert system, implementing technical analysis automation, creating sentiment tracking, or building a trending token discovery bot. Covers TypeScript prompt templates like `marketPrompts.technicalAnalysis()`, price/TA/sentiment parsing, trading signal generation, and MarketMonitor class with alert callbacks. Triggered by "price alert bot code", "market research TypeScript", "sentiment tracking automation", "technical analysis implementation".
version: 1.0.0
---

# Market Research - Developer Guide

Build bots and applications that perform market research via the Bankr API.

## Overview

Market research through Bankr supports:
- Token price queries
- Market data (cap, volume, supply)
- Technical analysis indicators
- Social sentiment analysis
- Price charts
- Trending token discovery

**Supported Chains:** All chains - data aggregated from multiple sources

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for market research operations
const marketPrompts = {
  // Price queries
  price: (token: string) => `What's the price of ${token}?`,
  priceWithDetails: (token: string) => `Show me ${token} market data`,

  // Market data
  marketCap: (token: string) => `What's the market cap of ${token}?`,
  volume: (token: string) => `What's the 24h volume for ${token}?`,
  holders: (token: string) => `How many holders does ${token} have?`,

  // Technical analysis
  technicalAnalysis: (token: string) => `Do technical analysis on ${token}`,
  rsi: (token: string) => `What's the RSI for ${token}?`,
  priceAction: (token: string) => `Analyze ${token} price action`,

  // Sentiment
  sentiment: (token: string) => `What's the sentiment on ${token}?`,
  socialMentions: (token: string) => `Check social mentions for ${token}`,

  // Charts
  chart: (token: string, period?: string) =>
    period ? `Show ${token} price chart for ${period}` : `Show ${token} price chart`,

  // Discovery
  trending: () => "What tokens are trending today?",
  topGainers: () => "Show top gainers today",
  topLosers: () => "Show top losers today",
  trendingOnChain: (chain: string) => `What's trending on ${chain}?`,

  // Comparison
  compare: (token1: string, token2: string) => `Compare ${token1} vs ${token2}`,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse, RichData, Chart } from "./bankr-client";

interface PriceData {
  token: string;
  priceUsd: number;
  change24h: number;
  change7d?: number;
  marketCap?: number;
  volume24h?: number;
  ath?: number;
  atl?: number;
}

interface TechnicalIndicators {
  rsi: number;
  rsiSignal: "oversold" | "neutral" | "overbought";
  macd: "bullish" | "bearish" | "neutral";
  ma50: number;
  ma200: number;
  trend: "bullish" | "bearish" | "neutral";
}

interface SentimentData {
  overall: "bullish" | "bearish" | "neutral";
  bullishPercent: number;
  socialMentions: number;
  mentionChange: number;
  keyTopics: string[];
}

// Parse price response
function parsePriceData(response: string, token: string): PriceData | null {
  // Example: "Ethereum (ETH): $3,245.67\n24h: +2.3%\n7d: -1.5%\nMarket Cap: $390.2B"
  const priceMatch = response.match(/\$([0-9,.]+)/);
  const change24hMatch = response.match(/24h:\s*([+-]?[0-9.]+)%/);
  const change7dMatch = response.match(/7d:\s*([+-]?[0-9.]+)%/);
  const marketCapMatch = response.match(/Market Cap:\s*\$([0-9,.]+)([BMK])?/);

  if (!priceMatch) return null;

  let marketCap: number | undefined;
  if (marketCapMatch) {
    marketCap = parseFloat(marketCapMatch[1].replace(",", ""));
    const suffix = marketCapMatch[2];
    if (suffix === "B") marketCap *= 1e9;
    else if (suffix === "M") marketCap *= 1e6;
    else if (suffix === "K") marketCap *= 1e3;
  }

  return {
    token,
    priceUsd: parseFloat(priceMatch[1].replace(",", "")),
    change24h: change24hMatch ? parseFloat(change24hMatch[1]) : 0,
    change7d: change7dMatch ? parseFloat(change7dMatch[1]) : undefined,
    marketCap,
  };
}

// Parse technical analysis
function parseTechnicalAnalysis(response: string): TechnicalIndicators | null {
  const rsiMatch = response.match(/RSI:\s*(\d+)\s*\((\w+)\)/i);
  const macdMatch = response.match(/MACD:\s*(\w+)/i);
  const ma50Match = response.match(/50\s*MA:\s*\$([0-9,.]+)/i);
  const ma200Match = response.match(/200\s*MA:\s*\$([0-9,.]+)/i);

  if (!rsiMatch) return null;

  const rsi = parseInt(rsiMatch[1]);

  function getRsiSignal(value: number): "oversold" | "neutral" | "overbought" {
    if (value < 30) return "oversold";
    if (value > 70) return "overbought";
    return "neutral";
  }

  function getMacdSignal(match: RegExpMatchArray | null): "bullish" | "bearish" | "neutral" {
    const signal = match?.[1]?.toLowerCase() || "";
    if (signal.includes("bullish")) return "bullish";
    if (signal.includes("bearish")) return "bearish";
    return "neutral";
  }

  return {
    rsi,
    rsiSignal: getRsiSignal(rsi),
    macd: getMacdSignal(macdMatch),
    ma50: ma50Match ? parseFloat(ma50Match[1].replace(",", "")) : 0,
    ma200: ma200Match ? parseFloat(ma200Match[1].replace(",", "")) : 0,
    trend: "neutral",
  };
}

// Parse sentiment
function parseSentiment(response: string): SentimentData | null {
  // Example: "Sentiment: 67% Bullish\nSocial mentions: 12.5K (up 15%)"
  const sentimentMatch = response.match(/(\d+)%\s*(Bullish|Bearish|Neutral)/i);
  const mentionsMatch = response.match(/mentions:\s*([0-9.]+)K?\s*\((\w+)\s*(\d+)%\)/i);

  if (!sentimentMatch) return null;

  const bullishPercent = parseInt(sentimentMatch[1]);
  let overall: "bullish" | "bearish" | "neutral" = "neutral";
  if (bullishPercent > 60) overall = "bullish";
  else if (bullishPercent < 40) overall = "bearish";

  return {
    overall,
    bullishPercent,
    socialMentions: mentionsMatch ? parseFloat(mentionsMatch[1]) * 1000 : 0,
    mentionChange: mentionsMatch ? parseInt(mentionsMatch[3]) : 0,
    keyTopics: [], // Extract from response if available
  };
}

// Extract chart URL from rich data
function extractChartUrl(richData?: RichData[]): string | null {
  if (!richData) return null;
  const chart = richData.find((d) => d.type === "chart") as Chart | undefined;
  return chart?.url || null;
}

async function handleMarketResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Market research successful");
    console.log(result.response);

    // Check for charts
    const chartUrl = extractChartUrl(result.richData);
    if (chartUrl) {
      console.log("Chart available:", chartUrl);
    }
  } else if (result.status === "failed") {
    console.error("Market research failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Market research specific types
interface PriceAlert {
  token: string;
  type: "above" | "below" | "change";
  threshold: number;
  enabled: boolean;
}

interface TokenWatchlist {
  token: string;
  addedAt: Date;
  alertPrice?: number;
  notes?: string;
}

interface MarketSnapshot {
  timestamp: Date;
  prices: Map<string, number>;
  totalMarketCap?: number;
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Price Bot | Track token prices | `What's the price of ETH?` |
| Alert System | Notify on price changes | Poll + compare |
| TA Bot | Technical analysis | `Do technical analysis on BTC` |
| Sentiment Tracker | Monitor social mood | `What's the sentiment on SOL?` |
| Discovery Bot | Find trending tokens | `What tokens are trending?` |

## Code Example: Market Monitor Bot

```typescript
import { execute } from "./bankr-client";

interface PriceAlert {
  token: string;
  type: "above" | "below";
  threshold: number;
  callback: (price: number, token: string) => void;
}

class MarketMonitor {
  private priceCache: Map<string, PriceData> = new Map();
  private alerts: PriceAlert[] = [];
  private watchlist: string[] = [];

  async getPrice(token: string): Promise<PriceData | null> {
    const result = await execute(marketPrompts.price(token));
    if (result.status !== "completed" || !result.response) return null;

    const data = parsePriceData(result.response, token);
    if (data) this.priceCache.set(token, data);
    return data;
  }

  async getMarketData(token: string): Promise<PriceData | null> {
    const result = await execute(marketPrompts.priceWithDetails(token));
    return result.status === "completed" && result.response ? parsePriceData(result.response, token) : null;
  }

  async getTechnicalAnalysis(token: string): Promise<TechnicalIndicators | null> {
    const result = await execute(marketPrompts.technicalAnalysis(token));
    return result.status === "completed" && result.response ? parseTechnicalAnalysis(result.response) : null;
  }

  async getSentiment(token: string): Promise<SentimentData | null> {
    const result = await execute(marketPrompts.sentiment(token));
    return result.status === "completed" && result.response ? parseSentiment(result.response) : null;
  }

  async getChart(token: string, period = "7d"): Promise<string | null> {
    const result = await execute(marketPrompts.chart(token, period));
    return result.status === "completed" ? extractChartUrl(result.richData) : null;
  }

  async getTrending(): Promise<string[]> {
    const result = await execute(marketPrompts.trending());
    if (result.status !== "completed" || !result.response) return [];

    return result.response
      .split("\n")
      .map((line) => line.match(/\d+\.\s*(\w+)/)?.[1])
      .filter((token): token is string => Boolean(token));
  }

  addAlert(alert: PriceAlert): void {
    this.alerts.push(alert);
  }

  addToWatchlist(token: string): void {
    if (!this.watchlist.includes(token)) this.watchlist.push(token);
  }

  async checkAlerts(): Promise<void> {
    for (const alert of this.alerts) {
      const price = await this.getPrice(alert.token);
      if (!price) continue;

      const triggered =
        (alert.type === "above" && price.priceUsd >= alert.threshold) ||
        (alert.type === "below" && price.priceUsd <= alert.threshold);

      if (triggered) {
        alert.callback(price.priceUsd, alert.token);
        this.alerts = this.alerts.filter((a) => a !== alert);
      }
    }
  }

  async updateWatchlist(): Promise<Map<string, PriceData>> {
    const updates = new Map<string, PriceData>();

    for (const token of this.watchlist) {
      const data = await this.getPrice(token);
      if (data) updates.set(token, data);
      await new Promise((r) => setTimeout(r, 1000));
    }

    return updates;
  }

  getCachedPrice(token: string): PriceData | undefined {
    return this.priceCache.get(token);
  }
}

// Usage
const monitor = new MarketMonitor();

// Get current price
const ethPrice = await monitor.getPrice("ETH");
console.log("ETH price:", ethPrice);

// Get technical analysis
const btcTa = await monitor.getTechnicalAnalysis("BTC");
console.log("BTC TA:", btcTa);

// Get sentiment
const solSentiment = await monitor.getSentiment("SOL");
console.log("SOL sentiment:", solSentiment);

// Set up alerts
monitor.addAlert({
  token: "ETH",
  type: "below",
  threshold: 3000,
  callback: (price, token) => {
    console.log(`ALERT: ${token} dropped to $${price}!`);
  },
});

// Check alerts periodically
setInterval(() => monitor.checkAlerts(), 60000);

// Get trending
const trending = await monitor.getTrending();
console.log("Trending tokens:", trending);
```

## Price Alert System

```typescript
interface AlertConfig {
  token: string;
  conditions: AlertCondition[];
  notifyMethod: "console" | "webhook" | "callback";
  webhookUrl?: string;
}

type AlertCondition =
  | { type: "priceAbove"; value: number }
  | { type: "priceBelow"; value: number }
  | { type: "changeAbove"; value: number; period: "1h" | "24h" }
  | { type: "changeBelow"; value: number; period: "1h" | "24h" };

class AlertSystem {
  private configs: AlertConfig[] = [];
  private priceHistory: Map<string, { time: Date; price: number }[]> = new Map();

  addConfig(config: AlertConfig): void {
    this.configs.push(config);
  }

  async checkCondition(
    condition: AlertCondition,
    currentPrice: PriceData
  ): Promise<boolean> {
    switch (condition.type) {
      case "priceAbove":
        return currentPrice.priceUsd >= condition.value;
      case "priceBelow":
        return currentPrice.priceUsd <= condition.value;
      case "changeAbove":
        return (
          (condition.period === "24h" ? currentPrice.change24h : 0) >=
          condition.value
        );
      case "changeBelow":
        return (
          (condition.period === "24h" ? currentPrice.change24h : 0) <=
          condition.value
        );
      default:
        return false;
    }
  }

  async notify(config: AlertConfig, price: PriceData): Promise<void> {
    const message = `Alert: ${config.token} at $${price.priceUsd} (${price.change24h}% 24h)`;

    switch (config.notifyMethod) {
      case "console":
        console.log(message);
        break;
      case "webhook":
        if (config.webhookUrl) {
          await fetch(config.webhookUrl, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ token: config.token, price, message }),
          });
        }
        break;
    }
  }
}
```

## Trading Signal Generation

```typescript
interface TradingSignal {
  token: string;
  action: "buy" | "sell" | "hold";
  confidence: number;
  reasons: string[];
  timestamp: Date;
}

async function generateSignal(
  monitor: MarketMonitor,
  token: string
): Promise<TradingSignal> {
  const [price, ta, sentiment] = await Promise.all([
    monitor.getPrice(token),
    monitor.getTechnicalAnalysis(token),
    monitor.getSentiment(token),
  ]);

  const reasons: string[] = [];
  let score = 0;

  // Technical indicators
  if (ta) {
    if (ta.rsiSignal === "oversold") {
      score += 2;
      reasons.push("RSI oversold (potential buy)");
    } else if (ta.rsiSignal === "overbought") {
      score -= 2;
      reasons.push("RSI overbought (potential sell)");
    }

    if (ta.macd === "bullish") {
      score += 1;
      reasons.push("MACD bullish");
    } else if (ta.macd === "bearish") {
      score -= 1;
      reasons.push("MACD bearish");
    }
  }

  // Sentiment
  if (sentiment) {
    if (sentiment.overall === "bullish") {
      score += 1;
      reasons.push(`Sentiment bullish (${sentiment.bullishPercent}%)`);
    } else if (sentiment.overall === "bearish") {
      score -= 1;
      reasons.push(`Sentiment bearish`);
    }
  }

  // Price momentum
  if (price && price.change24h) {
    if (price.change24h > 5) {
      score += 1;
      reasons.push(`Strong momentum (+${price.change24h}%)`);
    } else if (price.change24h < -5) {
      score -= 1;
      reasons.push(`Negative momentum (${price.change24h}%)`);
    }
  }

  // Determine action
  let action: "buy" | "sell" | "hold" = "hold";
  if (score >= 2) action = "buy";
  else if (score <= -2) action = "sell";

  return {
    token,
    action,
    confidence: Math.min(Math.abs(score) / 4, 1),
    reasons,
    timestamp: new Date(),
  };
}
```

## Error Handling

Common errors and how to handle them in bots:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Token not found | Invalid symbol | Check symbol, try alternatives |
| Price unavailable | Delisted or new token | Skip or use external source |
| Chart generation failed | Server issue | Retry or skip chart |
| Sentiment data empty | Insufficient social data | Return neutral sentiment |
| Rate limited | Too many requests | Implement backoff, cache results |

```typescript
async function safeGetPrice(
  monitor: MarketMonitor,
  token: string
): Promise<PriceData | null> {
  try {
    const price = await monitor.getPrice(token);

    if (!price) {
      console.warn(`No price data for ${token}`);
      return null;
    }

    return price;
  } catch (err) {
    const error = err as Error;

    if (error.message.includes("not found")) {
      console.error(`Token ${token} not found, check symbol`);
    } else if (error.message.includes("rate limit")) {
      console.error("Rate limited, implementing backoff");
      await new Promise((r) => setTimeout(r, 5000));
    }

    return null;
  }
}

async function safeGetMarketData(
  monitor: MarketMonitor,
  token: string
): Promise<{ price?: PriceData; ta?: TechnicalIndicators; sentiment?: SentimentData }> {
  const [price, ta, sentiment] = await Promise.allSettled([
    monitor.getPrice(token),
    monitor.getTechnicalAnalysis(token),
    monitor.getSentiment(token),
  ]);

  return {
    price: price.status === "fulfilled" ? price.value || undefined : undefined,
    ta: ta.status === "fulfilled" ? ta.value || undefined : undefined,
    sentiment: sentiment.status === "fulfilled" ? sentiment.value || undefined : undefined,
  };
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-token-trading` - Execute trades based on research
- `bankr-automation` - Set up automated responses to signals
