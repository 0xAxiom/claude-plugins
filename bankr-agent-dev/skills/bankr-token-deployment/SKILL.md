---
name: Bankr Token Deployment (Developer Guide)
description: This skill should be used when building a token deployment system, implementing Clanker integration, creating fee auto-claiming, or automating token metadata updates. Covers TypeScript prompt templates like `tokenPrompts.deployWithSocials()`, deployment result parsing, batch deployment with rate limiting, and FeeAutoClaimer class. Triggered by "token deployment bot code", "Clanker integration TypeScript", "fee claiming automation", "batch token deployment".
version: 1.0.0
---

# Token Deployment - Developer Guide

Build bots and applications that deploy and manage tokens via Clanker and the Bankr API.

## Overview

Token deployment through Bankr (via Clanker) supports:
- Deploy new ERC20 tokens
- Update token metadata
- Update token images
- Claim trading fees
- Manage reward recipients

**Supported Chains:** Base (primary), Unichain (secondary)

> **Note:** All types shown in this skill are for reference. Import canonical types from `bankr-client.ts` (see `bankr-client-patterns` skill). Response parsing functions are examples based on typical formats - test against real API responses and add fallback handling for production use.

## Prompt Templates

```typescript
// Prompt builders for token deployment operations
const tokenPrompts = {
  // Deploy token
  deploy: (name: string, symbol: string) =>
    `Deploy a token called ${name} with symbol ${symbol}`,

  deployWithDescription: (name: string, symbol: string, description: string) =>
    `Deploy a token called ${name} (${symbol}) with description: ${description}`,

  deployWithSocials: (
    name: string,
    symbol: string,
    options: { website?: string; twitter?: string; telegram?: string }
  ) => {
    let prompt = `Deploy token ${name} (${symbol})`;
    if (options.website) prompt += ` with website ${options.website}`;
    if (options.twitter) prompt += ` and Twitter @${options.twitter}`;
    if (options.telegram) prompt += ` and Telegram @${options.telegram}`;
    return prompt;
  },

  // Fee management
  checkFees: () => "Check my Clanker fees",
  claimFees: (token?: string) =>
    token ? `Claim fees for my token ${token}` : "Claim all my Clanker fees",
  claimLegacyFees: () => "Claim legacy Clanker fees",

  // Metadata updates
  updateDescription: (token: string, description: string) =>
    `Update description for ${token}: ${description}`,
  updateSocials: (token: string, socials: { twitter?: string; website?: string }) => {
    let prompt = `Update ${token}`;
    if (socials.twitter) prompt += ` Twitter to @${socials.twitter}`;
    if (socials.website) prompt += ` website to ${socials.website}`;
    return prompt;
  },
  updateImage: (token: string, imageUrl: string) =>
    `Update logo for ${token} to ${imageUrl}`,

  // Reward recipient
  updateRewardRecipient: (token: string, address: string) =>
    `Update reward recipient for ${token} to ${address}`,
};
```

## Response Handling

