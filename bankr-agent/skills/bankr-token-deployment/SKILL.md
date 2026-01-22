---
name: Bankr Token Deployment
description: This skill should be used when the user asks to "deploy token", "create token", "launch token", "Clanker", "claim fees", "token metadata", "update token", "mint new token", or any token deployment operation. Provides guidance on deploying ERC20 tokens via Clanker.
version: 1.0.0
---

# Bankr Token Deployment

Deploy and manage ERC20 tokens using Clanker.

## Overview

Clanker enables token deployment on:
- **Base**: Primary deployment chain
- **Unichain**: Secondary option

## Operations

### Deploy Token

Create a new ERC20 token:

```
"Deploy a token called MyToken with symbol MTK"
"Create a new memecoin called DOGE2"
"Launch a token named BankrFan"
```

### Claim Fees

Collect unclaimed trading fees:

```
"Claim fees for my token"
"Check unclaimed Clanker fees"
"Collect my token rewards"
```

### Update Metadata

Modify token information:

```
"Update my token description"
"Change token social links"
"Add audit URL to my token"
```

### Update Image

Change token logo:

```
"Update my token image"
"Change logo for MyToken"
```

## Deployment Details

### Required Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| Name | Token name | "MyToken" |
| Symbol | Ticker (3-5 chars) | "MTK" |

### Optional Parameters

| Parameter | Description |
|-----------|-------------|
| Description | Token description |
| Image | Logo URL or upload |
| Website | Project website |
| Twitter | Twitter/X handle |
| Telegram | Telegram group |
| Discord | Discord server |

## Prompt Examples

### Deploying

```
"Deploy a token called BankrFan with symbol BFAN"
"Create a memecoin: name=DogeKiller, symbol=DOGEK"
"Launch a token named CryptoGems (GEMS)"
"Deploy new token: MyProject (PROJ)"
```

### With Metadata

```
"Deploy token MyToken (MTK) with description 'Community token for fans'"
"Create token with website myproject.com and Twitter @myproject"
"Launch token with logo [image URL]"
```

### Fee Management

```
"Claim fees for my token MTK"
"Check my Clanker fees"
"How much in unclaimed fees do I have?"
"Claim all my token fees"
```

### Legacy Fees

```
"Claim legacy Clanker fees"
"Check old token fees"
```

### Metadata Updates

```
"Update description for MyToken"
"Add Twitter link to my token"
"Update audit URL for MTK"
"Change my token's website"
```

### Image Updates

```
"Update logo for MyToken"
"Change my token image to [URL]"
```

### Reward Recipient

```
"Update reward recipient for my token"
"Change fee collection address"
```

## Rate Limits

| User Type | Daily Limit |
|-----------|-------------|
| Standard Users | 1 token/day |
| Bankr Club Members | 10 tokens/day |

## Response Format

### Deployment Success

```json
{
  "status": "completed",
  "response": "Token deployed successfully!\n\nName: MyToken\nSymbol: MTK\nChain: Base\nContract: 0x1234...abcd\n\nYour token is now live and tradeable!",
  "transactions": [{
    "type": "deploy_token",
    "metadata": {...}
  }]
}
```

### Fee Claim

```json
{
  "response": "Fees claimed successfully!\n\nToken: MyToken (MTK)\nAmount: 0.5 ETH\nTransaction: 0x..."
}
```

### Fee Check

```json
{
  "response": "Your unclaimed Clanker fees:\n\nMyToken (MTK): 0.5 ETH\nOtherToken (OTK): 0.1 ETH\n\nTotal: 0.6 ETH"
}
```

## Token Lifecycle

### 1. Planning

Before deploying:
- Choose memorable name and symbol
- Prepare logo/branding
- Write token description
- Set up social links

### 2. Deployment

```
"Deploy token MyToken (MTK)"
```

Token is created with:
- Initial supply
- Trading enabled on DEX
- Fee mechanism active

### 3. Management

After deployment:
- Update metadata as needed
- Claim fees regularly
- Engage community

## Fee Structure

### Trading Fees

- Small fee on each trade
- Accumulated for token creator
- Claimable anytime

### Legacy Fees

- Fees from older Clanker versions
- Claim separately with legacy command
- Base chain only

## Best Practices

### Naming

- **Unique**: Stand out from others
- **Memorable**: Easy to remember
- **Clear**: Avoid confusion with established tokens

### Symbol

- 3-5 characters recommended
- All caps convention
- Avoid existing symbols

### Metadata

- Add description immediately
- Include social links
- Upload quality logo

### Fee Management

- Claim fees regularly
- Monitor trading activity
- Consider reinvestment

## Common Issues

| Issue | Resolution |
|-------|------------|
| Rate limit reached | Wait 24 hours or upgrade |
| Name taken | Choose different name |
| Symbol exists | Use unique symbol |
| Image upload failed | Check format/size |

## Supported Chains

### Base (Primary)

- Full Clanker support
- Legacy fee claims
- Most liquidity

### Unichain

- Token deployment
- Fee management
- Growing ecosystem

## Tips

1. **Research first** - Check if name/symbol exists
2. **Quality branding** - Good logo matters
3. **Complete metadata** - Fill all fields
4. **Claim regularly** - Don't leave fees unclaimed
5. **Engage community** - Build around your token
