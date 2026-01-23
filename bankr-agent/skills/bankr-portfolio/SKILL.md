---
name: Bankr Agent - Portfolio
description: This skill should be used when the user asks about "my balance", "portfolio", "token holdings", "check balance", "how much do I have", "wallet balance", "what tokens do I own", "show my holdings", or any balance/portfolio query. Provides guidance on checking balances across chains.
version: 1.0.0
---

# Bankr Portfolio

Query token balances and portfolio across all supported chains.

## Overview

Check your holdings across:
- **EVM Chains**: Base, Polygon, Ethereum, Unichain
- **Solana**: SOL and SPL tokens
- **Aggregated View**: Total portfolio value

## Operations

### Check Total Portfolio

View all holdings across chains:

```
"Show my portfolio"
"What's my total balance?"
"How much crypto do I have?"
```

### Check Specific Chain

View holdings on one chain:

```
"Show my Base balance"
"What do I have on Ethereum?"
"Check my Solana holdings"
```

### Check Specific Token

View balance of a specific token:

```
"How much ETH do I have?"
"What's my USDC balance?"
"Check my BNKR holdings"
```

## Prompt Examples

### Full Portfolio

```
"Show my portfolio"
"What's my total balance?"
"List all my crypto holdings"
"How much is my wallet worth?"
```

### Chain-Specific

```
"Show my Base balance"
"What tokens do I have on Polygon?"
"Check my Ethereum holdings"
"Show my Solana balance"
```

### Token-Specific

```
"How much ETH do I have?"
"What's my USDC balance?"
"Check my BNKR balance"
"How many SOL do I own?"
```

### Comparisons

```
"Where do I have the most ETH?"
"Which chain has my USDC?"
"Show my ETH across all chains"
```

## Response Format

### Portfolio Summary

```json
{
  "response": "Your portfolio ($12,345.67 total):\n\nBase:\n- ETH: 2.5 ($8,125.00)\n- USDC: 1,500.00 ($1,500.00)\n- BNKR: 50,000 ($500.00)\n\nEthereum:\n- ETH: 0.5 ($1,625.00)\n\nSolana:\n- SOL: 25 ($595.67)"
}
```

### Single Chain

```json
{
  "response": "Your Base holdings:\n- ETH: 2.5 ($8,125.00)\n- USDC: 1,500.00 ($1,500.00)\n- BNKR: 50,000 ($500.00)\n\nTotal: $10,125.00"
}
```

### Single Token

```json
{
  "response": "Your ETH balance:\n- Base: 2.5 ETH ($8,125.00)\n- Ethereum: 0.5 ETH ($1,625.00)\n- Total: 3.0 ETH ($9,750.00)"
}
```

## Supported Assets

### Native Tokens

| Chain | Token |
|-------|-------|
| Base | ETH |
| Polygon | MATIC |
| Ethereum | ETH |
| Unichain | ETH |
| Solana | SOL |

### Common Tokens

Bankr tracks balances for popular tokens:
- Stablecoins: USDC, USDT, DAI
- DeFi: UNI, AAVE, LINK
- Memecoins: DOGE, SHIB, PEPE
- Project tokens: BNKR, ARB, OP

## Features

### USD Valuation

All balances include current USD value based on market prices.

### Multi-Chain Aggregation

See the same token across multiple chains:

```
"Show my USDC on all chains"
→ Base: 1,000 USDC
→ Polygon: 500 USDC
→ Ethereum: 250 USDC
→ Total: 1,750 USDC
```

### Real-Time Prices

Values reflect current market prices at query time.

## Use Cases

### Before Trading

Check if you have enough balance:

```
"Do I have enough ETH to swap for 100 USDC?"
"Check my USDC balance before I place a bet"
```

### Portfolio Review

Regular check-ins on holdings:

```
"How's my portfolio doing?"
"What's my largest holding?"
"Show portfolio breakdown by chain"
```

### After Transactions

Verify transaction completed:

```
"Did my ETH arrive?"
"Show my new BNKR balance"
```

## Common Questions

### "How much [token] do I have?"

Returns balance across all chains with USD value.

### "What's my balance on [chain]?"

Returns all token holdings on that specific chain.

### "Show my portfolio"

Returns complete breakdown by chain with USD totals.

### "What tokens do I own?"

Lists all tokens with non-zero balance.

## Notes

- **Read-only operation**: Balance queries don't execute transactions
- **Real-time data**: Prices and balances are current
- **All wallets**: Shows balance of connected wallet address
- **Dust filtering**: Very small balances may be excluded
