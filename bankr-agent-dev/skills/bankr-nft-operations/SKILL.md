---
name: Bankr Dev - NFT Operations
description: This skill should be used when building an NFT sniping bot, implementing floor price monitoring, creating a collection sweeper, or automating NFT purchases via OpenSea. Covers TypeScript prompt templates like `nftPrompts.buyFloor()`, floor tracking with trend detection, and NftBot class with watchlist management. Triggered by "NFT bot code", "floor sweeper TypeScript", "OpenSea automation", "NFT sniping implementation".
version: 1.0.0
---

# NFT Operations - Developer Guide

Build bots and applications that interact with NFTs via the Bankr API.

## Overview

NFT operations through Bankr (via OpenSea) support:
- Browsing NFT collections
- Finding best listings and floor prices
- Purchasing NFTs
- Viewing NFT holdings

**Supported Chains:** Base, Ethereum, Polygon, and other EVM chains

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for NFT operations
const nftPrompts = {
  // Browse collections
  searchCollection: (collection: string) => `Find NFTs from the ${collection} collection`,
  trendingCollections: () => "Show me trending NFT collections",
  searchByChain: (chain: string) => `Search for NFT collections on ${chain}`,

  // Check listings
  floorPrice: (collection: string) => `What's the floor price for ${collection}?`,
  cheapestNfts: (collection: string, count: number = 5) =>
    `Show the ${count} cheapest NFTs in ${collection}`,
  listingsUnder: (collection: string, price: string) =>
    `Find NFT listings under ${price} in ${collection}`,

  // Buy NFTs
  buyFloor: (collection: string) => `Buy the floor NFT from ${collection}`,
  buyCheapest: (collection: string) => `Buy the cheapest NFT from ${collection}`,
  buyByUrl: (url: string) => `Buy this NFT: ${url}`,
  buyWithBudget: (collection: string, maxPrice: string) =>
    `Buy a ${collection} NFT under ${maxPrice}`,

  // View holdings
  viewHoldings: () => "Show my NFTs",
  viewHoldingsOnChain: (chain: string) => `Show my NFTs on ${chain}`,
  viewCollection: (collection: string) => `Show my NFTs from ${collection}`,
};

