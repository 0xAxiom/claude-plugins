---
name: Bankr Token Trading
description: This skill should be used when the user asks to "buy crypto", "sell tokens", "swap ETH", "trade on Base", "exchange tokens", "cross-chain swap", "bridge tokens", "convert ETH to WETH", or any token trading operation. Provides guidance on supported chains, amount formats, and swap operations.
version: 1.0.0
---

# Bankr Token Trading

Execute token trades and swaps across multiple blockchains.

## Supported Chains

| Chain | Network | Native Token |
|-------|---------|--------------|
| Base | EVM | ETH |
| Polygon | EVM | MATIC |
| Ethereum | EVM (Mainnet) | ETH |
| Unichain | EVM | ETH |
| Solana | Solana | SOL |

## Operations

### Same-Chain Swaps

Trade tokens on the same blockchain:

```
"Swap 0.1 ETH for USDC on Base"
"Buy $50 of BNKR on Base"
"Sell 100 USDC for ETH"
```

### Cross-Chain Swaps

Trade tokens across different blockchains:

```
"Bridge 0.5 ETH from Ethereum to Base"
"Swap ETH on Mainnet for SOL on Solana"
"Move 100 USDC from Polygon to Base"
```

### ETH/WETH Conversion

Convert between native ETH and wrapped ETH:

```
"Convert 0.1 ETH to WETH"
"Unwrap 0.5 WETH to ETH"
```

## Amount Formats

Bankr accepts three amount formats:

| Format | Example | Description |
|--------|---------|-------------|
| USD | `$50` | Dollar amount to spend |
| Percentage | `50%` | Percentage of your balance |
| Exact | `0.1 ETH` | Specific token amount |

### Examples

```
"Buy $50 of ETH"           → Spends $50 USD worth
"Sell 25% of my BNKR"      → Sells quarter of holdings
"Swap 0.1 ETH for USDC"    → Swaps exactly 0.1 ETH
```

## Trading Prompts

### Buying Tokens

```
"Buy $100 of ETH on Base"
"Buy 0.05 ETH worth of BNKR"
"Purchase some Solana"
```

### Selling Tokens

```
"Sell all my BNKR for ETH"
"Sell $50 worth of USDC"
"Sell 50% of my ETH holdings"
```

### Swapping Tokens

```
"Swap 0.1 ETH for USDC on Base"
"Exchange 100 USDC for BNKR"
"Trade my MATIC for ETH on Polygon"
```

### Cross-Chain Operations

```
"Bridge 1 ETH from Mainnet to Base"
"Swap ETH on Ethereum for USDC on Polygon"
"Move my USDC from Polygon to Solana"
```

## Chain Selection

### Default Behavior

- If no chain specified, Bankr selects the most appropriate chain
- Base is preferred for most operations due to low fees
- Cross-chain routes are automatically optimized

### Specifying Chains

Include chain name in the prompt:

```
"Buy ETH on Polygon"
"Swap tokens on Ethereum mainnet"
"Trade SOL on Solana"
```

## Slippage

- Default slippage tolerance is applied automatically
- For volatile tokens, Bankr adjusts slippage as needed
- If slippage is exceeded, the transaction fails safely

## Response Handling

Successful trades return:

```json
{
  "status": "completed",
  "response": "Successfully swapped 0.1 ETH for 324.56 USDC on Base",
  "transactions": [{
    "type": "swap",
    "metadata": {
      "__ORIGINAL_TX_DATA__": {
        "humanReadableMessage": "Swap 0.1 ETH for USDC",
        "inputTokenTicker": "ETH",
        "outputTokenTicker": "USDC",
        "inputTokenAmount": "0.1",
        "outputTokenAmount": "324.56"
      }
    }
  }]
}
```

## Common Issues

| Issue | Resolution |
|-------|------------|
| Insufficient balance | Reduce amount or add funds |
| Token not found | Check token symbol/address |
| High slippage | Try smaller amounts |
| Network congestion | Wait and retry |

## Example Prompts

**Simple trades:**
- "Buy $50 of ETH"
- "Sell my BNKR for USDC"
- "Swap 0.1 ETH for USDC"

**Chain-specific:**
- "Buy ETH on Base"
- "Swap SOL for USDC on Solana"
- "Trade on Polygon"

**Cross-chain:**
- "Bridge ETH from Mainnet to Base"
- "Move USDC to Solana"
- "Swap ETH on Ethereum for SOL"

**Advanced:**
- "Sell 50% of my ETH holdings"
- "Buy $100 of BNKR and stake it"
- "Convert all my WETH to ETH"
