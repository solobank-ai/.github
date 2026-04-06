<div align="center">

<img src="https://raw.githubusercontent.com/solobank-ai/.github/main/assets/logo.jpg" width="180" />

# Solobank

### Bank for AI Agents. Give your Agent a financial life.

[![npm](https://img.shields.io/npm/v/@solobank/sdk?label=sdk&color=black)](https://www.npmjs.com/package/@solobank/sdk)
[![npm](https://img.shields.io/npm/v/@solobank/cli?label=cli&color=black)](https://www.npmjs.com/package/@solobank/cli)
[![npm](https://img.shields.io/npm/v/@solobank/mcp?label=mcp&color=black)](https://www.npmjs.com/package/@solobank/mcp)
[![Gateway](https://img.shields.io/badge/gateway-mpp.solobank.lol-blue)](https://mpp.solobank.lol/health)

**Wallet** &middot; **Payments** &middot; **Swaps** &middot; **Lending** &middot; **MCP** &middot; **Pay-per-call APIs**

[Install](#install) &middot; [Talk to your Agent](#talk-to-your-agent) &middot; [Architecture](#architecture) &middot; [Docs](https://solobank.lol/docs)

</div>

---

## What is Solobank?

Solobank gives AI agents a full financial stack on Solana: a non-custodial wallet, token transfers, Jupiter swaps, Kamino & MarginFi lending, and pay-per-call access to 50+ APIs — all controllable via CLI, SDK, or MCP (Model Context Protocol).

```
Agent needs data  ->  solobank pay https://mpp.solobank.lol/openai/v1/chat/completions
Agent earns idle  ->  solobank lend 100 USDC --protocol auto
Agent rebalances  ->  solobank rebalance 50 USDC --min-apy-delta 0.5
```

## Install

```bash
npm install -g @solobank/cli
solobank init
```

The init wizard creates a wallet, configures safeguards, and auto-installs the MCP server into Claude Desktop, Cursor, and Windsurf.

## Demo

<div align="center">

```
🟡 🟢 🔴
```

</div>

```
◎ Solobank — AI Bank Account on Solana

  Decentrathon 5.0 · Case 2: AI + Blockchain

  ▸ Step 1: Install & create wallet

  $ curl -fsSL https://solobank.lol/install.sh | bash

  ✓ @solobank/cli installed
  ✓ Wallet created: Aa78c...SaNT
  ✓ MCP server configured for Claude Desktop

  ▸ Step 2: Agent checks DeFi rates (GPT-4o-mini decides)

  $ solobank lend-rates USDC

  Protocol     Market     APY       APR
  kamino       main       8.21%     7.89%
  marginfi     main       7.05%     6.81%

  🤖 AI decision: "Kamino offers 1.16% higher APY with similar risk"
     Action:     lend 100 USDC to Kamino
     Confidence: 85%
     Cost:       $0.00009 (GPT-4o-mini, 397 tokens)

  ▸ Step 3: Agent executes on-chain

  $ solobank lend 100 USDC --protocol kamino

  ✓ Supplied 100 USDC to Kamino (main market)
    APY: 8.21% · Earning ~$0.022/day
    Signature: 5ScEUtUfzq4SdDti...
    Explorer: https://solscan.io/tx/5ScEU...

  ▸ Step 4: Agent pays for API with USDC

  $ solobank pay https://mpp.solobank.lol/openai/v1/chat/completions \
      --method POST --data '{"model":"gpt-4o","messages":[...]}'

  ✓ 402 challenge: $0.01 USDC → Aa78c...SaNT
  ✓ TX confirmed on Solana (1361ms)
  ✓ Gateway verified (355ms via Helius)
  ✓ OpenAI responded: "Hello! How can I help you?"

  ▸ Step 5: Safeguards protect the wallet

  $ solobank config set maxPerTx 50
  ✓ maxPerTx = 50

  $ solobank lock
  🔒 Wallet locked. All transactions disabled.
  ⚠  AI can lock, but only human can unlock via CLI.

  ▸ Step 6: MCP tools for Claude / Cursor

  18 tools available:
  balance · send · pay · swap · swap_quote · lend · borrow
  withdraw · repay · rebalance · lending_rates · lock · config
  address · vault_status · vault_deposit · vault_analyze · vault_decide

  $ solobank mcp install
  ✓ Installed in Claude Desktop
  ✓ Installed in Cursor
```

```
  Results:
  ─────────────────────────────────────
  Settlement:       1361ms (Solana)
  Gateway verify:    355ms (Helius)
  AI decision:      ~200ms (GPT-4o-mini)
  API cost:        $0.005-$0.10/call
  AI cost:         $0.0001/decision
  ─────────────────────────────────────
  ✅ Mainnet USDC payments working
  ✅ 51 API services available
  ✅ Replay protection (409 Conflict)
```

---

## Talk to your Agent

After installing, just ask your AI agent in natural language:

> "Check my balance"
>
> "Send 5 USDC to alice"
>
> "Swap 1 SOL to USDC"
>
> "Find the best lending rate for USDC and deposit 100"
>
> "What APIs are available? Call OpenAI to summarize this article"
>
> "Lock my wallet, something looks wrong"

The agent uses Solobank MCP tools behind the scenes — no commands to memorize.

## CLI Commands

For direct use or scripting:

```bash
solobank balance                          # Check balances
solobank send 5 alice --asset USDC        # Send to contact
solobank swap 1 SOL USDC                  # Swap via Jupiter
solobank lend 100 USDC --protocol auto    # Lend to best APY
solobank lend-rates USDC                  # Compare rates
solobank history                          # Transaction history
solobank pay https://mpp.solobank.lol/openai/v1/chat/completions \
  --method POST --data '{"model":"gpt-4o","messages":[...]}'
```

## Architecture

```
┌───────────────────────────────────────────────────────────┐
│                       AI Agent                            │
│          (Claude, GPT, Cursor, custom agent)              │
└─────────┬──────────────────┬──────────────────────────────┘
          │                  │
    MCP (stdio)         SDK / CLI
          │                  │
┌─────────▼──────────────────▼──────────────────────────────┐
│                   @solobank/sdk                           │
│                                                           │
│  Wallet · Send · Swap · Lend · Borrow · Pay               │
│  Safeguards · Contacts · History · Session                │
└─────┬──────────┬───────────┬───────────┬──────────────────┘
      │          │           │           │
  Solana RPC   Jupiter    Kamino     MarginFi
      │        Aggregator  (klend)   (v2 SDK)
      │          │           │           │
┌─────▼──────────▼───────────▼───────────▼──────────────────┐
│                     Solana Blockchain                     │
│           ┌──────────────────────────────┐                │
│           │  Treasury Contract (Anchor)  │                │
│           │  save 0.1% · borrow 0.05%    │                │
│           │  swap 0.1% · admin controls  │                │
│           └──────────────────────────────┘                │
└───────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────┐
│                    MPP Gateway                            │
│              mpp.solobank.lol                             │
│                                                           │
│  50+ APIs: OpenAI · Anthropic · Gemini · Groq · Mistral   │
│  Perplexity · fal.ai · Brave · ElevenLabs · Replicate     │
│  CoinGecko · Jupiter · Helius · Birdeye · DexScreener     │
│  and 35+ more...                                          │
│                                                           │
│  Agent pays USDC on-chain -> Gateway verifies -> Proxy    │
└───────────────────────────────────────────────────────────┘
```

## Repositories

| Repo | Description |
|------|-------------|
| **[package](https://github.com/solobank-ai/package)** | SDK, CLI, and MCP server monorepo |
| **[backend](https://github.com/solobank-ai/backend)** | MPP Gateway — pay-per-call API proxy |
| **[mpp-solana](https://github.com/solobank-ai/mpp-solana)** | Solana USDC payment method for MPP |
| **[contracts](https://github.com/solobank-ai/contracts)** | Anchor smart contracts for fee collection |

## SDK

```typescript
import { Solobank } from '@solobank/sdk';

const agent = await Solobank.create();

// Balance
const { sol, usdc } = await agent.getBalance();

// Send
await agent.send({ to: 'alice', amount: 5, asset: 'USDC' });

// Swap
await agent.swap({ fromAsset: 'USDC', toAsset: 'SOL', amount: 25 });

// Lend
await agent.lend({ asset: 'USDC', amount: 100, protocol: 'auto' });

// Pay for API
const response = await agent.pay({
  url: 'https://mpp.solobank.lol/openai/v1/chat/completions',
  method: 'POST',
  body: { model: 'gpt-4o', messages: [{ role: 'user', content: 'hello' }] },
});
```

## MCP Server

15 tools + 16 prompt templates for AI agents:

```json
{
  "mcpServers": {
    "solobank": {
      "command": "npx",
      "args": ["-y", "@solobank/mcp"]
    }
  }
}
```

**Tools:** `balance` · `send` · `pay` · `swap` · `swap_quote` · `lend` · `borrow` · `withdraw` · `repay` · `rebalance` · `lending_rates` · `services` · `lock` · `config` · `address`

**Prompts:** `financial-report` · `optimize-yield` · `morning-briefing` · `send-money` · `emergency` · `sweep` · `risk-check` · `onboarding` · and 8 more

## Safeguards

Built-in spending controls that persist across restarts:

```bash
solobank config set maxPerTx 100      # Max $100 per transaction
solobank config set maxDailySend 500   # Max $500 per day
solobank lock                          # Emergency freeze (only CLI can unlock)
```

The MCP server **refuses to start** until safeguards are configured. AI agents can lock the wallet but cannot unlock it — only humans can via the CLI.

## MPP Gateway

The gateway at `mpp.solobank.lol` turns any API into a pay-per-call service. Agents pay with USDC on Solana — no API keys, no subscriptions.

```bash
# List available services
curl https://mpp.solobank.lol/services

# 50+ services from $0.001 to $0.10 per call
```

## Stack

**SDK:** TypeScript · @solana/kit v2 · Jupiter · Kamino · MarginFi · mppx

**Gateway:** Hono · PostgreSQL · Redis · Docker

**Contracts:** Rust · Anchor · SPL Token

---

<div align="center">

**Solobank** — Give your Agent a financial life.

Built on Solana.

</div>
