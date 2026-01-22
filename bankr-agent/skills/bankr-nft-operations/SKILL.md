---
name: Bankr NFT Operations
description: This skill should be used when the user asks to "buy NFT", "purchase NFT", "OpenSea", "NFT collection", "view my NFTs", "NFT holdings", "mint NFT", "NFT listings", or any NFT-related operation. Provides guidance on browsing, purchasing, and managing NFTs.
version: 1.0.0
---

# Bankr NFT Operations

Browse, purchase, and manage NFTs across chains.

## Overview

Bankr supports NFT operations through OpenSea integration:
- Browse NFT collections
- Find best listings
- Purchase NFTs
- View NFT holdings

**Supported Chains**: Base, Ethereum, Polygon, and other EVM chains

## Operations

### Browse NFTs

Search for NFT collections:

```
"Find NFTs from the Bored Ape collection"
"Search for CryptoPunks on OpenSea"
"Show me trending NFT collections"
```

### View Listings

Find the best deals in a collection:

```
"What's the floor price for Pudgy Penguins?"
"Show cheapest NFTs in Azuki collection"
"Find listings under 0.1 ETH in Doodles"
```

### Buy NFTs

Purchase an NFT:

```
"Buy the cheapest Bored Ape"
"Purchase this NFT: [OpenSea URL]"
"Buy floor NFT from CryptoPunks"
```

### View Holdings

Check your NFT portfolio:

```
"Show my NFTs"
"What NFTs do I own?"
"List my NFT holdings on Base"
```

## Purchase Methods

### By Collection Name

```
"Buy the floor NFT from Pudgy Penguins"
"Purchase cheapest NFT in Azuki collection"
```

### By OpenSea URL

```
"Buy this NFT: https://opensea.io/assets/ethereum/0x.../1234"
"Purchase the NFT at [URL]"
```

### By Specific Criteria

```
"Buy a Bored Ape under 30 ETH"
"Purchase the cheapest blue Pudgy Penguin"
```

## Prompt Examples

### Browsing

```
"Search for NFT collections on Base"
"Find popular NFT projects"
"Show me NFTs similar to CryptoPunks"
"What are trending NFT collections?"
```

### Listings

```
"What's the floor price for Bored Apes?"
"Show the 5 cheapest NFTs in Azuki"
"Find NFT listings under 0.5 ETH"
"What are the best deals in Pudgy Penguins?"
```

### Buying

```
"Buy the cheapest NFT from Doodles"
"Purchase this OpenSea listing: [URL]"
"Buy a Bored Ape"
"Get me the floor NFT from CryptoPunks"
```

### Holdings

```
"Show my NFT collection"
"What NFTs do I own on Ethereum?"
"List my NFT holdings"
"Show NFTs in my wallet"
```

## Supported Operations

- **Buy** - Purchase NFTs from marketplace listings
- **Transfer** - Send NFTs to another wallet
- **Mint** - Mint from supported platforms (Manifold, SeaDrop)
- **View** - Check your NFT holdings

## Collection Resolution

Bankr resolves collection names to OpenSea slugs:

| Input | Resolved |
|-------|----------|
| "Bored Apes" | boredapeyachtclub |
| "BAYC" | boredapeyachtclub |
| "Pudgy Penguins" | pudgypenguins |
| "CryptoPunks" | cryptopunks |

Supports common names, abbreviations, and variations.

## Chain Considerations

### Ethereum
- Most valuable collections
- Higher gas fees
- Primary OpenSea marketplace

### Base
- Growing NFT ecosystem
- Very low gas fees
- Good for newer collections

### Polygon
- Low gas costs
- Some popular collections
- Gaming NFTs

## Common Issues

| Issue | Resolution |
|-------|------------|
| Collection not found | Try alternative names |
| NFT already sold | Try another listing |
| Insufficient funds | Check balance |
| High gas | Wait or try L2 |

## Safety Tips

- **Verify collection** - Check official links
- **Check floor price** - Avoid overpaying
- **Review before buying** - Confirm the NFT
- **Beware of fakes** - Look for verified collections
- **Gas considerations** - Factor in transaction costs
