---
name: Bankr Agent - Polymarket
description: This skill should be used when the user asks about "Polymarket", "prediction markets", "betting odds", "place a bet", "check odds", "market predictions", "what are the odds", "bet on election", "sports betting", or any prediction market operation. Provides guidance on searching markets, placing bets, and managing positions.
version: 1.0.0
---

# Bankr Polymarket

Interact with Polymarket prediction markets.

## Overview

Polymarket is a decentralized prediction market platform where users can:
- Search and discover markets
- View odds and market details
- Place bets (buy shares)
- Manage and redeem positions

**Chain**: Polygon (requires USDC.e for betting)

## Operations

### Search Markets

Find prediction markets by topic:

```
"Search Polymarket for election markets"
"What prediction markets are trending?"
"Find markets about the Super Bowl"
```

### Check Odds

View current odds for specific events:

```
"What are the odds Trump wins the election?"
"Check the odds on the Eagles game"
"Show me the NYC mayor prediction"
```

### Place Bets

Buy shares in a prediction outcome:

```
"Bet $10 on Yes for Trump winning"
"Place $5 on the Eagles to win"
"Buy shares in the election market"
```

### View Positions

Check your current bets:

```
"Show my Polymarket positions"
"What bets do I have open?"
"Check my prediction market holdings"
```

### Redeem Positions

Claim winnings from resolved markets:

```
"Redeem my Polymarket positions"
"Claim my winnings from the election market"
```

## Betting Details

### Currency

- Bets are placed in **USDC** (specifically USDC.e on Polygon)
- Auto-bridging: Bankr can bridge USDC from other chains if needed

### Share System

- You buy shares of "Yes" or "No" outcomes
- Share price reflects market probability
- If your outcome wins, shares pay $1 each
- Profit = $1 - purchase price (per share)

### Example

```
Market: "Will it rain tomorrow?"
Yes shares: $0.60 (60% probability)
No shares: $0.40 (40% probability)

Bet $10 on "Yes":
- Buy ~16.67 shares at $0.60 each
- If Yes wins: Get $16.67 (profit: $6.67)
- If No wins: Get $0 (lose $10)
```

## Prompt Examples

### Searching Markets

```
"Search Polymarket for crypto markets"
"Find prediction markets about Bitcoin"
"What are the trending markets on Polymarket?"
"Show me sports betting markets"
```

### Checking Odds

```
"What are the odds Biden wins?"
"Check the probability of ETH reaching $5000"
"What's the current prediction for the Super Bowl?"
"Show odds for the upcoming election"
```

### Placing Bets

```
"Bet $10 on Trump winning"
"Place a $5 bet on Yes"
"Buy $20 of Yes shares on the election market"
"Bet on the Eagles to win this week"
```

### Managing Positions

```
"Show my Polymarket positions"
"What prediction market bets do I have?"
"Check my open positions"
"Redeem my winning positions"
```

## Response Handling

### Market Search Results

```json
{
  "response": "Found 5 markets about elections:\n1. Presidential Election 2024 - 65% Yes\n2. Senate Control - 52% Democrats\n..."
}
```

### Bet Confirmation

```json
{
  "status": "completed",
  "response": "Placed $10 bet on 'Yes' for Trump winning. Bought 15.38 shares at $0.65 each.",
  "transactions": [...]
}
```

### Position Summary

```json
{
  "response": "Your Polymarket positions:\n- Presidential Election: 20 Yes shares ($13 value)\n- Super Bowl: 10 No shares ($6.50 value)"
}
```

## Auto-Bridging

If you don't have USDC on Polygon:

1. Bankr detects insufficient USDC.e balance
2. Automatically bridges USDC from another chain
3. Converts to USDC.e if needed
4. Places the bet

**Note**: This may add a small delay and gas costs.

## Common Issues

| Issue | Resolution |
|-------|------------|
| Market not found | Try different search terms |
| Insufficient USDC | Add USDC or let auto-bridge |
| Market closed | Can't bet on resolved markets |
| Outcome unclear | Check market details for exact outcomes |

## Tips

- **Research first**: Search and check odds before betting
- **Start small**: Test with small amounts first
- **Check liquidity**: Low-liquidity markets may have worse prices
- **Watch fees**: Gas costs on Polygon are low but exist
- **Redeem promptly**: Claim winnings after markets resolve

## Market Types

| Category | Examples |
|----------|----------|
| Politics | Elections, legislation, appointments |
| Sports | Game outcomes, championships |
| Crypto | Price predictions, ETF approvals |
| Culture | Awards, entertainment events |
| Science | Discoveries, climate events |
