---
description: Send a query to Bankr for crypto, trading, or Polymarket operations
argument-hint: [query]
---

Send the following query to the Bankr API: $ARGUMENTS

Follow the `bankr-job-workflow` skill for execution:
1. Submit the query using `bankr_agent_submit_prompt`
2. Poll for status using `bankr_agent_get_job_status` every 2 seconds
3. Report status updates to the user as they come in
4. When complete, share the final response

For context on specific operations, load the appropriate capability skill:
- Trading: `bankr-token-trading`
- Transfers: `bankr-transfers`
- Polymarket: `bankr-polymarket`
- Leverage: `bankr-leverage-trading`
- NFTs: `bankr-nft-operations`
- Portfolio: `bankr-portfolio`
- Research: `bankr-market-research`
- Automation: `bankr-automation`
- Token deployment: `bankr-token-deployment`

If errors occur, consult the `bankr-error-handling` skill.

If no query is provided, ask the user what they'd like to do with Bankr (crypto trading, price checks, market analysis, Polymarket predictions, etc.).
