# Nevermind

**Autonomous cross-chain yield infrastructure for DeFi.**

AI-powered yield analysis, verifiably executed on-chain through Chainlink CRE.

[Launch App](https://nevermind.fi) · [Documentation](https://github.com/nevermind-fi)

---

## How it works

```text
Deposit USDC ──> CRE Workflow triggers ──> AI analyzes yields across chains
                                                      │
              Auto-rebalance executes <── DON verifies recommendation
```

1. **Deposit & Configure** — Connect wallet, deposit USDC, set risk tolerance
2. **AI Yield Analysis** — CRE workflow fetches live yield data from DeFi Llama, feeds it to GPT-4o-mini via OpenRouter, returns allocation recommendation — all verified by Chainlink DON
3. **Auto-Rebalance** — Executes on-chain rebalancing when yield delta exceeds threshold

## Architecture

| Layer               | Stack                                                                 |
| ------------------- | --------------------------------------------------------------------- |
| **Smart Contracts** | Solidity + Foundry, deployed on Tenderly Virtual TestNet              |
| **CRE Workflow**    | Chainlink CRE SDK — cron trigger, HTTP fetch, LLM analysis, EVM write |
| **Frontend**        | Next.js 16 + Tailwind + thirdweb (email/social wallet login)          |

## Repositories

| Repo                                                           | Description                                                            |
| -------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [`frontend`](https://github.com/nevermind-fi/frontend)         | Dashboard UI — deposit, yield charts, agent activity log               |
| [`contracts`](https://github.com/nevermind-fi/contracts)       | YieldPilotVault.sol — vault logic with CRE-authorized rebalancing      |
| [`cre-workflow`](https://github.com/nevermind-fi/cre-workflow) | Chainlink CRE workflow — DeFi Llama + GPT-4o-mini + on-chain execution |

## Built with

Chainlink CRE · Ethereum · Arbitrum · OpenRouter · Tenderly · thirdweb

---

_Built for the [Chainlink Hackathon 2026](https://chain.link/hackathon) — CRE & AI track_
