# MCHPai Trench ⚡

> 77 Alpha Wallet Copy Trader · Solana · GitHub Pages · No server required

A fully self-contained browser trading app. Connect Phantom, paste your Helius key, go live. Everything runs in your browser — no backend, no server, no node.js.

---

## What it does

- **Monitors 77 alpha wallets** in real time via PumpPortal WebSocket
- **Auto-copies buys** using PumpPortal Local API → Phantom signs → multi-RPC race
- **Auto-exits** via trailing stop (5% from peak), hard stop (10% from entry), take profit (30%)
- **Mirror exit** — when an alpha wallet sells a token you hold, you sell immediately
- **Circuit breaker** — halts trading after 5 consecutive losses
- **Daily limit** — max 50 trades per 24h
- **SOL reserve** — never trades below minimum wallet balance
- **All settings adjustable** from the mobile UI without redeployment

---

## Deploy to GitHub Pages (5 minutes)

### Step 1 — Fork or create repo

```bash
# Option A: Use GitHub web UI
# 1. Go to github.com/new
# 2. Name it "mchpai-trench" (or anything)
# 3. Upload index.html

# Option B: Git CLI
git init mchpai-trench
cd mchpai-trench
cp /path/to/index.html .
git add .
git commit -m "MCHPai Trench v1"
git remote add origin https://github.com/YOUR_USERNAME/mchpai-trench.git
git push -u origin main
```

### Step 2 — Enable GitHub Pages

1. Go to your repo → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** · Folder: **/ (root)**
4. Click **Save**
5. Your site will be live at `https://YOUR_USERNAME.github.io/mchpai-trench`

That's it. No build step. No npm. No server.

---

## Usage

### First time setup

1. Open your GitHub Pages URL
2. Paste your **Helius RPC** key (get free at [helius.dev](https://helius.dev))
3. Optionally add a backup RPC (QuickNode, 2nd Helius key)
4. Click **Connect Phantom**
5. Approve the connection in Phantom
6. The app starts monitoring immediately

### How trading works

```
Alpha wallet buys a token on pump.fun
    ↓
PumpPortal WebSocket delivers the event (~50-200ms)
    ↓
Risk checks pass (CB, daily limit, balance)
    ↓
POST to pumpportal.fun/api/trade-local → get unsigned tx
    ↓
Phantom pops up → you approve (~1-2 seconds)
    ↓
Multi-RPC race: Helius + backup + public → fastest wins
    ↓
Position opened · price monitor starts
    ↓
Exits at TP / trail stop / hard stop / mirror exit
```

### Settings (adjustable live in the UI)

| Setting | Default | Description |
|---------|---------|-------------|
| Take Profit | 30% | Close position at this gain |
| Hard Stop | 10% | Close position at this loss |
| Trailing Stop | 5% | Close if price drops 5% from peak |
| Trade Size | 0.5 SOL | Max SOL per copy trade |
| SOL Reserve | 0.1 SOL | Always keep this much in wallet |
| Daily Limit | 50 | Max trades per 24h |
| Circuit Breaker | 5 losses | Halt after N consecutive losses |

---

## Performance expectations

Running from home (Youngstown / 1Gbps):

| Stage | Expected |
|-------|---------|
| PumpPortal event delivery | 50–200ms |
| Phantom sign prompt | 1–3s (user action) |
| Multi-RPC submit | 20–50ms |
| Transaction lands | 1–3 Solana slots |

> **Note:** Phantom sign requires user interaction each trade. For fully automated trading with no confirmation prompts, use the Rust engine (`mchpai-copytrade`) instead — it runs headlessly with your private key stored server-side.

---

## Wallet tiers

| Tier | Count | Win Rate | Notes |
|------|-------|----------|-------|
| S-tier | 40 | 60-100% 7d WR | Top performers — copy immediately |
| A-tier | 37 | 50-60% 7d WR | Good performers — copy with normal sizing |

Ranks 1-5 (100% WR) are weighted heaviest. Circuit breaker threshold is relaxed for elite wallets.

---

## Security

- Your private key **never leaves your browser** — Phantom keeps it
- RPC keys are stored in `localStorage` on your device only
- No telemetry, no analytics, no tracking
- The only external connections are:
  - `wss://pumpportal.fun/api/data` — trade data (free)
  - `https://pumpportal.fun/api/trade-local` — unsigned tx construction
  - Your Helius/QuickNode RPC — tx submission

---

## Files

```
mchpai-trench/
├── index.html    ← the entire app (no dependencies to install)
└── README.md
```

---

## Advanced: run locally

```bash
# Any static file server works
python3 -m http.server 8080
# Open http://localhost:8080
```

---

## Difference vs Rust engine

| Feature | GitHub Pages (this) | Rust Engine |
|---------|-------------------|-------------|
| Hosting | GitHub Pages (free) | Your desktop/VPS |
| Data feed | PumpPortal WebSocket | Yellowstone gRPC |
| Detection latency | 50–200ms | 10–30ms |
| Execution | Phantom sign (manual) | Fully automated |
| Private key | Stays in Phantom | In .env file |
| Best for | Mobile trading, monitoring | High-frequency automation |

---

## Disclaimer

This is trading software. Memecoin trading carries extreme risk. You can lose your entire position. Start with the minimum trade size and test thoroughly before increasing position sizes.