```typescript
import { execute, JobStatusResponse } from "./bankr-client";

interface DeployedToken {
  name: string;
  symbol: string;
  contractAddress: string;
  chain: string;
  deployedAt: Date;
}

interface TokenFees {
  token: string;
  symbol: string;
  unclaimedAmount: number;
  currency: string;
}

// Parse deployment response
function parseDeploymentResult(response: string): DeployedToken | null {
  // Example: "Token deployed successfully!\n\nName: MyToken\nSymbol: MTK\nChain: Base\nContract: 0x1234...abcd"
  const nameMatch = response.match(/Name:\s*(.+)/);
  const symbolMatch = response.match(/Symbol:\s*(\w+)/);
  const chainMatch = response.match(/Chain:\s*(\w+)/);
  const contractMatch = response.match(/Contract:\s*(0x[a-fA-F0-9]+)/);

  if (!contractMatch) return null;

  return {
    name: nameMatch?.[1] || "",
    symbol: symbolMatch?.[1] || "",
    contractAddress: contractMatch[1],
    chain: chainMatch?.[1] || "Base",
    deployedAt: new Date(),
  };
}

// Parse fees response
function parseFees(response: string): TokenFees[] {
  const fees: TokenFees[] = [];
  // Example: "Your unclaimed Clanker fees:\n\nMyToken (MTK): 0.5 ETH\nOtherToken (OTK): 0.1 ETH"
  const lines = response.split("\n").filter((l) => l.includes(":") && l.includes("ETH"));

  for (const line of lines) {
    const match = line.match(/(.+?)\s*\((\w+)\):\s*([0-9.]+)\s*(\w+)/);
    if (match) {
      fees.push({
        token: match[1].trim(),
        symbol: match[2],
        unclaimedAmount: parseFloat(match[3]),
        currency: match[4],
      });
    }
  }

  return fees;
}

// Parse claim response
function parseClaimResult(response: string): { amount: number; currency: string } | null {
  // Example: "Fees claimed successfully!\n\nToken: MyToken (MTK)\nAmount: 0.5 ETH"
  const amountMatch = response.match(/Amount:\s*([0-9.]+)\s*(\w+)/);
  if (amountMatch) {
    return {
      amount: parseFloat(amountMatch[1]),
      currency: amountMatch[2],
    };
  }
  return null;
}

async function handleDeploymentResponse(result: JobStatusResponse) {
  if (result.status === "completed") {
    console.log("Token operation successful");
    console.log(result.response);

    const deployed = parseDeploymentResult(result.response || "");
    if (deployed) {
      console.log(`Token deployed: ${deployed.symbol} at ${deployed.contractAddress}`);
    }
  } else if (result.status === "failed") {
    console.error("Token operation failed:", result.error);
  }
}
```

## TypeScript Types

```typescript
// Token deployment specific types
interface TokenConfig {
  name: string;
  symbol: string;
  description?: string;
  imageUrl?: string;
  website?: string;
  twitter?: string;
  telegram?: string;
  discord?: string;
}

interface DeploymentResult {
  success: boolean;
  token?: DeployedToken;
  error?: string;
}

interface FeeClaimResult {
  success: boolean;
  amount: number;
  currency: string;
  txHash?: string;
}

// Rate limits
const RATE_LIMITS = {
  standard: 1, // tokens per day
  bankrClub: 10, // tokens per day
};
```

## Bot Use Cases

| Use Case | Description | Example Prompt |
|----------|-------------|----------------|
| Token Launcher | Deploy new tokens | `Deploy token MyToken (MTK)` |
| Fee Claimer | Auto-claim fees | `Claim all my Clanker fees` |
| Metadata Manager | Update token info | `Update description for MTK` |
| Batch Deployer | Launch multiple tokens | Multiple deploy calls |

## Code Example: Token Manager

