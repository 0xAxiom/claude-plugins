---
description: Scaffold a new project that uses the Bankr Agent API
argument-hint: [project-type]
allowed-tools: Read, Write, Bash, AskUserQuestion, Glob, Grep
---

# Bankr Project Scaffold

Create a complete TypeScript/Node.js project scaffold for building on the Bankr Agent API.

## Process

1. **Determine project type** - If `$ARGUMENTS` specifies a type, use it. Otherwise, ask the user.

   Load the `bankr-project-templates` skill for available types:
   - **bot** - Automated trading bot, price monitor, alert system, scheduled task
   - **web-service** - HTTP API that wraps or extends Bankr functionality
   - **dashboard** - Web UI for portfolio tracking, market analysis, monitoring
   - **cli** - Command-line tool for Bankr operations

2. **Ask for project details**:
   - Project name (kebab-case, e.g., `my-trading-bot`)
   - Brief description of what it will do
   - Any specific Bankr operations it will use (trading, prices, polymarket, defi)
   - Package manager preference: **bun** (recommended - faster) or **npm**

3. **Create project structure**:
   - Load `bankr-project-templates` skill for the directory structure
   - Load `bankr-client-patterns` skill for common files and client code
   - Create all directories using `mkdir -p`
   - Write each file using the Write tool
   - Customize based on user's description

4. **Explain next steps**:
   - How to get a Bankr API key (https://bankr.bot/api)
   - How to run the project (use selected package manager: `bun install && bun dev` or `npm install && npm run dev`)
   - What to customize first based on their use case

## Skills to Load

**Core Skills (always load):**
- `bankr-project-templates` - Directory structures for each project type
- `bankr-client-patterns` - bankr-client.ts and common files (package.json, tsconfig.json, .env.example, .gitignore)
- `bankr-api-basics` - API documentation for reference

**Capability Skills (load based on project purpose):**
- `bankr-token-trading` - For trading bots: prompt templates, swap response handling
- `bankr-transfers` - For payment bots: recipient resolution, transfer patterns
- `bankr-polymarket` - For prediction market bots: betting API, position tracking
- `bankr-leverage-trading` - For leverage bots: Avantis patterns, position management
- `bankr-nft-operations` - For NFT bots: OpenSea browsing, purchase automation
- `bankr-portfolio` - For dashboards: balance aggregation, portfolio tracking
- `bankr-market-research` - For price bots: monitoring patterns, alert systems
- `bankr-automation` - For order bots: limit orders, DCA, TWAP implementation
- `bankr-token-deployment` - For token launchers: Clanker integration, fee claiming

## Implementation Notes

- Create all directories first using `mkdir -p`
- Write each file using the Write tool
- Include helpful comments in generated code
- Generate a README.md with setup instructions and usage examples
