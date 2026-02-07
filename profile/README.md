# Nevermind

**Autonomous cross-chain yield optimizer powered by Chainlink CRE and AI.**

Deposit USDC. Set your risk profile. An AI agent running inside a Chainlink CRE workflow finds the best yields across protocols and rebalances your position automatically — all verifiably executed by a Decentralized Oracle Network.

---

## How It Works

1. **Deposit & Configure** — Connect wallet via thirdweb (email/social login). Approve USDC, deposit into vault. Set risk tolerance (Conservative / Balanced / Aggressive).

2. **AI Yield Analysis** — Click "Analyze Yields" to trigger the CRE workflow pipeline:
   - Fetches live USDC yield rates from DeFi Llama (Aave V3 + Compound V3 on Ethereum, Arbitrum, Base, Optimism)
   - Reads vault state on-chain (total deposits, current allocations)
   - Sends data to GPT-4o-mini via OpenRouter with the user's risk profile
   - Returns recommended allocation with per-protocol reasoning and risk score

3. **Auto-Rebalance** — Click "Apply Rebalance" to execute on-chain. The `executeCRERebalance()` function records the new allocation in the vault. On Tenderly VNet, this uses impersonated transactions from the CRE-authorized address.

The AI decision happens _inside_ the CRE workflow, verified by Chainlink's DON consensus — not a centralized backend call.

## Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│  Frontend (Next.js 16 + thirdweb)                               │
│  - Connect wallet (email/social via thirdweb)                   │
│  - Deposit/withdraw USDC                                        │
│  - Set risk profile (Conservative / Balanced / Aggressive)      │
│  - View AI recommendations + apply rebalance                    │
└──────────────┬──────────────────────────────┬───────────────────┘
               │                              │
     thirdweb SDK                    /api/analyze + /api/rebalance
     (contract reads)                (Next.js API routes)
               │                              │
┌──────────────▼──────────────┐  ┌────────────▼────────────────────┐
│  NevermindVault.sol        │  │  CRE Workflow (DON)              │
│  Tenderly VNet (chain 73571)│  │                                  │
│                             │  │  1. Fetch yields (DeFi Llama)    │
│  - deposit(amount)          │  │     HTTPClient + consensus       │
│  - withdraw(amount)         │  │  2. Read vault state (EVM)       │
│  - setRiskProfile(level)    │  │     EVMClient.callContract       │
│  - executeCRERebalance()    │◄─┤  3. Analyze with LLM             │
│    (onlyCRE modifier)       │  │     GPT-4o-mini via OpenRouter   │
│  - getAllocations()         │  │  4. Execute rebalance             │
│  - getBalance(user)         │  │     DON-signed writeReport       │
└─────────────────────────────┘  └─────────────────────────────────┘
```

### How CRE Integrates

The [Chainlink CRE](https://docs.chain.link/cre) (Compute Runtime Environment) runs the yield optimization workflow on a Decentralized Oracle Network (DON). Each of the 4 steps executes independently on multiple DON nodes and reaches **consensus** before proceeding:

1. **HTTPClient** fetches yield data from DeFi Llama — verified by `consensusIdenticalAggregation`
2. **EVMClient** reads `totalDeposits` from the vault — verified at `LAST_FINALIZED_BLOCK_NUMBER`
3. **HTTPClient** sends yield data + risk profile to GPT-4o-mini — LLM response verified by consensus
4. **EVMClient** submits a DON-signed report to `executeCRERebalance()` — trustless execution

No single server controls the AI decisions or fund movements. The DON provides verifiable, decentralized orchestration.

### How Tenderly Virtual TestNet Integrates

The smart contract is deployed on a [Tenderly Virtual TestNet](https://docs.tenderly.co/virtual-testnets) — an Ethereum mainnet fork (chain ID 73571):

- **Real USDC contract** at its mainnet address (no mock tokens)
- **Impersonated transactions** — send txs from any address without a private key (used for CRE-authorized rebalance)
- **Built-in block explorer** — view contracts, transactions, and state changes
- **Instant confirmations** — no waiting for blocks

### How thirdweb Integrates

[thirdweb v5 SDK](https://portal.thirdweb.com/) handles all user-facing wallet interactions:

- **ConnectButton** — email, social login, or traditional wallet connection
- **useReadContract** — reads vault balance, risk profile, allocations
- **useSendTransaction** — USDC approval + vault deposit/withdraw
- CRE handles the autonomous backend logic; thirdweb handles user onboarding

## Stack

| Layer        | Tech                                             |
| ------------ | ------------------------------------------------ |
| Contracts    | Solidity 0.8.20, Foundry                         |
| CRE Workflow | TypeScript, @chainlink/cre-sdk, viem             |
| Frontend     | Next.js 16, React 19, Tailwind CSS v4            |
| Wallet       | thirdweb v5 (email/social login)                 |
| AI           | GPT-4o-mini via OpenRouter + CRE HTTP capability |
| Data         | DeFi Llama /pools API                            |
| TestNet      | Tenderly Virtual TestNet (Ethereum mainnet fork) |

## Deployed

| Contract       | Address                                      | Network                     |
| -------------- | -------------------------------------------- | --------------------------- |
| NevermindVault | `0x2A8e741e626F795784f1926052DD61Af14A01638` | Tenderly VNet (chain 73571) |
| USDC           | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` | Mainnet fork                |

**Tenderly VNet RPC:** `https://virtual.rpc.tenderly.co/BESTOFRENTO/project/public/nevermind`

**Tenderly Explorer:** [dashboard.tenderly.co/explorer/vnet/47a3406e-9b8c-4386-a0f5-55c140d9cfc9](https://dashboard.tenderly.co/explorer/vnet/47a3406e-9b8c-4386-a0f5-55c140d9cfc9)

## Repositories

| Repo                                                           | Description                                                            |
| -------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [`frontend`](https://github.com/nevermind-fi/frontend)         | Dashboard UI — deposit, yield charts, agent activity log               |
| [`contracts`](https://github.com/nevermind-fi/contracts)       | NevermindVault.sol — vault logic with CRE-authorized rebalancing       |
| [`cre-workflow`](https://github.com/nevermind-fi/cre-workflow) | Chainlink CRE workflow — DeFi Llama + GPT-4o-mini + on-chain execution |

## Prize Tracks

### CRE & AI ($17K / $10.5K / $6.5K)

CRE orchestrates the full pipeline: blockchain reads (EVMClient) + external API (DeFi Llama via HTTPClient) + LLM analysis (GPT-4o-mini via HTTPClient) + on-chain execution (DON-signed writeReport). Each step runs with consensus across DON nodes.

### Tenderly Virtual TestNets ($5K / $2.5K / $1.75K)

Smart contracts deployed on Tenderly VNet (Ethereum mainnet fork, chain 73571). All transactions visible in Tenderly Explorer with full traces and state changes.

### thirdweb x CRE (subscription prizes)

thirdweb handles user onboarding (ConnectButton with email/social login) and all contract interactions. CRE handles autonomous yield optimization. Clean separation: thirdweb = user layer, CRE = agent layer.

---

Built for the [Chainlink Convergence Hackathon 2026](https://chain.link/hackathon)

Chainlink CRE | Tenderly | thirdweb
