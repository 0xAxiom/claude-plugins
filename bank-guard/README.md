# Bank Guard

> ⚠️ **UNDER DEVELOPMENT** — This plugin is not yet ready for production use. APIs may change. Do not rely on it for real security until v1.0 release.

Security middleware for Bankr agents. Protect your agent from draining its wallet, signing malicious transactions, or leaking keys.

## Features

- **Transaction Firewall** — Spending limits, contract allowlists, simulation before signing
- **Prompt Injection Defense** — Sanitize on-chain data before feeding to LLM
- **Secret Isolation** — Keys never exposed to LLM context
- **Audit Trail** — Every action logged

## Installation

```bash
# Claude Code
claude plugin marketplace add 0xAxiom/claude-plugins
claude plugin install bank-guard@0xaxiom-claude-plugins

# Other tools
bunx skills add 0xAxiom/claude-plugins
```

## Quick Start

```typescript
import { BankGuard } from 'bank-guard';

// Create a guard with strict settings
const guard = BankGuard.strict('https://mainnet.base.org');

// Check transaction before signing
const result = await guard.checkTransaction({
  to: '0x...',
  value: parseEther('0.1'),
  data: '0x...'
});

if (!result.allowed) {
  console.log('Blocked:', result.reason);
}

// Sanitize on-chain data before LLM
const input = await guard.sanitizeInput(tokenMetadata);

// Redact secrets from output
const output = await guard.redactOutput(agentResponse);
```

## Components

| Component | Purpose |
|-----------|---------|
| **TransactionFirewall** | Spending limits, contract allowlists, tx simulation |
| **PromptSanitizer** | Blocks prompt injection from on-chain data |
| **SecretIsolator** | Redacts private keys, seed phrases from output |
| **AuditLogger** | Logs all security events |

## Configuration

```typescript
const guard = new BankGuard({
  maxDailySpendEth: 1,      // Max 1 ETH per day
  maxPerTxSpendEth: 0.1,    // Max 0.1 ETH per transaction
  allowedContracts: [       // Only interact with these
    '0x...',
  ],
  strictMode: true,         // Reject any suspicious input
  rpcUrl: 'https://mainnet.base.org'
});
```

## Why Bank Guard?

Bankr gives agents powerful trading capabilities. But power without safety is dangerous:

- ❌ Bad prompt → agent drains wallet
- ❌ Malicious token metadata → prompt injection
- ❌ No audit trail → can't debug or prove intent

Bank Guard adds the missing security layer.

## Links

- [AgentGuard (Solana)](https://github.com/0xAxiom/agentguard) — Sister project for Solana agents
- [Bankr](https://bankr.bot) — Multi-chain trading infrastructure
- [Builder](https://twitter.com/AxiomBot) — Built by Axiom

## License

MIT
