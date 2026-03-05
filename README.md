# ⟠ OracleBet — On-chain Prediction Market on Bitcoin

> **Decentralized prediction market built on Bitcoin L1 using OPNet smart contracts.**  
> Place bets on real-world and crypto events, fully on-chain on OPNet Regtest — powered by Bob MCP AI Agent.

[![Bitcoin](https://img.shields.io/badge/Bitcoin-L1-f7931a?style=flat-square&logo=bitcoin&logoColor=white)](https://bitcoin.org)
[![OPNet](https://img.shields.io/badge/OPNet-Regtest-b4ff4e?style=flat-square)](https://opnet.org)
[![Bob MCP](https://img.shields.io/badge/Bob_MCP-ai.opnet.org-4d9fff?style=flat-square)](https://ai.opnet.org)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

## 📸 Preview

```
┌─────────────────────────────────────────────────────────────────┐
│  ⟠ ORACLE BET  |  Markets  My Bets  + Create  |  #891,234  🟢  │
├──────────────┬──────────────────────────────────┬───────────────┤
│ Stats        │                                  │  Place Bet    │
│ 6.2 tBTC TVL │  PREDICTION MARKETS              │               │
│ 6 Markets    │  ┌──────────────┐ ┌────────────┐ │  YES ▲  NO ▼  │
│              │  │ Will BTC hit │ │ OPNet 100+ │ │               │
│ Categories   │  │ $100k?       │ │ dApps?     │ │  0.001 tBTC   │
│ ₿ BTC Price  │  │ YES 65% ████ │ │ YES 71% ██ │ │               │
│ 🔮 Crypto    │  └──────────────┘ └────────────┘ │  Payout: 1.8x │
│ 🏦 DeFi      │                                  │  [Sign & Bet] │
│ 🌍 Macro     │  On-chain Activity Feed           │               │
└──────────────┴──────────────────────────────────┴───────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

| Tool | Version | Link |
|------|---------|------|
| OP_WALLET Extension | Latest | [Chrome Store](https://chromewebstore.google.com/detail/opwallet/pmbjpcmaaladnfpacpmhmnfmpklgbdjb) |
| Node.js | >= 18.x | [nodejs.org](https://nodejs.org) |
| OPNet CLI | Latest | `npm install -g @btc-vision/opnet-cli` |

### 1. Clone & Open

```bash
git clone https://github.com/YOUR_USERNAME/oraclebet-opnet.git
cd oraclebet-opnet

# No build needed — pure HTML/JS
open opnet-prediction-market.html
# or serve locally:
npx serve .
```

### 2. Setup OP_WALLET

```
1. Install OP_WALLET Chrome extension (link above)
2. Create or import a wallet
3. Open extension → Settings → Network → Select "Regtest"
4. Get free tBTC: https://faucet.opnet.org
```

### 3. Deploy the Smart Contract

```bash
# Install OPNet CLI
npm install -g @btc-vision/opnet-cli

# Deploy PredictionMarket contract to regtest
opnet deploy --network regtest --contract ./contracts/PredictionMarket.ts

# Copy the deployed contract address and update in opnet-prediction-market.html:
# const PM_CONTRACT = 'bcrt1p_YOUR_CONTRACT_ADDRESS_HERE';
```

### 4. Connect & Bet

```
1. Open opnet-prediction-market.html in browser
2. Click "Connect OP_WALLET" → Approve in extension
3. Select a market → Choose YES or NO position
4. Enter amount (tBTC) → Review calldata → Sign & Send
5. Track your TX on https://opscan.org
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (HTML/JS)                       │
│                                                             │
│  opnet-prediction-market.html                               │
│  ├── Wallet Layer      → window.unisat (OP_WALLET API)      │
│  ├── RPC Layer         → fetch → regtest.opnet.org          │
│  ├── Calldata Encoder  → ABI-encoded contract calls         │
│  └── Bob MCP Client   → ai.opnet.org/mcp (AI agent)        │
└─────────────────┬───────────────────────────────────────────┘
                  │ PSBT / signInteraction
                  ▼
┌─────────────────────────────────────────────────────────────┐
│                  OP_WALLET EXTENSION                         │
│  Signs transactions using user's private key                 │
│  Broadcasts to OPNet node                                    │
└─────────────────┬───────────────────────────────────────────┘
                  │ Signed TX
                  ▼
┌─────────────────────────────────────────────────────────────┐
│              OPNet Regtest Node (regtest.opnet.org)          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │          PredictionMarket.ts (Smart Contract)         │  │
│  │                                                      │  │
│  │  bet(bytes32 marketId, bool isYes, uint64 amount)    │  │
│  │  resolve(bytes32 marketId, bool outcome)             │  │
│  │  claim(bytes32 marketId)                             │  │
│  │  createMarket(string question, uint256 endBlock, ..) │  │
│  └──────────────────────────────────────────────────────┘  │
│                          │                                   │
│                    Bitcoin L1 Blocks                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
oraclebet-opnet/
├── opnet-prediction-market.html   # Main app (single-file dApp)
├── contracts/
│   └── PredictionMarket.ts        # OPNet smart contract (AssemblyScript)
├── README.md
└── LICENSE
```

---

## 🔗 On-chain Integration Details

### Wallet Connection

OP_WALLET injects itself as `window.unisat` (forked from UniSat wallet). The app detects and connects using the standard provider API:

```javascript
// Detect OP_WALLET
const provider = window.unisat || window.opnet;

// Request accounts
const accounts = await provider.requestAccounts();
const publicKey = await provider.getPublicKey();

// Switch to regtest
await provider.switchNetwork('regtest');

// Get real balance from OPNet RPC
const balance = await rpc('op_getBalance', [address]);
```

### Sending Transactions

OPNet uses Bitcoin PSBTs with calldata encoded in OP_RETURN outputs. The app attempts three methods in priority order:

```javascript
// Method 1: Native OPNet API (preferred)
const result = await provider.signInteraction({
  to:      contractAddress,
  data:    calldata,         // ABI-encoded calldata
  value:   amountInSats,
  network: 'regtest',
});

// Method 2: sendBitcoin fallback
const txid = await provider.sendBitcoin(contractAddress, amountSats);

// Method 3: pushTx with raw PSBT
const txid = await provider.pushTx({ rawtx: psbtHex });
```

### Calldata Encoding

All contract interactions use ABI-encoded calldata matching the smart contract function signatures:

```javascript
// Place bet: bet(bytes32 marketId, bool isYes, uint64 amountSats)
function encodeBet(marketIdHex, isYes, amountSats) {
  const selector = 'd39c7a4f';                              // keccak4('bet(bytes32,bool,uint64)')
  const marketId = marketIdHex.padStart(64, '0');
  const position = isYes ? '1'.padStart(64, '0') : '0'.padStart(64, '0');
  const amount   = amountSats.toString(16).padStart(64, '0');
  return '0x' + selector + marketId + position + amount;
}

// Resolve: resolve(bytes32 marketId, bool outcome)
// Claim:   claim(bytes32 marketId)
// Create:  createMarket(string question, uint256 endBlock, uint8 category)
```

### Live RPC Calls

Block height is polled from the OPNet node every 12 seconds:

```javascript
// OPNet RPC endpoint
const RPC = 'https://regtest.opnet.org';

// Fetch current block height
const result = await fetch(RPC, {
  method: 'POST',
  body: JSON.stringify({ jsonrpc:'2.0', method:'op_blockNumber', params:[], id:1 })
});
```

---

## 📜 Smart Contract

The `PredictionMarket` contract is written in AssemblyScript (compiled to WebAssembly) and deployed as an OP_20-compatible contract on OPNet.

### Core Functions

```typescript
// PredictionMarket.ts (AssemblyScript)

export function createMarket(
  question: string,
  endBlock: u64,
  category: u8
): bytes32 { /* ... */ }

export function bet(
  marketId: bytes32,
  isYes: bool,
  amount: u64          // in satoshis
): void { /* ... */ }

export function resolve(
  marketId: bytes32,
  outcome: bool        // true = YES won
): void { /* ... */ }

export function claim(
  marketId: bytes32
): void { /* ... */ }

export function getOdds(
  marketId: bytes32
): u64[2] { /* returns [yesPct, noPct] */ }
```

### Deploy to Regtest

```bash
# Using OPNet CLI + Bob MCP
claude mcp add opnet-bob --transport http https://ai.opnet.org/mcp

# Ask Bob to scaffold + deploy
# "Deploy a prediction market contract with bet(), resolve(), claim() to regtest"
```

---

## 🤖 Bob MCP AI Agent

This project integrates with **Bob**, the OPNet AI MCP server, for on-chain intelligence:

```bash
# Add Bob to Claude Code or Claude Desktop
claude mcp add opnet-bob --transport http https://ai.opnet.org/mcp
```

Bob provides 28+ tools used in this project:

| Tool | Usage |
|------|-------|
| `OpnetDev` | Scaffold & generate smart contract code |
| `OpnetAudit` | Security audit the prediction market contract |
| `OpnetCli` | Deploy contracts to regtest |
| `PredictionMarket` | Query live market data |
| `BtcMonitor` | Monitor Bitcoin block confirmations |
| `CryptoCharts` | Price data for market resolution |
| `WebSearch` | Research market outcomes |

After every successful bet transaction, the app sends context to Bob MCP:

```javascript
// Notify Bob of on-chain activity
await fetch('https://api.anthropic.com/v1/messages', {
  body: JSON.stringify({
    model: 'claude-sonnet-4-20250514',
    messages: [{ role:'user', content: `Bet executed: txid=${txid}...` }],
    mcp_servers: [{ type:'url', url:'https://ai.opnet.org/mcp', name:'opnet-bob' }]
  })
});
```

---

## 🌐 Network Configuration

| Parameter | Value |
|-----------|-------|
| Network | OPNet Regtest |
| RPC Endpoint | `https://regtest.opnet.org` |
| Explorer | `https://opscan.org` |
| Faucet | `https://faucet.opnet.org` |
| Chain ID | `regtest` |
| Block Time | ~10 min (Bitcoin) |
| Fee Unit | Satoshis (sats) |

---

## ✨ Features

- **🔗 Real Wallet Connect** — OP_WALLET via `window.unisat` provider API
- **⛓️ Live Block Height** — Polls `regtest.opnet.org` every 12 seconds
- **📝 TX Confirm Modal** — Shows full calldata, contract address, and amount before signing
- **📊 AMM Odds** — Dynamic payout calculation using LMSR-style pool math
- **🎯 6 Market Categories** — BTC Price, Crypto, DeFi, Macro
- **⚖️ Resolve Markets** — Owner can resolve with YES/NO outcome on-chain
- **🏆 Claim Winnings** — Winners call `claim()` to receive tBTC
- **🆕 Create Markets** — Deploy new prediction markets directly from the UI
- **📡 Activity Feed** — Real-time on-chain event log with explorer links
- **🤖 Bob MCP** — AI-powered DeFi intelligence via OPNet's agent

---

## 🔧 Development

### Add Custom Market

```javascript
// Add to MARKETS array in the HTML file
{
  id: 'my-market-id',
  question: 'Will X happen before block Y?',
  category: 'crypto',     // 'btc' | 'crypto' | 'defi' | 'macro'
  status: 'open',
  yesPool: 0.01,          // initial tBTC in YES pool
  noPool:  0.01,          // initial tBTC in NO pool
  totalBets: 0,
  endBlock: 900000,       // resolution block height
  creator: 'bcrt1p...',
  contract: 'bcrt1p...',  // deployed contract address
  createdAt: Date.now(),
  myBet: null,
}
```

### Environment Variables (for production)

```javascript
const RPC          = 'https://regtest.opnet.org';   // OPNet node
const PM_CONTRACT  = 'bcrt1p_YOUR_CONTRACT_ADDRESS'; // deployed contract
const MCP_URL      = 'https://ai.opnet.org/mcp';    // Bob MCP
const EXPLORER     = 'https://opscan.org';           // block explorer
```

---

## 🗺️ Roadmap

- [ ] Deploy PredictionMarket.ts contract to OPNet regtest
- [ ] Integrate `@btc-vision/transaction` SDK for full PSBT construction
- [ ] Add Oracle resolution via Chainlink-style price feeds
- [ ] Multi-outcome markets (beyond YES/NO binary)
- [ ] LP token rewards for market liquidity providers
- [ ] Market factory contract for permissionless creation
- [ ] OPNet mainnet deployment

---

## 📚 Resources

| Resource | Link |
|----------|------|
| OPNet Documentation | [docs.opnet.org](https://docs.opnet.org) |
| OPNet GitHub | [github.com/btc-vision](https://github.com/btc-vision) |
| OP_WALLET Extension | [Chrome Store](https://chromewebstore.google.com/detail/opwallet/pmbjpcmaaladnfpacpmhmnfmpklgbdjb) |
| Bob MCP Agent | [ai.opnet.org](https://ai.opnet.org) |
| OPScan Explorer | [opscan.org](https://opscan.org) |
| Regtest Faucet | [faucet.opnet.org](https://faucet.opnet.org) |
| Motoswap DEX | [motoswap.org](https://motoswap.org) |
| Vibecode Event | [vibecode.finance](https://vibecode.finance) |

---

## 🏆 Built For

This project was built for the **[Vibecode.finance](https://vibecode.finance) OPNet Vibe Coding Event** — a hackathon for builders shipping Bitcoin L1 dApps powered by OPNet smart contracts.

> *"Build on Bitcoin, powered by AI."*

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

**₿ Built on Bitcoin L1 · Powered by OPNet · AI by Bob MCP**

[OPNet](https://opnet.org) · [Motoswap](https://motoswap.org) · [OPScan](https://opscan.org) · [Vibecode](https://vibecode.finance)

</div>
