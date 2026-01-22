---
name: Bankr Market Research
description: This skill should be used when the user asks about "token price", "market data", "technical analysis", "sentiment", "trending tokens", "price chart", "market cap", "token research", "what's the price of", "analyze token", or any market research query. Provides guidance on token research, analysis, and market intelligence.
version: 1.0.0
---

# Bankr Market Research

Research tokens and analyze market data.

## Overview

Bankr provides comprehensive market intelligence:
- Token search across chains
- Price and market data
- Technical analysis
- Social sentiment
- Price charts
- Trending tokens

## Operations

### Price Queries

Get current token prices:

```
"What's the price of ETH?"
"How much is Bitcoin worth?"
"Check the SOL price"
```

### Market Data

Get comprehensive token data:

```
"Show me ETH market data"
"What's the market cap of BNKR?"
"Get volume for Solana"
```

### Technical Analysis

Analyze price patterns:

```
"Do technical analysis on ETH"
"Show RSI for Bitcoin"
"Analyze BTC price action"
```

### Social Sentiment

Check community sentiment:

```
"What's the sentiment on ETH?"
"Is the community bullish on SOL?"
"Check social mentions for BNKR"
```

### Price Charts

Generate price visualizations:

```
"Show me ETH price chart"
"Generate BTC chart for last week"
"Chart SOL price history"
```

### Trending Tokens

Discover what's popular:

```
"What tokens are trending?"
"Show top gainers today"
"What's hot in crypto?"
```

## Prompt Examples

### Price Queries

```
"What's the price of Ethereum?"
"How much is 1 BTC in USD?"
"Check BNKR price"
"What's SOL trading at?"
```

### Market Data

```
"Show market cap for ETH"
"What's the 24h volume for Bitcoin?"
"Get trading data for Solana"
"How many holders does BNKR have?"
```

### Technical Analysis

```
"Do technical analysis on Bitcoin"
"What's the RSI for ETH?"
"Analyze BTC price trends"
"Is ETH overbought?"
```

### Sentiment

```
"What's the sentiment on Solana?"
"Is the market bullish on ETH?"
"Check Twitter sentiment for BTC"
"How is the community feeling about BNKR?"
```

### Charts

```
"Show ETH price chart"
"Generate weekly BTC chart"
"Chart SOL price for last month"
"Visual of ETH performance"
```

### Discovery

```
"What tokens are trending today?"
"Show top gainers this week"
"What's hot on Base?"
"Find trending memecoins"
```

### Comparison

```
"Compare ETH vs SOL"
"Which is better: BTC or ETH?"
"Show ETH and BTC side by side"
```

## Data Points

### Price Data

| Metric | Description |
|--------|-------------|
| Price | Current USD price |
| 24h Change | Percentage change |
| 7d Change | Weekly performance |
| ATH | All-time high |
| ATL | All-time low |

### Market Metrics

| Metric | Description |
|--------|-------------|
| Market Cap | Total value |
| Volume (24h) | Trading volume |
| Circulating Supply | Tokens in market |
| Total Supply | All tokens |
| Holders | Number of wallets |

### Technical Indicators

| Indicator | Description |
|-----------|-------------|
| RSI | Relative Strength Index |
| MACD | Moving Average Convergence |
| MA | Moving Averages (50, 200) |
| Support/Resistance | Key price levels |

## Response Format

### Price Query

```json
{
  "response": "Ethereum (ETH): $3,245.67\n24h: +2.3%\n7d: -1.5%\nMarket Cap: $390.2B"
}
```

### Technical Analysis

```json
{
  "response": "ETH Technical Analysis:\n- RSI: 58 (Neutral)\n- MACD: Bullish crossover\n- 50 MA: $3,180 (above)\n- 200 MA: $2,950 (above)\n\nOutlook: Moderately bullish"
}
```

### Sentiment

```json
{
  "response": "ETH Sentiment Analysis:\n- Social mentions: 12.5K (up 15%)\n- Sentiment: 67% Bullish\n- Key topics: ETF, staking, upgrades\n- Community mood: Optimistic"
}
```

### Chart

Returns a URL or base64 image in `richData`:

```json
{
  "response": "Here's the ETH price chart for the last 7 days:",
  "richData": [{
    "type": "chart",
    "url": "https://..."
  }]
}
```

## Supported Chains

Token research works across:
- **Base**: Native and ERC20 tokens
- **Polygon**: Native and ERC20 tokens
- **Ethereum**: All mainnet tokens
- **Solana**: SOL and SPL tokens
- **Unichain**: Emerging tokens

## Token Search

Find tokens by name or symbol:

```
"Search for BNKR token"
"Find tokens called Bankr"
"What is the contract for PEPE on Base?"
```

## Use Cases

### Before Trading

```
"What's the price of ETH?" → Check current price
"Analyze ETH technicals" → Evaluate entry point
"Check ETH sentiment" → Gauge market mood
```

### Market Research

```
"What's trending today?" → Find opportunities
"Compare SOL vs ETH" → Evaluate options
"Show top Base tokens" → Discover new projects
```

### Portfolio Analysis

```
"How is BNKR performing?" → Track holdings
"Chart my ETH performance" → Visualize gains
"What's the outlook for SOL?" → Plan strategy
```

## Limitations

- **Historical data**: Limited to available timeframes
- **Sentiment**: Based on available social data
- **New tokens**: May have limited data
- **Predictions**: Not investment advice