```typescript
import { execute } from "./bankr-client";

class TokenManager {
  private deployedTokens: DeployedToken[] = [];

  async deployToken(config: TokenConfig): Promise<DeploymentResult> {
    let prompt: string;
    if (config.website || config.twitter || config.telegram) {
      prompt = tokenPrompts.deployWithSocials(config.name, config.symbol, {
        website: config.website,
        twitter: config.twitter,
        telegram: config.telegram,
      });
    } else if (config.description) {
      prompt = tokenPrompts.deployWithDescription(config.name, config.symbol, config.description);
    } else {
      prompt = tokenPrompts.deploy(config.name, config.symbol);
    }

    const result = await execute(prompt, (msg) => console.log("  >", msg));
    if (result.status !== "completed") {
      return { success: false, error: result.error || "Failed to deploy token" };
    }

    const token = parseDeploymentResult(result.response || "");
    if (!token) return { success: false, error: "Failed to parse deployment response" };

    this.deployedTokens.push(token);
    return { success: true, token };
  }

  async checkFees(): Promise<TokenFees[]> {
    const result = await execute(tokenPrompts.checkFees());
    return result.status === "completed" && result.response ? parseFees(result.response) : [];
  }

  async claimFees(token?: string): Promise<FeeClaimResult> {
    const result = await execute(tokenPrompts.claimFees(token));
    if (result.status !== "completed") return { success: false, amount: 0, currency: "ETH" };

    const claimed = parseClaimResult(result.response || "");
    return claimed
      ? { success: true, amount: claimed.amount, currency: claimed.currency }
      : { success: false, amount: 0, currency: "ETH" };
  }

  async claimAllFees(): Promise<FeeClaimResult[]> {
    const fees = await this.checkFees();
    const results: FeeClaimResult[] = [];

    for (const fee of fees) {
      if (fee.unclaimedAmount > 0) {
        results.push(await this.claimFees(fee.symbol));
        await new Promise((r) => setTimeout(r, 3000));
      }
    }

    return results;
  }

  async updateMetadata(symbol: string, updates: { description?: string; twitter?: string; website?: string }): Promise<boolean> {
    if (updates.description) {
      const result = await execute(tokenPrompts.updateDescription(symbol, updates.description));
      if (result.status !== "completed") return false;
    }

    if (updates.twitter || updates.website) {
      const result = await execute(tokenPrompts.updateSocials(symbol, { twitter: updates.twitter, website: updates.website }));
      if (result.status !== "completed") return false;
    }

    return true;
  }

  async updateImage(symbol: string, imageUrl: string): Promise<boolean> {
    return (await execute(tokenPrompts.updateImage(symbol, imageUrl))).status === "completed";
  }

  getDeployedTokens(): DeployedToken[] {
    return this.deployedTokens;
  }
}

// Usage
const manager = new TokenManager();

// Deploy a token
const deployResult = await manager.deployToken({
  name: "MyAwesomeToken",
  symbol: "MAT",
  description: "A community token for awesome people",
  website: "https://mytoken.xyz",
  twitter: "mytoken",
});

console.log("Deployment result:", deployResult);

// Check fees
const fees = await manager.checkFees();
console.log("Unclaimed fees:", fees);

// Claim all fees
const claimResults = await manager.claimAllFees();
console.log("Claimed fees:", claimResults);

// Update metadata
await manager.updateMetadata("MAT", {
  description: "Updated description for the token",
  twitter: "newhandle",
});
```

## Batch Token Deployment

```typescript
interface BatchDeployConfig {
  tokens: TokenConfig[];
  delayBetween: number; // milliseconds
  maxPerDay: number;
}

class BatchDeployer {
  private deployed: DeployedToken[] = [];
  private deployedToday = 0;

  async deployBatch(
    configs: TokenConfig[],
    manager: TokenManager,
    options: { delayMs: number; maxPerDay: number }
  ): Promise<DeploymentResult[]> {
    const results: DeploymentResult[] = [];

    for (const config of configs) {
      if (this.deployedToday >= options.maxPerDay) break;

      const result = await manager.deployToken(config);
      results.push(result);

      if (result.success && result.token) {
        this.deployed.push(result.token);
        this.deployedToday++;
      }

      await new Promise((r) => setTimeout(r, options.delayMs));
    }

    return results;
  }

  resetDailyCount(): void {
    this.deployedToday = 0;
  }
}

// Example: Batch deploy tokens
const deployer = new BatchDeployer();
const tokens: TokenConfig[] = [
  { name: "Token One", symbol: "ONE" },
  { name: "Token Two", symbol: "TWO" },
  { name: "Token Three", symbol: "THREE" },
];

const batchResults = await deployer.deployBatch(tokens, manager, {
  delayMs: 10000, // 10 seconds between deploys
  maxPerDay: RATE_LIMITS.standard,
});
```

## Fee Auto-Claimer