// Popular collections mapping
const COLLECTION_ALIASES: Record<string, string> = {
  "bored apes": "boredapeyachtclub",
  "bayc": "boredapeyachtclub",
  "pudgy penguins": "pudgypenguins",
  "cryptopunks": "cryptopunks",
  "azuki": "azuki",
  "doodles": "doodles-official",
  "milady": "milady",
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface NftListing {
  tokenId: string;
  collection: string;
  price: number;
  currency: string;
  url?: string;
}

interface NftHolding {
  tokenId: string;
  collection: string;
  estimatedValue: number;
  chain: string;
}

// Parse listings from response
function parseListings(response: string): NftListing[] {
  const listings: NftListing[] = [];
  // Example: "1. #4521 - 8.5 ETH"
  const lines = response.split("\n").filter((l) => l.match(/^\d+\./));

  for (const line of lines) {
    const match = line.match(/^(\d+)\.\s+#?(\d+)\s+-\s+([0-9.]+)\s+(\w+)/);
    if (match) {
      listings.push({
        tokenId: match[2],
        collection: "", // From context
        price: parseFloat(match[3]),
        currency: match[4],
      });
    }
  }
  return listings;
}

// Parse floor price from response
function parseFloorPrice(response: string): { price: number; currency: string } | null {
  // Example: "Pudgy Penguins floor price: 8.5 ETH"
  const match = response.match(/floor\s*price:\s*([0-9.]+)\s*(\w+)/i);
  if (match) {
    return {
      price: parseFloat(match[1]),
      currency: match[2],
    };
  }
  return null;
}

// Parse holdings from response
function parseHoldings(response: string): NftHolding[] {
  const holdings: NftHolding[] = [];
  // Example: "- Pudgy Penguin #4521 (8.5 ETH value)"
  const lines = response.split("\n").filter((l) => l.startsWith("-"));

  for (const line of lines) {
    const match = line.match(/-\s+(.+?)\s+#(\d+)\s+\(([0-9.]+)\s+(\w+)\s+value\)/);
    if (match) {
      holdings.push({
        collection: match[1],
        tokenId: match[2],
        estimatedValue: parseFloat(match[3]),
        chain: "ethereum", // Default, may vary
      });
    }
  }
  return holdings;
}

async function handleNftResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("NFT operation successful");
    console.log(result.response);

    // Handle transactions
    if (result.transactions) {
      for (const tx of result.transactions) {
        const data = tx.metadata.__ORIGINAL_TX_DATA__;
        if (data) {
          console.log(`${data.humanReadableMessage}`);
          console.log(`Price: ${data.inputTokenAmount} ${data.inputTokenTicker}`);
        }
      }
    }
  } else if (result.status === "failed") {
    console.error("NFT operation failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// NFT-specific types for your bot
interface NftPurchaseConfig {
  collection: string;
  maxPrice: number; // ETH
  autoApprove: boolean;
}

interface FloorTracker {
  collection: string;
  targetFloor: number;
  currentFloor?: number;
  lastChecked?: Date;
}

interface PurchaseResult {
  success: boolean;
  tokenId: string;
  collection: string;
  price: number;
  currency: string;
  txHash?: string;
}
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Floor Sweeper | Buy floor when cheap | `Buy the floor NFT from Pudgy Penguins` |
| Sniper | Watch for underpriced | `Find listings under 5 ETH in Azuki` |
| Portfolio Tracker | Monitor holdings | `Show my NFTs` |
| Deal Finder | Find bargains | `Show cheapest NFTs in BAYC` |

## Code Example: NFT Monitoring Bot

```typescript
import { execute } from "./bankr-client";

interface CollectionWatch {
  collection: string;
  targetFloor: number; // Buy if floor drops below this
  maxBudget: number; // Max to spend
  enabled: boolean;
}

class NftBot {
  private watchlist: CollectionWatch[] = [];

  async getFloorPrice(collection: string): Promise<number | null> {
    const result = await execute(nftPrompts.floorPrice(collection));
    if (result.status !== "completed" || !result.response) return null;
    return parseFloorPrice(result.response)?.price || null;
  }

  async getCheapestListings(collection: string, count = 5): Promise<NftListing[]> {
    const result = await execute(nftPrompts.cheapestNfts(collection, count));
    return result.status === "completed" && result.response ? parseListings(result.response) : [];
  }

  async buyFloor(collection: string): Promise<PurchaseResult | null> {
    const result = await execute(nftPrompts.buyFloor(collection), (msg) => console.log("  >", msg));
    if (result.status !== "completed") return null;

    const txData = result.transactions?.[0]?.metadata.__ORIGINAL_TX_DATA__;
    if (!txData) return null;

    return {
      success: true,
      tokenId: "",
      collection,
      price: parseFloat(txData.inputTokenAmount),
      currency: txData.inputTokenTicker,
    };
  }

  async buyByUrl(url: string): Promise<PurchaseResult | null> {
    const result = await execute(nftPrompts.buyByUrl(url), (msg) => console.log("  >", msg));
    if (result.status !== "completed") return null;
    return { success: true, tokenId: "", collection: "", price: 0, currency: "ETH" };
  }

  async getHoldings(): Promise<NftHolding[]> {
    const result = await execute(nftPrompts.viewHoldings());
    return result.status === "completed" && result.response ? parseHoldings(result.response) : [];
  }

  addToWatchlist(watch: CollectionWatch): void {
    this.watchlist.push(watch);
  }

  async checkWatchlist(): Promise<void> {
    for (const watch of this.watchlist) {
      if (!watch.enabled) continue;

      const floor = await this.getFloorPrice(watch.collection);
      if (floor === null || floor > watch.targetFloor || floor > watch.maxBudget) continue;

      const purchase = await this.buyFloor(watch.collection);
      if (purchase?.success) {
        watch.enabled = false;
      }
    }
  }
}

// Usage
const bot = new NftBot();

// Check floor prices
const floor = await bot.getFloorPrice("Pudgy Penguins");
console.log("Pudgy Penguins floor:", floor);

// Get cheapest listings
const listings = await bot.getCheapestListings("Azuki", 5);
console.log("Cheapest Azuki:", listings);

// Add to watchlist
bot.addToWatchlist({
  collection: "Pudgy Penguins",
  targetFloor: 7.5,
  maxBudget: 8,
  enabled: true,
});

// Check watchlist periodically
setInterval(() => bot.checkWatchlist(), 60000);

// Get holdings
const holdings = await bot.getHoldings();
console.log("My NFTs:", holdings);
```

## Floor Tracking

```typescript
interface FloorHistory {
  timestamp: Date;
  floor: number;
}

class FloorTracker {
  private history: Map<string, FloorHistory[]> = new Map();

  async trackFloor(collection: string, bot: NftBot): Promise<void> {
    const floor = await bot.getFloorPrice(collection);
    if (floor === null) return;

    const history = this.history.get(collection) || [];
    history.push({
      timestamp: new Date(),
      floor,
    });

    // Keep last 100 data points
    if (history.length > 100) {
      history.shift();
    }

    this.history.set(collection, history);
  }

  getFloorTrend(collection: string): "up" | "down" | "stable" | null {
    const history = this.history.get(collection);
    if (!history || history.length < 2) return null;

    const recent = history.slice(-5);
    const oldAvg = recent.slice(0, 2).reduce((a, b) => a + b.floor, 0) / 2;
    const newAvg = recent.slice(-2).reduce((a, b) => a + b.floor, 0) / 2;

    const change = (newAvg - oldAvg) / oldAvg;
    if (change > 0.05) return "up";
    if (change < -0.05) return "down";
    return "stable";
  }

  getLowestFloor(collection: string): number | null {
    const history = this.history.get(collection);
    if (!history || history.length === 0) return null;
    return Math.min(...history.map((h) => h.floor));
  }
}
```

## Error Handling

Common errors and how to handle them:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Collection not found | Invalid name | Try aliases |
| NFT already sold | Race condition | Try next listing |
| Insufficient funds | Not enough ETH | Check balance first |
| High gas | Network congestion | Wait or use L2 |

```typescript
async function safePurchase(
  collection: string,
  maxPrice: number,
  bot: NftBot
): Promise<PurchaseResult | null> {
  // Check floor first
  const floor = await bot.getFloorPrice(collection);
  if (floor === null) {
    console.error("Could not get floor price");
    return null;
  }

  if (floor > maxPrice) {
    console.error(`Floor (${floor}) exceeds max price (${maxPrice})`);
    return null;
  }

  // Attempt purchase
  const result = await bot.buyFloor(collection);

  if (!result?.success) {
    // Try next cheapest if floor sold
    const listings = await bot.getCheapestListings(collection, 3);
    const affordable = listings.filter((l) => l.price <= maxPrice);

    if (affordable.length > 0) {
      // Could implement buying by token ID here
      console.log("Alternative listings found:", affordable);
    }
  }

  return result;
}
```

## Chain Selection for NFTs

```typescript
const NFT_CHAINS = {
  ethereum: { majorCollections: true, highGas: true },
  base: { majorCollections: false, highGas: false },
  polygon: { majorCollections: false, highGas: false },
};

function recommendChain(budget: number): string {
  return budget > 1 ? "Ethereum" : "Base";
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-portfolio` - Check ETH balance before purchases
