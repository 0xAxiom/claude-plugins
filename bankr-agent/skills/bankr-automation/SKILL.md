---
name: Bankr Automation
description: This skill should be used when the user asks about "limit order", "stop loss", "DCA", "TWAP", "schedule", "automate", "recurring order", "price trigger", "cancel automation", "my automations", or any automated trading operation. Provides guidance on limit orders, scheduled commands, and automated strategies.
version: 1.0.0
---

# Bankr Automation

Set up automated orders and scheduled trading strategies.

## Overview

Bankr supports various automation types:
- **Limit Orders**: Execute at target price
- **Stop Loss**: Sell when price drops
- **DCA**: Dollar Cost Averaging
- **TWAP**: Time-Weighted Average Price
- **Scheduled Commands**: Cron-based automation
- **Solana Triggers**: Jupiter trigger orders

## Order Types

### Limit Orders

Execute a trade when price reaches a target:

```
"Set a limit order to buy ETH at $3,000"
"Buy 0.5 ETH when price drops to $2,800"
"Limit order: sell BNKR when it hits $0.02"
```

**Parameters**:
- Token to buy/sell
- Target price
- Amount

### Stop Loss Orders

Automatically sell to limit losses:

```
"Set stop loss for my ETH at $2,500"
"Stop loss: sell 50% of BNKR if it drops 20%"
"Protect my SOL with stop loss at $150"
```

**Parameters**:
- Token to sell
- Trigger price or percentage
- Amount to sell

### DCA (Dollar Cost Averaging)

Invest fixed amounts at regular intervals:

```
"DCA $100 into ETH every week"
"Set up daily $50 Bitcoin purchases"
"Start a DCA: buy $25 of BNKR every day"
```

**Parameters**:
- Token to buy
- Amount per purchase
- Frequency (daily, weekly, monthly)
- Duration (optional)

### TWAP (Time-Weighted Average Price)

Spread a large order over time:

```
"TWAP: buy $1000 of ETH over 24 hours"
"Execute $500 ETH purchase using TWAP"
"Spread my sell order over 4 hours"
```

**Parameters**:
- Token and amount
- Total duration
- Number of intervals

### Scheduled Commands

Run any Bankr command on a schedule:

```
"Every morning, check my portfolio"
"Schedule a weekly rebalance"
"At 9am daily, check ETH price"
```

**Parameters**:
- Command to execute
- Schedule (cron or description)

## Prompt Examples

### Limit Orders

```
"Set a limit order to buy ETH at $3,000"
"Limit buy 0.5 BTC when price hits $90,000"
"Create limit sell for BNKR at $0.02"
"Buy SOL when it drops to $180"
```

### Stop Loss

```
"Set stop loss on my ETH at $2,800"
"Stop loss: sell all BNKR if it drops 30%"
"Protect my position with 15% stop loss"
"Stop loss for SOL at $140"
```

### DCA

```
"Set up DCA: $100 into ETH weekly"
"Start daily DCA of $50 into Bitcoin"
"DCA $25 into BNKR every day for a month"
"Begin weekly $200 ETH accumulation"
```

### TWAP

```
"TWAP buy $5000 of ETH over 48 hours"
"Spread my $2000 BTC purchase over 24 hours"
"Execute TWAP sell of 1000 USDC over 4 hours"
```

### Scheduled

```
"Every day at 9am, check ETH price"
"Weekly on Monday, show my portfolio"
"Schedule daily portfolio check"
```

### Managing Automations

```
"Show my automations"
"What limit orders do I have?"
"Cancel my ETH stop loss"
"List all my scheduled orders"
```

## Chain Support

### EVM Chains (Base, Polygon, Ethereum)

All order types supported:
- Limit orders
- Stop loss
- DCA
- TWAP
- Scheduled commands

### Solana

Uses Jupiter Trigger API:
- Limit orders
- Stop loss
- DCA orders

## Automation Management

### View Automations

```
"Show my automations"
"List my active orders"
"What DCA orders do I have?"
"Check my limit orders"
```

### Cancel Automations

```
"Cancel my ETH limit order"
"Remove stop loss on BNKR"
"Stop my DCA into Bitcoin"
"Cancel all my automations"
```

### Check History

```
"Show automation history"
"What orders executed today?"
"Check DCA execution history"
```

## Response Format

### Order Created

```json
{
  "status": "completed",
  "response": "Limit order created:\n- Buy 0.5 ETH when price reaches $3,000\n- Order ID: lmt_ABC123\n\nYou'll be notified when it executes."
}
```

### Automation List

```json
{
  "response": "Your active automations:\n\n1. Limit Order (lmt_ABC123)\n   Buy 0.5 ETH at $3,000\n   Status: Waiting\n\n2. DCA (dca_XYZ789)\n   $100 → ETH weekly\n   Next: Monday 9:00 AM\n\n3. Stop Loss (sl_DEF456)\n   Sell ETH if price < $2,500\n   Status: Active"
}
```

## Best Practices

### Limit Orders

- Set realistic target prices
- Consider market liquidity
- Monitor for partial fills

### Stop Loss

- Don't set too tight (normal volatility)
- Consider trailing stop loss for profits
- Review periodically

### DCA

- Good for long-term accumulation
- Reduces timing risk
- Set and forget approach

### TWAP

- Use for large orders
- Minimizes market impact
- Better average price

## Common Issues

| Issue | Resolution |
|-------|------------|
| Order not triggering | Check price threshold |
| Insufficient balance | Ensure funds available |
| Order cancelled | May expire or conflict |
| Network issues | Orders resume automatically |

## Fees

- **Order creation**: Minimal gas
- **Execution**: Standard trading fees
- **DCA**: Per-transaction fees apply
- **Cancellation**: Small gas cost

## Tips

1. **Start small** - Test with small amounts first
2. **Set alerts** - Get notified on execution
3. **Review regularly** - Update orders as needed
4. **Combine strategies** - DCA + stop loss
5. **Consider fees** - Factor into order sizes
