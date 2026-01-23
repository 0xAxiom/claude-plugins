---
name: Bankr Agent - Transfers
description: This skill should be used when the user asks to "send tokens", "transfer ETH", "send to ENS", "transfer to wallet", "send to @username", "transfer to Farcaster", "send to Twitter handle", or any asset transfer operation. Provides guidance on recipient resolution and transfer formats.
version: 1.0.0
---

# Bankr Transfers

Transfer tokens to addresses, ENS names, or social handles.

## Supported Transfers

### EVM Chains
- Native tokens (ETH, MATIC)
- ERC20 tokens (USDC, BNKR, etc.)
- Supported on: Base, Polygon, Ethereum, Unichain

### Solana
- Native SOL
- SPL tokens

## Recipient Formats

Bankr resolves recipients from multiple formats:

| Format | Example | Description |
|--------|---------|-------------|
| Address | `0x1234...abcd` | Direct wallet address |
| ENS | `vitalik.eth` | Ethereum Name Service |
| Twitter | `@elonmusk` | Twitter/X username |
| Farcaster | `@dwr.eth` | Farcaster username |
| Telegram | `@username` | Telegram handle |

### Resolution Priority

When using social handles, Bankr:
1. Looks up the username on the platform
2. Finds their linked wallet address
3. Validates the address before sending

## Transfer Prompts

### To Address

```
"Send 0.1 ETH to 0x1234567890abcdef..."
"Transfer 100 USDC to 0xabcd..."
```

### To ENS Name

```
"Send 0.5 ETH to vitalik.eth"
"Transfer USDC to mydomain.eth"
```

### To Social Handle

```
"Send $50 of ETH to @elonmusk on Twitter"
"Transfer 0.1 ETH to @dwr.eth on Farcaster"
"Send 100 USDC to @username on Telegram"
```

## Amount Formats

Same as trading - three formats supported:

| Format | Example | Description |
|--------|---------|-------------|
| USD | `$50` | Dollar amount |
| Percentage | `50%` | Percentage of balance |
| Exact | `0.1 ETH` | Specific amount |

### Examples

```
"Send $100 worth of ETH to vitalik.eth"
"Transfer 50% of my USDC to @friend"
"Send exactly 0.5 ETH to 0x..."
```

## Chain Selection

### Automatic

If not specified, Bankr selects the appropriate chain:
- Checks where recipient has activity
- Considers gas costs
- Prefers Base for low fees

### Manual

Specify chain in the prompt:

```
"Send ETH on Base to vitalik.eth"
"Transfer USDC on Polygon to 0x..."
"Send SOL on Solana to @username"
```

## Supported Transfer Types

- **Native tokens** - ETH, MATIC, etc.
- **ERC20 tokens** - USDC, USDT, any token
- **NFTs** - See NFT Operations skill

## Common Issues

| Issue | Resolution |
|-------|------------|
| ENS not found | Verify the ENS name exists |
| Social handle not found | Check username is correct |
| No linked wallet | User hasn't linked wallet to social |
| Insufficient balance | Reduce amount or add funds |
| Invalid address | Check address format |

## Security Notes

- Always verify recipient before confirming
- Double-check addresses for transfers
- Social handle resolution shows the resolved address
- Large transfers may require additional confirmation

## Example Prompts

**To addresses:**
- "Send 0.5 ETH to 0x1234..."
- "Transfer 100 USDC to 0xabcd..."

**To ENS:**
- "Send 1 ETH to vitalik.eth"
- "Transfer $50 of USDC to mydomain.eth"

**To social handles:**
- "Send $20 of ETH to @friend on Twitter"
- "Transfer 0.1 ETH to @user on Farcaster"
- "Send USDC to @contact on Telegram"

**With chain specified:**
- "Send ETH on Base to vitalik.eth"
- "Transfer USDC on Polygon to @friend"

**Percentage amounts:**
- "Send 10% of my ETH to @friend"
- "Transfer half my USDC to vitalik.eth"
