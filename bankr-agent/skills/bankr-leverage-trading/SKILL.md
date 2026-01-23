---
name: Bankr Agent - Leverage Trading
description: This skill should be used when the user asks about "leverage trading", "long position", "short position", "Avantis", "derivatives", "forex trading", "commodities trading", "open a position", "close position", "stop loss", "take profit", or any leveraged trading operation. Provides guidance on Avantis perpetuals trading.
version: 1.0.0
---

# Bankr Leverage Trading

Trade with leverage using Avantis perpetuals on Base.

## Overview

Avantis is a decentralized perpetuals exchange offering:
- Long and short positions
- Up to 100x+ leverage
- Crypto, forex, and commodities markets
- Stop loss and take profit orders

**Chain**: Base

## Supported Markets

### Crypto

```
BTC, ETH, SOL, ARB, AVAX, BNB, DOGE, LINK, OP, MATIC, etc.
```

### Forex

```
EUR/USD, GBP/USD, USD/JPY, AUD/USD, USD/CAD, etc.
```

### Commodities

```
Gold (XAU), Silver (XAG), Oil (WTI), Natural Gas, etc.
```

## Operations

### Open Long Position

Profit when price goes up:

```
"Open a 5x long on ETH with $100"
"Long Bitcoin with 10x leverage"
"Buy ETH with leverage"
```

### Open Short Position

Profit when price goes down:

```
"Short ETH with 5x leverage and $50"
"Open a short position on Bitcoin"
"Go short on Gold with 3x leverage"
```

### View Positions

Check open positions:

```
"Show my Avantis positions"
"What leveraged positions do I have?"
"Check my open trades"
```

### Close Positions

Exit a position:

```
"Close my ETH long"
"Exit all my Avantis positions"
"Close my Bitcoin short"
```

## Position Parameters

### Leverage

- **Default**: 1x (if not specified)
- **Range**: 1x to 100x+ depending on asset
- Higher leverage = higher risk

```
"Long ETH with 5x leverage"   → 5x
"Short BTC 10x"               → 10x
"Long SOL"                    → 1x (default)
```

### Collateral

Specify the amount to use as collateral:

```
"Long ETH with $100 and 5x leverage"
"Short BTC using 0.1 ETH as collateral"
```

### Stop Loss

Automatically close position to limit losses:

```
"Long ETH 5x with stop loss at $3000"
"Short BTC with 10% stop loss"
```

### Take Profit

Automatically close position to lock in gains:

```
"Long ETH 5x with take profit at $4000"
"Short BTC with 20% take profit"
```

### Combined

```
"Long ETH 5x with $100, stop loss at $3000, take profit at $4000"
```

## Prompt Examples

### Opening Positions

```
"Open a 5x long on ETH with $100"
"Short Bitcoin with 10x leverage and $50 collateral"
"Long Gold with 2x leverage"
"Go long on EUR/USD with 20x"
"Open a short position on Oil"
```

### With Risk Management

```
"Long ETH 5x with stop loss at -10%"
"Short BTC 10x with take profit at 20%"
"Long SOL 3x with SL at $150 and TP at $200"
```

### Checking Positions

```
"Show my Avantis positions"
"What leverage positions do I have open?"
"Check my PnL on Avantis"
"List my open trades"
```

### Closing Positions

```
"Close my ETH long position"
"Exit all Avantis positions"
"Close my short on Bitcoin"
```

## Response Handling

### Position Opened

```json
{
  "status": "completed",
  "response": "Opened 5x long on ETH with $100 collateral. Entry: $3,245.67. Liquidation: $2,596.54",
  "transactions": [{
    "type": "avantisTrade",
    "metadata": {
      "chainId": 8453,
      "description": "Open 5x long ETH-USD"
    }
  }]
}
```

### Position Summary

```json
{
  "response": "Your Avantis positions:\n- ETH Long 5x: +$23.45 (7.2%)\n- BTC Short 3x: -$12.30 (-4.1%)"
}
```

## Risk Management

### Liquidation

- Positions are liquidated if losses exceed collateral
- Higher leverage = closer liquidation price
- Monitor positions and use stop losses

### Leverage Guidelines

| Risk Level | Leverage | Use Case |
|------------|----------|----------|
| Conservative | 1-3x | Long-term views |
| Moderate | 3-10x | Swing trading |
| Aggressive | 10-25x | Short-term scalps |
| High Risk | 25x+ | Experienced only |

### Best Practices

1. **Start with low leverage** (2-5x)
2. **Always use stop loss** to limit downside
3. **Don't over-leverage** - position sizing matters
4. **Monitor positions** - markets move fast
5. **Understand liquidation** - know your risk

## Common Issues

| Issue | Resolution |
|-------|------------|
| Insufficient collateral | Add more funds |
| Asset not supported | Check available markets |
| Liquidation | Position closed, collateral lost |
| High funding rate | Consider shorter hold time |

## Fees

- **Opening fee**: Small percentage of position size
- **Closing fee**: Small percentage of position size
- **Funding rate**: Periodic fee based on market conditions
- Gas costs on Base (very low)

