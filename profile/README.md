<div align="center">

# Solobank

### A Bank Account for AI Agents on Solana

[![SDK](https://img.shields.io/npm/v/@solobank/sdk?label=sdk&color=14F195)](https://www.npmjs.com/package/@solobank/sdk)
[![CLI](https://img.shields.io/npm/v/@solobank/cli?label=cli&color=14F195)](https://www.npmjs.com/package/@solobank/cli)
[![MCP](https://img.shields.io/npm/v/@solobank/mcp?label=mcp&color=14F195)](https://www.npmjs.com/package/@solobank/mcp)
[![Gateway](https://img.shields.io/badge/gateway-mpp.solobank.lol-9945FF)](https://mpp.solobank.lol)
[![Solana](https://img.shields.io/badge/chain-Solana-black)](https://solana.com)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

**Wallet** · **Payments** · **Swaps** · **Lending** · **MCP** · **50+ Pay-per-call APIs**

[Install](#install) · [Demo](#demo) · [Architecture](#architecture) · [Docs](https://solobank.lol/docs)

</div>

---

## The Problem

AI agents can think, plan, and execute — but they can't hold money, earn yield, or pay for services. Every API call requires a human to manage keys, every financial operation requires manual approval, and there's no transparent way to let AI manage assets autonomously.

## The Solution

Solobank gives AI agents a **complete financial stack** on Solana:

| Capability | How it works |
|---|---|
| **Wallet** | Non-custodial Solana wallet with AES-256 encrypted keys |
| **Send** | Transfer SOL/USDC to any address or saved contact |
| **Swap** | Exchange tokens via Jupiter (20+ DEXs aggregated) |
| **Lend** | Earn yield on Kamino or MarginFi (auto-selects best APY) |
| **Borrow** | Borrow against collateral with health factor monitoring |
| **Pay APIs** | 50+ APIs (OpenAI, Anthropic, etc.) — pay per request with USDC |
| **Safeguards** | Per-tx limits, daily caps, emergency lock (AI can lock, only human unlocks) |

> *Your agent earns yield while idle, pays for APIs when working, and every action is verifiable on Solana.*

---

## Demo

### 1. Install (one command)

```bash
curl -fsSL https://solobank.lol/install.sh | bash
```

<details>
<summary>Windows (PowerShell)</summary>

```powershell
npm install -g @solobank/cli
solobank init
```
</details>

### 2. Talk to your AI agent

After installing, just ask in natural language:

```
You:    "Check my balance"
Agent:  ✓ SOL: 1.25 ($187.50) | USDC: $500.00

You:    "Find the best lending rate for USDC and deposit 100"
Agent:  ✓ Kamino 8.21% APY > MarginFi 7.05%
        ✓ Supplied 100 USDC to Kamino — earning ~$0.022/day

You:    "Swap 1 SOL to USDC"
Agent:  ✓ Swapped 1 SOL → 150.42 USDC via Jupiter (Raydium route)

You:    "Call OpenAI to summarize this article"
Agent:  ✓ Paid $0.01 USDC → mpp.solobank.lol/openai
        ✓ Response: "The article discusses..."

You:    "Lock my wallet, something looks wrong"
Agent:  🔒 Wallet locked. Only CLI can unlock.
```

### 3. Or use the CLI directly

```bash
solobank balance                           # SOL + USDC balances
solobank send 5 USDC alice                 # Send to contact
solobank swap 1 SOL USDC                   # Swap via Jupiter
solobank lend 100 USDC                     # Auto-select best APY
solobank lend-rates USDC                   # Compare Kamino vs MarginFi
solobank pay https://mpp.solobank.lol/openai/v1/chat/completions \
  --method POST --data '{"model":"gpt-4o","messages":[...]}'
solobank lock                              # Emergency freeze
solobank config set maxPerTx 50            # Spending limit
```

### 4. Mainnet payment test results

```
Gateway:     mpp.solobank.lol (Solana mainnet)
RPC:         Helius
Payment:     $0.005 USDC → Jupiter Price API

Timing:
  402 challenge:      252ms
  Build & sign TX:    164ms
  On-chain confirm:  1361ms  ← Solana finality
  Gateway verify:     355ms  ← Helius getTransaction
  ─────────────────────────
  Total:             2132ms

✓ Payment accepted
✓ Replay protection: 409 Conflict (blocked)
```

---

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
│  Wallet · Send · Swap · Lend · Borrow · Pay              │
│  Safeguards · Contacts · History · Session               │
└─────┬──────────┬───────────┬───────────┬──────────────────┘
      │          │           │           │
  Solana RPC   Jupiter    Kamino     MarginFi
      │        Aggregator  (klend)   (v2 SDK)
      │          │           │           │
┌─────▼──────────▼───────────▼───────────▼──────────────────┐
│                     Solana Blockchain                      │
│           ┌──────────────────────────────┐                │
│           │  Treasury Contract (Anchor)  │                │
│           │  save 0.1% · borrow 0.05%   │                │
│           │  swap 0.1% · admin controls  │                │
│           └──────────────────────────────┘                │
└───────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────┐
│                    MPP Gateway                            │
│              mpp.solobank.lol                             │
│                                                           │
│  50+ APIs: OpenAI · Anthropic · Gemini · Groq · Mistral  │
│  Perplexity · fal.ai · Brave · ElevenLabs · Replicate    │
│  CoinGecko · Jupiter · Helius · Birdeye · DexScreener    │
│  and 35+ more...                                          │
│                                                           │
│  Agent pays USDC on-chain -> Gateway verifies -> Proxy    │
└───────────────────────────────────────────────────────────┘
```

### Payment Flow (MPP Protocol)

```
Agent                          Gateway                    Solana
  │                              │                          │
  ├── POST /openai/v1/chat ─────►│                          │
  │◄── 402 {amount, recipient} ──┤                          │
  │                              │                          │
  ├── sign + send USDC ─────────────────────────────────────►│
  │◄── confirmed ───────────────────────────────────────────┤
  │                              │                          │
  ├── POST + x-payment-sig ─────►│                          │
  │                              ├── getTransaction() ─────►│
  │                              │◄── verified ─────────────┤
  │                              ├── proxy to OpenAI        │
  │◄── 200 + API response ──────┤                          │
```

---

## Hackathon Compliance

<div align="center">

| Criterion | Points | Status | Implementation |
|---|---|---|---|
| **Product & Idea** | 20 | ✅ | AI bank account — real use case, working on mainnet |
| **Technical Implementation** | 25 | ✅ | SDK + MCP + CLI + Gateway + Smart Contract |
| **Use of Solana** | 15 | ✅ | SPL tokens, Kamino, MarginFi, Jupiter, Helius RPC |
| **Innovation** | 15 | ✅ | AI autonomously manages DeFi + pays for APIs with USDC |
| **UX & Product Thinking** | 10 | ✅ | Natural language interface, safeguards, one-line install |
| **Demo & Presentation** | 10 | ✅ | Live mainnet payments, real OpenAI responses |
| **Completeness & Docs** | 5 | ✅ | 7 repos, full docs, agent skills, install script |

</div>

### AI → On-Chain Decision Flow

```
1. AI agent (GPT-4o-mini) analyzes DeFi rates
   Input:  Kamino USDC 8.21% APY, MarginFi USDC 7.05% APY

2. AI makes structured decision
   { action: "lend", asset: "USDC", amount: 100,
     protocol: "kamino", confidence: 85%, risk: "low" }

3. SDK executes on-chain
   → Kamino lend instruction → Solana TX confirmed

4. Verifiable on Solana Explorer
   → TX hash, amount, protocol — all transparent
```

---

## Repositories

| Repo | Description |
|---|---|
| **[package](https://github.com/solobank-ai/package)** | SDK, CLI, and MCP server (monorepo) |
| **[backend](https://github.com/solobank-ai/backend)** | MPP Gateway — 50+ pay-per-call APIs |
| **[contracts](https://github.com/solobank-ai/contracts)** | Anchor smart contracts |
| **[solobank_frontend](https://github.com/solobank-ai/solobank_frontend)** | Website — solobank.lol |
| **[solobank-skills](https://github.com/solobank-ai/solobank-skills)** | 11 agent skills for Claude/Cursor |
| **[mpp-solana](https://github.com/solobank-ai/mpp-solana)** | Solana USDC payment method |
| **[prices-tracker-bot](https://github.com/solobank-ai/prices-tracker-bot)** | Telegram price monitor |

## SDK

```typescript
import { Solobank } from '@solobank/sdk';

const agent = await Solobank.create();

await agent.balance();                                    // Check SOL + USDC
await agent.send({ to: 'alice', amount: 5, asset: 'USDC' });  // Send
await agent.swap({ fromAsset: 'SOL', toAsset: 'USDC', amount: 1 });
await agent.lend({ asset: 'USDC', amount: 100, protocol: 'auto' });
await agent.pay({ url: 'https://mpp.solobank.lol/openai/v1/chat/completions',
  method: 'POST', body: { model: 'gpt-4o', messages: [...] } });
```

## MCP Server — 18 Tools

```json
{ "mcpServers": { "solobank": { "command": "npx", "args": ["-y", "@solobank/mcp"] } } }
```

`balance` · `send` · `pay` · `swap` · `swap_quote` · `lend` · `borrow` · `withdraw` · `repay` · `rebalance` · `lending_rates` · `lock` · `config` · `address` · `vault_status` · `vault_deposit` · `vault_analyze` · `vault_decide`

## Stack

| Layer | Technology |
|---|---|
| **SDK** | TypeScript · @solana/kit · Jupiter · Kamino · MarginFi |
| **Gateway** | Hono · PostgreSQL · Redis · Docker · Helius RPC |
| **Contracts** | Rust · Anchor · SPL Token |
| **Frontend** | Next.js 16 · React 19 · Tailwind CSS |
| **AI** | OpenAI GPT-4o-mini (DeFi analysis) |
| **Deploy** | Vercel (frontend) · Docker + GitHub Actions (backend) |

---

<div align="center">

**Solobank** — Give your agent a financial life.

Built on Solana · Powered by AI

</div>
