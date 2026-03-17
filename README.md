# FlipCoin Skill for OpenClaw

Trade prediction markets on [FlipCoin](https://www.flipcoin.fun) directly from your AI assistant via Telegram or WhatsApp.

## What is FlipCoin?

FlipCoin is a prediction markets platform on Base blockchain. You buy YES or NO shares on real-world events. If you're right, each share pays $1. If wrong, $0. All trades use USDC.

## What This Skill Does

- **Browse markets** — search and filter open prediction markets
- **Trade** — buy/sell YES and NO shares with natural language
- **Portfolio** — check your positions, PnL, and resolved markets
- **Create markets** — launch new prediction markets
- **Alerts** — periodic notifications on price moves and expiring markets (via heartbeat)

## Installation

### From ClawHub

```bash
clawhub install flipcoin-fun/flipcoin-openclaw-skill
```

### From GitHub

```bash
openclaw skills add https://github.com/flipcoin-fun/flipcoin-openclaw-skill
```

### Manual

Clone this repo and symlink into your OpenClaw skills directory:

```bash
git clone https://github.com/flipcoin-fun/flipcoin-openclaw-skill.git
ln -s $(pwd)/flipcoin-openclaw-skill ~/.openclaw/workspace/skills/flipcoin
```

Or copy the files directly:

```bash
mkdir -p ~/.openclaw/workspace/skills/flipcoin
cp SKILL.md ~/.openclaw/workspace/skills/flipcoin/SKILL.md
# Optionally merge HEARTBEAT.md into your workspace HEARTBEAT.md
```

## Setup

### 1. Create an Agent & Get an API Key

1. Go to [flipcoin.fun/agents](https://www.flipcoin.fun/agents)
2. Connect your wallet (MetaMask, Rabby, Coinbase Wallet, etc.)
3. Click **Add Agent** — fill in the agent name, description, and other fields
4. Click **Add Key** to generate an API key
5. Copy the key — it starts with `fc_` and is shown only once!

### 2. Configure OpenClaw

Set your API key as an environment variable:

```bash
# In your OpenClaw config or shell profile
export FLIPCOIN_API_KEY=fc_your_key_here
```

Or add it to your OpenClaw workspace config (`openclaw.json`):

```json
{
  "skills": {
    "entries": {
      "flipcoin": {
        "env": {
          "FLIPCOIN_API_KEY": "fc_your_key_here"
        }
      }
    }
  }
}
```

### 3. Enable Trading (Optional)

For read-only access (browsing markets, checking prices), the API key is enough.

To trade, you need two more things:

1. **Deposit USDC to Vault** — your wallet USDC balance is NOT the same as Vault balance
   - Go to [/agents](https://www.flipcoin.fun/agents) or [/settings](https://www.flipcoin.fun/settings) and click **Add Funds**
   - This handles USDC approval + deposit in one flow
   - Minimum: $35 (low tier), $139 (medium), $693 (high). Trial markets are free.

2. **Create Autopilot Key (session key)** — enables auto-signing trades without manual wallet confirmations
   - Go to [/agents](https://www.flipcoin.fun/agents) → select your agent
   - Click **Create Autopilot Key** (choose 24h or 7d duration)
   - Sign the `setDelegation()` transaction when your wallet prompts
   - Wait for on-chain confirmation

Without these, you can browse markets but trades will fail.

---

## Example Conversations

### Browsing Markets

```
You: What markets are open on FlipCoin?

Bot: Here are the top markets by volume on FlipCoin:

  1. Will BTC reach $100k by June? — YES: 62%, Volume: $4,200
  2. Will ETH flip SOL in market cap? — YES: 28%, Volume: $1,850
  3. Will Base hit 10M daily txns? — YES: 73%, Volume: $980

  Want details on any of these, or want to place a trade?

You: Tell me more about the Bitcoin one

Bot: Will BTC reach $100k by June?

  - YES: 62% / NO: 38%
  - Volume: $4,200 (42 trades)
  - Liquidity: $139
  - Expires: June 30, 2026 (105 days left)
  - Resolution: CoinGecko BTC/USD price

  Last trades: 3 buys in the past hour, price moved up from 58%.
  Want to buy YES or NO?
```

### Trading

```
You: Buy $10 of YES on the Bitcoin market

Bot: Placing a $10 YES buy on "Will BTC reach $100k by June?"...

  Executed! Here's your trade:
  - Bought 15.8 YES shares at avg price $0.63
  - Fee: $0.15
  - TX: 0xab12...ef56

  If BTC hits $100k, those shares pay $15.80. Your max loss is $10.

You: What does my portfolio look like?

Bot: Your FlipCoin Portfolio:

  1. Will BTC reach $100k by June?
     15.8 YES shares — current value: $9.80 (-$0.20)

  Total: 1 active market
```

### Selling

```
You: Sell my YES shares on the Bitcoin market

Bot: Selling 15.8 YES shares on "Will BTC reach $100k by June?"...

  Done! Received $10.25 USDC (after $0.15 fee).
  Profit: +$0.10

  Your portfolio is now empty.
```

### Creating a Market

```
You: Create a market about whether Apple will announce AR glasses at WWDC

Bot: Creating market on FlipCoin:

  Title: "Will Apple announce AR glasses at WWDC 2026?"
  Resolution: Apple Newsroom (apple.com/newsroom)
  Deadline: June 15, 2026
  Starting price: YES 30%
  Liquidity: Low ($35)

  Should I go ahead with these parameters?

You: Yes, do it

Bot: Market created!
  Address: 0x7f3a...2b1c
  Link: flipcoin.fun/app/market/will-apple-announce-ar-glasses-at-wwdc-2026

  You can share this link or start trading on it right away.
```

### Onboarding (First Time)

```
You: What can you do with FlipCoin?

Bot: FlipCoin is a prediction markets platform! You can:

  - Browse markets on real-world events (crypto, politics, sports, tech)
  - Buy YES or NO shares — if you're right, each share pays $1
  - Create your own markets and earn creator fees

  To get started, you'll need an API key:
  1. Go to flipcoin.fun/agents
  2. Connect your wallet
  3. Click "Add Agent" — fill in name and description
  4. Click "Add Key" to generate an API key (starts with fc_)

  Once you have it, tell me the key and I'll save it for future use.
  You can start by just browsing markets — no deposit needed for that!

  For trading, you'll also need to deposit USDC and create an
  Autopilot Key — I'll walk you through that when you're ready.

You: Here's my key: fc_abc123def456

Bot: Key saved! Let me verify...

  Connected as "My Trading Bot". Everything looks good.
  You have $50 USDC in your Vault — ready to trade.

  Want to see what markets are trending?
```

### Heartbeat Alert (Automatic)

```
Bot: FlipCoin update:

  Your position in "Will ETH hit $5k?" moved — YES now at 45%
  (was 38% yesterday). You hold 20 YES shares, current value: $9.00.

  Also: "Will Base hit 10M txns?" expires in 6 hours.
  You have 10 YES shares (YES at 73%).
```

---

## How It Works

```
flipcoin-openclaw-skill/
├── SKILL.md         # Main skill — API instructions, onboarding, error handling
├── HEARTBEAT.md     # Periodic market monitoring checklist
└── README.md        # This file
```

The skill uses FlipCoin's [Agent API](https://www.flipcoin.fun/docs/agent-api) exclusively via HTTP (curl). No SDK or binary dependencies required.

**API Flow for Trading:**
1. `POST /trade/intent` — get a quote and create a signed intent
2. `POST /trade/relay` with `auto_sign: true` — execute the trade on-chain
3. Intent must be relayed within 15 seconds (LMSR quote expiry)

**Two access modes:**
- **Read-only** (API key only) — browse markets, check prices, view portfolio
- **Full trading** (API key + Vault deposit + session key) — buy, sell, create markets

## Links

- [FlipCoin App](https://www.flipcoin.fun)
- [Agent API Docs](https://www.flipcoin.fun/docs/agent-api)
- [FlipCoin Discord](https://discord.gg/flipcoin)
- [OpenClaw](https://github.com/openclaw/openclaw)
- [ClawHub](https://github.com/openclaw/clawhub)

## License

MIT
