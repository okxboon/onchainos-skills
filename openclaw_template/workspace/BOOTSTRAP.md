# Bootstrap Protocol

**On the user's first message in every session, run ALL steps below before responding.** This file self-deletes after initial onboarding is complete.

## Bootstrap Gate

Before running any steps, check if bootstrap has already completed this session:

```bash
cat ~/.onchainos/bootstrap_status
```

- If the file contains today's date (e.g. `2026-04-28 OK`), **skip Steps 1–2** and go directly to Step 3 (Login check).
- If the file is missing, empty, or contains a different date, **run all steps from Step 1**.

**Welcome gate:** Also check `~/.onchainos/welcome_shown`:
- If it contains today's date, **skip Step 4** (welcome already shown this session).
- Otherwise, show the welcome message and write today's date to the file.

## Step 1 — Verify installation

Check that the CLI binary, workflows, and skills exist:

```bash
ls ~/.local/bin/onchainos
ls ~/.onchainos/workflows/
ls ~/.onchainos/skills/
```

If **any** of these are missing, run the setup script to install everything:

```bash
curl -sSL https://raw.githubusercontent.com/okx/onchainos-skills/main/openclaw_template/setup.sh | sh
```

Then verify all three paths again before proceeding.

## Step 2 — Configure PATH and skills

**This step is mandatory. Do not skip it.**

1. Verify the bare `onchainos` command resolves (the build symlinks it onto a PATH directory):

```bash
onchainos --version
echo "PATH configured: $(which onchainos)"
```

If `onchainos` is not found, the build did not symlink the binary into a runtime-PATH directory. As a session-local fallback, add `~/.local/bin` to PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
onchainos --version
```

If still not found, the binary was not installed correctly. Re-run the setup script from Step 1.

2. Symlink the skills directory into the workspace so OpenClaw can discover them:

```bash
mkdir -p skills
ln -sf ~/.onchainos/skills/* skills/
```

3. Verify onchainos skills are loaded:

```bash
openclaw skills list
```

Check the output and confirm that onchainos skills are present (e.g. `okx-dex-token`, `okx-dex-swap`, `okx-agentic-wallet`, etc.). If onchainos skills do not appear:
- Verify symlinks exist: `ls -la skills/`
- Verify source is populated: `ls ~/.onchainos/skills/`
- Re-run the symlink command from sub-step 2 above
- If still missing, re-run setup from Step 1

**Do not proceed until onchainos skills are confirmed in `openclaw skills list`.**

Note: `setup.sh` writes `~/.onchainos/bootstrap_status` on success, so subsequent messages in the same session will skip Steps 1–2 via the Bootstrap Gate.

## Step 3 — Login (REQUIRED)

This template requires the agentic wallet. **Refuse all on-chain commands** (token search, price lookup, swap, portfolio, signals, etc.) until login is complete.

Run:

```bash
onchainos wallet status
```

- **If logged in** (status returns a valid address): continue to Step 4.
- **If not logged in**: invoke the `okx-agentic-wallet` skill and follow this flow:

  1. Prompt the user:
     > To get started, log in with your email — I'll send a verification code. Your wallet is TEE-secured: I never see your private key.

  2. **Email path** (default — works with no secrets configured):
     - Ask for the user's email and locale (default `en`).
     - Run: `onchainos wallet login <email> --locale <locale>`
     - Prompt the user for the OTP code that arrives in their inbox.
     - Run: `onchainos wallet verify <code>`
     - On success, run `onchainos wallet status` again and show the user their wallet addresses.

  3. **API key path** (alternative — if `OKX_API_KEY`, `OKX_SECRET_KEY`, and `OKX_PASSPHRASE` are set in secrets): the CLI uses them automatically; no user action needed. Confirm with `onchainos wallet status`.

If both paths fail, tell the user clearly and stop. Do not proceed to Step 4 or accept any on-chain command until `wallet status` returns a valid address.

Record the wallet address in `IDENTITY.md` under a `## Wallet` section after first successful login.

## Step 4 — Welcome

Check `~/.onchainos/welcome_shown`. If it already contains today's date, skip this step. Otherwise, show the welcome message and persist:

```bash
echo "$(date +%Y-%m-%d)" > ~/.onchainos/welcome_shown
```

> Welcome to onchainos ⛓️
>
> **Workflows** — just say what you want:
> - 🔍 "Research this token: `<address>`" — price, security, holders, smart money signals
> - 📡 "What is smart money buying?" — SM signals with per-token due diligence
> - 🐸 "Scan new tokens on pump.fun" — MIGRATED tokens with safety & dev enrichment
> - 👛 "Analyse this wallet: `<address>`" — 7d/30d PnL, trading behaviour, activity
> - 📊 "Check my portfolio" — balances, total value, PnL overview
> - 📰 "Give me a daily brief" — market prices + hot tokens + smart money + new launches
> - 👁 "Watch this wallet: `<address>`" — alert me when it trades
>
> **Skills** — ask me directly about anything:
> - 🪙 Token search, price, holders, top traders, cluster analysis
> - 📈 Prices, K-line charts, wallet PnL
> - 🦈 Smart money / KOL / whale signals & leaderboard
> - 🐸 Meme/pump.fun scanning, dev reputation, bundle detection
> - 🔄 DEX swap execution across 500+ liquidity sources
> - ⚡ Real-time WebSocket monitoring
> - 🛡️ Token risk, DApp phishing, tx pre-execution scan
> - 💼 Public wallet balance & token holdings
> - 👛 Wallet: balance, send, tx history
> - 🔗 Gas estimation, tx simulation, broadcasting
> - 🌾 DeFi: discover, deposit, withdraw, claim rewards
> - 📈 DeFi portfolio across protocols
> - 💳 x402 gas-free payment authorization
> - 📋 Audit log & command history