```typescript
interface FeeClaimerConfig {
  checkInterval: number; // milliseconds
  minClaimAmount: number; // minimum ETH to trigger claim
  autoClaimLegacy: boolean;
}

class FeeAutoClaimer {
  private isRunning = false;
  private intervalId?: NodeJS.Timeout;

  constructor(private manager: TokenManager, private config: FeeClaimerConfig) {}

  start(): void {
    if (this.isRunning) return;
    this.isRunning = true;

    this.intervalId = setInterval(() => this.checkAndClaim(), this.config.checkInterval);
    this.checkAndClaim();
  }

  stop(): void {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = undefined;
    }
    this.isRunning = false;
  }

  private async checkAndClaim(): Promise<void> {
    const fees = await this.manager.checkFees();
    const totalUnclaimed = fees.reduce((sum, f) => sum + f.unclaimedAmount, 0);

    if (totalUnclaimed >= this.config.minClaimAmount) {
      const results = await this.manager.claimAllFees();
      const totalClaimed = results.filter((r) => r.success).reduce((sum, r) => sum + r.amount, 0);
      console.log(`Claimed ${totalClaimed} ETH`);
    }

    if (this.config.autoClaimLegacy) {
      await execute(tokenPrompts.claimLegacyFees());
    }
  }
}

// Usage
const claimer = new FeeAutoClaimer(manager, {
  checkInterval: 3600000, // Check every hour
  minClaimAmount: 0.01, // Claim when > 0.01 ETH accumulated
  autoClaimLegacy: true,
});

claimer.start();
```

## Token Naming Validation

```typescript
interface NamingValidation {
  valid: boolean;
  errors: string[];
  suggestions?: string[];
}

function validateTokenConfig(config: TokenConfig): NamingValidation {
  const errors: string[] = [];
  const suggestions: string[] = [];

  // Check symbol
  if (config.symbol.length < 2) {
    errors.push("Symbol too short (min 2 characters)");
  }
  if (config.symbol.length > 5) {
    errors.push("Symbol too long (max 5 characters)");
  }
  if (!/^[A-Z0-9]+$/.test(config.symbol.toUpperCase())) {
    errors.push("Symbol should be alphanumeric");
  }

  // Check name
  if (config.name.length < 2) {
    errors.push("Name too short");
  }
  if (config.name.length > 32) {
    errors.push("Name too long (max 32 characters)");
  }

  // Check for common issues
  const commonSymbols = ["ETH", "BTC", "USDC", "USDT", "SOL", "BNKR"];
  if (commonSymbols.includes(config.symbol.toUpperCase())) {
    errors.push("Symbol conflicts with major token");
    suggestions.push(`Consider: ${config.symbol}2, x${config.symbol}`);
  }

  return {
    valid: errors.length === 0,
    errors,
    suggestions: suggestions.length > 0 ? suggestions : undefined,
  };
}

// Example usage
const validation = validateTokenConfig({
  name: "My Token",
  symbol: "MTK",
});

if (!validation.valid) {
  console.log("Validation errors:", validation.errors);
  if (validation.suggestions) {
    console.log("Suggestions:", validation.suggestions);
  }
} else {
  console.log("Config is valid, ready to deploy");
}
```

## Error Handling

Common errors and how to handle them:

| Error | Cause | Bot Response |
|-------|-------|--------------|
| Rate limit reached | Daily limit exceeded | Wait 24 hours or upgrade |
| Name taken | Duplicate name | Choose different name |
| Symbol exists | Duplicate symbol | Choose different symbol |
| Image upload failed | Invalid format/size | Validate before upload |

```typescript
async function safeDeployToken(
  config: TokenConfig,
  manager: TokenManager
): Promise<DeploymentResult> {
  // Validate first
  const validation = validateTokenConfig(config);
  if (!validation.valid) {
    return {
      success: false,
      error: `Validation failed: ${validation.errors.join(", ")}`,
    };
  }

  // Attempt deployment
  const result = await manager.deployToken(config);

  if (!result.success) {
    // Handle specific errors
    if (result.error?.includes("rate limit")) {
      return {
        success: false,
        error: "Daily deployment limit reached. Try again in 24 hours.",
      };
    }
    if (result.error?.includes("name taken") || result.error?.includes("symbol exists")) {
      return {
        success: false,
        error: "Name or symbol already exists. Please choose a different one.",
      };
    }
  }

  return result;
}
```

## Related Skills

- `bankr-client-patterns` - Client code and TypeScript types
- `bankr-api-basics` - API fundamentals
- `bankr-token-trading` - Trade deployed tokens
- `bankr-market-research` - Track token performance
