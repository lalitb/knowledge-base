# OpenClaw + Raspberry Pi 5 + Ollama: A Practical Local Assistant Tutorial

> **Run a personal AI assistant on Raspberry Pi 5 with OpenClaw as the control plane and Ollama as the local model runtime — completely offline-capable.**

---

## Who Is This For?

You want a local-first, always-on AI assistant and you're okay with Raspberry Pi constraints (CPU-only inference, smaller models, slower responses than desktop GPUs). No cloud API keys required for the core path.

**What we're building:** an always-on OpenClaw gateway on Pi 5, backed by local Ollama models, with one messaging channel connected end-to-end.

---

## Table of Contents

- [Feasibility](#feasibility-short-answer)
- [Prerequisites & Hardware](#prerequisites)
- [Part 1: Prepare Raspberry Pi 5](#part-1-prepare-raspberry-pi-5)
- [Part 2: Install and Verify Ollama](#part-2-install-and-verify-ollama)
- [Part 3: Install OpenClaw](#part-3-install-openclaw)
- [Part 4: Start Gateway and Validate Local Agent Path](#part-4-start-gateway-and-validate-local-agent-path)
- [Part 5: Connect a Telegram Channel (MVP)](#part-5-connect-a-telegram-channel-mvp)
- [Part 6: Hardening for 24/7 Pi Usage](#part-6-hardening-for-247-pi-usage)
- [Part 7: Performance Tuning Playbook](#part-7-performance-tuning-playbook)
- [Part 8: Troubleshooting](#part-8-troubleshooting)
- [Part 9: Next Steps](#part-9-suggested-next-step-expansions)
- [Part 10: Pi-hole + AI Narrator (Bonus)](#part-10-pi-hole--ai-narrator-bonus)
- [Quick Validation Checklist](#quick-validation-checklist)

---

## Feasibility (Short Answer)

Yes, this works on Pi 5 for lightweight assistant tasks with small quantized models.

### What to Expect on Pi 5

| Area | Realistic Expectation |
|---|---|
| Model size | Best with 1.5B–3B models; 7B is possible but noticeably slower |
| First response | 10–30 seconds cold start (model loads into RAM); subsequent responses faster |
| Latency | Good for short prompts; long contexts get progressively slower |
| Hardware | Active cooling is strongly recommended |
| Stability | Good if you limit concurrency and keep context size moderate |

---

## Prerequisites

### Hardware Shopping List

| Item | Recommendation | Why |
|---|---|---|
| Raspberry Pi 5 | **8GB RAM** model | More headroom for models; 4GB works with strict limits |
| Power supply | Official 27W USB-C PSU | Pi 5 draws more power than Pi 4; underpowering causes throttling |
| Cooling | Active cooler (fan + heatsink) | Sustained inference heats up fast; passive cooling is not enough |
| Storage | **USB SSD recommended** over microSD | Faster model loading, much better longevity under heavy I/O |
| microSD (if no SSD) | 32GB+ A2-rated card | Minimum viable; expect slower model loads and shorter card life |
| Network | Stable LAN (Ethernet) or Wi-Fi | Required for onboarding and channel connectivity |

> **Estimated cost:** ~$80–120 USD for Pi 5 8GB + PSU + cooler + SSD.

### Software Requirements

| Requirement | Version | Notes |
|---|---|---|
| OS | Raspberry Pi OS 64-bit (Bookworm) | Must be 64-bit for Ollama |
| Node.js | 22+ | OpenClaw runtime requirement |
| npm or pnpm | latest | Needed for OpenClaw install |
| Ollama | latest | Local model runtime |

### Disk Space Budget

| Model | Approximate Size |
|---|---|
| qwen2.5:1.5b | ~1 GB |
| qwen2.5:3b | ~2 GB |
| qwen2.5:7b | ~4.5 GB |

Keep at least 5 GB free beyond your chosen model for OS, OpenClaw, swap, and logs.

---

## Part 0: Architecture You're Building

```text
Messaging Channel (Telegram/Discord/etc)
                |
                v
          OpenClaw Gateway (Pi 5, Node.js)
                |
                v
          Ollama (localhost:11434)
                |
                v
          Local LLM (qwen2.5:1.5b / 3b)
```

OpenClaw manages channels, routing, tools, and session behavior. Ollama runs the LLM locally on the Pi's CPU.

---

# Part 1: Prepare Raspberry Pi 5

## Step 1: Install Raspberry Pi OS (if your Pi is still blank)

On another computer:

1. Download and open **[Raspberry Pi Imager](https://www.raspberrypi.com/software/)**.
2. Choose **Raspberry Pi OS (64-bit)** — Bookworm or newer.
3. Select your microSD card (or USB SSD), then click Write.
4. In Imager advanced options (gear icon), preconfigure:
   - hostname (e.g., `clawd`),
   - username / password,
   - Wi-Fi credentials (if not using Ethernet),
   - **Enable SSH** (recommended for headless setup).

Insert the media into the Pi 5 and power on. Log in via SSH or local keyboard/monitor.

Checkpoint: confirm 64-bit ARM:

```bash
uname -m
# Expected: aarch64
```

## Step 2: Update System

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

## Step 3: Configure Swap / zram (Important for 4GB Pi, Helpful for 8GB)

Ollama can spike memory during model loading. Increase swap to prevent OOM kills:

```bash
# Check current swap
free -h

# Increase swap to 2GB (default is often 200MB)
sudo dphys-swapfile swapoff
sudo sed -i 's/CONF_SWAPSIZE=.*/CONF_SWAPSIZE=2048/' /etc/dphys-swapfile
sudo dphys-swapfile setup
sudo dphys-swapfile swapon

# Verify
free -h
```

For 4GB Pi, consider zram for compressed in-memory swap:

```bash
sudo apt install -y zram-tools
echo -e "ALGO=lz4\nPERCENT=50" | sudo tee /etc/default/zramswap
sudo systemctl restart zramswap
```

## Step 4: Install Core Utilities

```bash
sudo apt install -y curl git build-essential jq htop
```

## Step 5: Install Node.js 22+

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

Checkpoint: `node -v` prints `v22.x.x` or newer.

> ### ✅ What Just Happened?
>
> You have a fully updated 64-bit Pi 5 with adequate swap, Node.js 22+, and core
> build tools. The Pi is ready to host both Ollama and OpenClaw.

---

# Part 2: Install and Verify Ollama

## Step 1: Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

## Step 2: Enable and Start Service

```bash
sudo systemctl enable ollama
sudo systemctl start ollama
sudo systemctl status ollama --no-pager
```

## Step 3: Tune Ollama for Pi Constraints

Create an environment override to limit parallelism and context size:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo tee /etc/systemd/system/ollama.service.d/pi-tuning.conf > /dev/null <<'EOF'
[Service]
Environment="OLLAMA_NUM_PARALLEL=1"
Environment="OLLAMA_MAX_LOADED_MODELS=1"
EOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

This ensures Ollama only loads one model at a time and serves one request at a time — the right default for Pi 5.

## Step 4: Pull a Starter Model

Start small:

```bash
ollama pull qwen2.5:1.5b
```

Optional upgrade after your baseline is stable:

```bash
ollama pull qwen2.5:3b
```

## Step 5: Sanity Test and Benchmark

```bash
# Quick test
ollama run qwen2.5:1.5b "Say hello in one short sentence."

# Measure tokens/sec (look for "eval rate" in output)
curl -s http://localhost:11434/api/generate \
  -d '{"model":"qwen2.5:1.5b","prompt":"Explain RAM in one sentence.","stream":false}' \
  | jq '.eval_count, .eval_duration'
```

> **Expected on Pi 5 (8GB, 1.5B model):** roughly 5–15 tokens/sec depending on prompt length.
> The **first request** after a restart takes 10–30 seconds as the model loads into RAM. Subsequent requests are faster.

Checkpoint: model returns a response reliably without crashing or OOM.

> ### ✅ What Just Happened?
>
> Ollama is running as a systemd service, tuned for single-request Pi usage.
> You have a working local LLM with a measured baseline speed. This is the
> inference engine that OpenClaw will call.

---

# Part 3: Install OpenClaw

## Step 1: Install CLI

```bash
npm install -g openclaw@latest
openclaw --version
```

## Step 2: Run Onboarding Wizard

```bash
openclaw onboard --install-daemon
```

During onboarding, when prompted for model configuration:
- Select **local/self-hosted** model usage (not cloud API).
- Set the Ollama endpoint: `http://localhost:11434`.
- Set the default model name: `qwen2.5:1.5b` (or whichever you pulled).

## Step 3: Run Health Check

```bash
openclaw doctor
```

Checkpoint: doctor should not report blocking errors.

> ### ✅ What Just Happened?
>
> OpenClaw is installed and configured to use your local Ollama instance.
> The daemon is registered with systemd so it can survive reboots (we'll
> harden this further in Part 6).

---

# Part 4: Start Gateway and Validate Local Agent Path

## Step 1: Start Gateway (Foreground Debug Run)

```bash
openclaw gateway --port 18789 --verbose
```

Keep this terminal open. You should see startup logs like:
```
Gateway listening on port 18789
```

## Step 2: Send Local Agent Prompt (Second Terminal)

```bash
openclaw agent --message "Give me a 3-item checklist for backing up a Raspberry Pi."
```

> **Timing expectation:** First request may take 10–30 seconds (cold model load).
> Subsequent requests in the same session should respond in a few seconds.

Checkpoint: response returns successfully and gateway logs show the request flowing through Ollama.

> ### ✅ What Just Happened?
>
> You confirmed the full local path works: OpenClaw CLI → Gateway → Ollama →
> local LLM → response. No cloud APIs involved. Everything runs on the Pi.

---

# Part 5: Connect a Telegram Channel (MVP)

Pick **one** channel first. Telegram is easiest for most users — it's free, has a simple bot API, and works well with OpenClaw.

## Step 1: Create a Telegram Bot

1. Open Telegram and message **[@BotFather](https://t.me/BotFather)**.
2. Send `/newbot`.
3. Choose a display name (e.g., `My Pi Assistant`).
4. Choose a username (must end in `bot`, e.g., `mypi_assistant_bot`).
5. BotFather replies with an **API token** — copy it. It looks like:
   ```
   7123456789:AAH...long-string...
   ```

## Step 2: Configure OpenClaw for Telegram

Add Telegram credentials during onboarding or edit the config directly:

```bash
openclaw onboard
```

When prompted for channels, select **Telegram** and paste your bot token.

## Step 3: Security — Set DM Pairing Policy

OpenClaw treats inbound DMs as untrusted by default. Verify pairing mode is active:

```bash
openclaw doctor
```

The first time you message your bot, it will reply with a **pairing code**. Approve it:

```bash
openclaw pairing approve telegram <code>
```

This adds your Telegram account to the local allowlist.

## Step 4: Test End-to-End

1. Open Telegram and send a message to your bot:
   ```
   What's the weather like? (just testing)
   ```
2. Watch the gateway logs — you should see the message arrive, get routed to Ollama, and a response sent back.
3. Confirm the reply appears in Telegram.

Checkpoint: one clean round-trip: **Telegram → OpenClaw Gateway → Ollama → Telegram**.

> ### ✅ What Just Happened?
>
> You connected a real messaging channel to your Pi-based assistant. Messages
> from Telegram are received by OpenClaw, processed by a local LLM on the Pi,
> and responses are sent back — all without any cloud AI service.

---

# Part 6: Hardening for 24/7 Pi Usage

## Step 1: Ensure OpenClaw Restarts on Crash/Reboot

If the onboarding wizard installed the daemon, verify it:

```bash
systemctl --user status openclaw-gateway
```

If it's not registered, create a user service:

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/openclaw-gateway.service <<'EOF'
[Unit]
Description=OpenClaw Gateway
After=network-online.target ollama.service
Wants=network-online.target

[Service]
ExecStart=/usr/bin/env openclaw gateway --port 18789
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
EOF

systemctl --user daemon-reload
systemctl --user enable --now openclaw-gateway

# Enable user services to start at boot (without login)
sudo loginctl enable-linger $USER
```

## Step 2: Monitor Thermals

```bash
# Current CPU temperature
vcgencmd measure_temp

# Check if throttling has occurred (0x0 = no throttling)
vcgencmd get_throttled
```

Set up a simple cron alert if temperature exceeds 80°C:

```bash
(crontab -l 2>/dev/null; echo '*/5 * * * * temp=$(vcgencmd measure_temp | grep -oP "\d+\.\d+"); echo "$temp" | awk "{if (\$1 > 80) system(\"logger -t pi-thermal WARNING: CPU at \" \$1 \"C\")}"') | crontab -
```

## Step 3: Confirm Memory Headroom

```bash
free -h
htop
```

If memory pressure is high, step down to a smaller model or increase swap (see Part 1 Step 3).

## Step 4: Log Rotation

Prevent logs from filling the SD card:

```bash
# Ollama logs are managed by journald; set a size cap
sudo tee /etc/systemd/journald.conf.d/size-limit.conf > /dev/null <<'EOF'
[Journal]
SystemMaxUse=200M
EOF
sudo systemctl restart systemd-journald
```

> ### ✅ What Just Happened?
>
> Your Pi assistant now auto-starts on boot, auto-restarts on crash, monitors
> thermals, and won't fill up your disk with logs. It's ready for 24/7 operation.

---

# Part 7: Performance Tuning Playbook

Tune in this order — each step gives diminishing returns:

### 1. Model Size (Biggest Impact)

| Model | RAM Usage | Speed (approx) | Quality |
|---|---|---|---|
| qwen2.5:1.5b | ~1.5 GB | 10–15 tok/s | Good for simple tasks |
| qwen2.5:3b | ~2.5 GB | 5–10 tok/s | Better reasoning |
| qwen2.5:7b | ~5 GB | 2–5 tok/s | Best quality, slow on Pi |

Start with 1.5B. Only upgrade after confirming stability.

### 2. Context Window

Shorter conversation history = faster responses. If OpenClaw supports context limits, keep them modest (e.g., 2048 tokens).

### 3. Request Pattern

Serial requests only. The Ollama tuning from Part 2 (`OLLAMA_NUM_PARALLEL=1`) enforces this.

### If Responses Are Too Slow

1. Drop back to a smaller model.
2. Trim conversation context / start fresh sessions.
3. Reduce background assistant activity (cron skills, watchers).
4. Check for thermal throttling (`vcgencmd get_throttled`).

---

# Part 8: Troubleshooting

## Ollama won't start or respond

```bash
sudo systemctl status ollama --no-pager
sudo journalctl -u ollama -n 100 --no-pager

# Check if port is in use
ss -tlnp | grep 11434

# Check disk space (model won't load if disk is full)
df -h
```

## Model fails to load (OOM)

```bash
free -h
# If swap is low, increase it (Part 1 Step 3)
# Or switch to a smaller model:
ollama pull qwen2.5:1.5b
```

## OpenClaw startup / config issues

```bash
openclaw doctor
openclaw gateway --verbose
```

## Gateway starts but channel doesn't respond

- Verify the bot token is correct: re-run `openclaw onboard`.
- Check pairing: `openclaw pairing approve telegram <code>`.
- Check Ollama is reachable: `curl http://localhost:11434/api/tags`.

## High latency / slow responses

- First request is always slow (cold model load) — this is normal.
- Check CPU temp: `vcgencmd measure_temp` (throttling starts ~80°C).
- Switch to smaller model.
- Reduce prompt length.

## Port conflict

```bash
# Check what's using port 18789
ss -tlnp | grep 18789
```

---

# Part 9: Suggested "Next Step" Expansions

1. **Add a second channel** (Discord, WhatsApp) after Telegram is stable.
2. **Add skills/tools** for your daily workflows (reminders, home automation triggers).
3. **Try the "what-changed" sentinel** — watch a folder for config/doc changes and get semantic diffs via your channel.
4. **Remote access** — set up Tailscale or WireGuard so you can reach your Pi assistant from anywhere.
5. **Backup** — regularly back up `~/.openclaw/` and your Ollama models directory.
6. **Upgrade path** — when you outgrow the Pi, move to a mini-PC with the same OpenClaw setup (zero workflow changes).

---

# Part 10: Pi-hole + AI Narrator (Bonus)

**What is Pi-hole?** A network-wide ad blocker that runs on your Pi. It acts as
your home network's DNS server — every device (phones, smart TVs, laptops)
sends DNS queries through it, and Pi-hole silently blocks requests to known
ad and tracking domains. No browser extensions needed, works on every device
automatically.

**What we're adding:** a weekly AI-powered digest that reads Pi-hole's logs and
sends you a plain-language summary via Telegram: which devices are chattiest,
what tracking domains were blocked, and anything suspicious.

```text
Pi-hole (DNS filtering, port 53)
        |
        v
    Query logs (/etc/pihole/)
        |
        v
    Cron script (weekly)
        |
        v
    OpenClaw agent → Ollama → Telegram summary
```

## Step 1: Install Pi-hole

```bash
curl -sSL https://install.pi-hole.net | bash
```

During setup:
- Choose your network interface (usually `eth0` for Ethernet).
- Accept default upstream DNS (e.g., Google, Cloudflare, or pick your own).
- **Enable the web admin interface** (useful for quick checks).
- Note the admin password shown at the end (or set one with `pihole -a -p`).

## Step 2: Point Your Network at Pi-hole

Set your router's DHCP DNS server to the Pi's IP address. Alternatively, set
DNS per-device if you only want to test with one machine first.

Find your Pi's IP:

```bash
hostname -I
```

Verify Pi-hole is working:

```bash
# From any device on your network:
nslookup doubleclick.net <PI_IP>
# Should return 0.0.0.0 (blocked)
```

## Step 3: Confirm Pi-hole Doesn't Interfere with Ollama

Pi-hole uses almost no CPU or RAM — DNS lookups are tiny. But let's verify:

```bash
# Check system load with both running
htop

# Confirm Ollama still responds normally
ollama run qwen2.5:1.5b "Quick test."
```

Optional: lower Ollama's CPU priority so DNS is never starved:

```bash
sudo tee /etc/systemd/system/ollama.service.d/nice.conf > /dev/null <<'EOF'
[Service]
Nice=10
EOF
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

## Step 4: Create the Narrator Script

This script extracts Pi-hole stats and sends them to your OpenClaw assistant
for summarization.

```bash
mkdir -p ~/scripts

cat > ~/scripts/pihole-narrator.sh <<'SCRIPT'
#!/bin/bash
# pihole-narrator.sh — Weekly Pi-hole digest via OpenClaw

# Collect stats from Pi-hole API
STATS=$(curl -s "http://localhost/admin/api.php?summary")

# Extract key numbers
TOTAL_QUERIES=$(echo "$STATS" | jq -r '.dns_queries_today // "unknown"')
BLOCKED=$(echo "$STATS" | jq -r '.ads_blocked_today // "unknown"')
PERCENT=$(echo "$STATS" | jq -r '.ads_percentage_today // "unknown"')
UNIQUE_DOMAINS=$(echo "$STATS" | jq -r '.unique_domains // "unknown"')

# Get top blocked domains (last 7 days from query log)
TOP_BLOCKED=$(sqlite3 /etc/pihole/pihole-FTL.db \
  "SELECT domain, COUNT(*) as cnt FROM queries
   WHERE status IN (1,4,5,9,10,11)
   AND timestamp > strftime('%s','now','-7 days')
   GROUP BY domain ORDER BY cnt DESC LIMIT 10;" 2>/dev/null \
  || echo "Could not read query database")

# Get top client devices
TOP_CLIENTS=$(sqlite3 /etc/pihole/pihole-FTL.db \
  "SELECT client, COUNT(*) as cnt FROM queries
   WHERE timestamp > strftime('%s','now','-7 days')
   GROUP BY client ORDER BY cnt DESC LIMIT 5;" 2>/dev/null \
  || echo "Could not read query database")

# Build the prompt
PROMPT="You are a network security narrator. Summarize this Pi-hole weekly report in a friendly, clear way. Highlight anything unusual or concerning. Keep it under 15 lines.

Stats today:
- Total DNS queries: $TOTAL_QUERIES
- Ads/trackers blocked: $BLOCKED ($PERCENT%)
- Unique domains seen: $UNIQUE_DOMAINS

Top 10 blocked domains (last 7 days):
$TOP_BLOCKED

Top 5 devices by query volume:
$TOP_CLIENTS"

# Send to OpenClaw agent (which routes to local Ollama)
openclaw agent --message "$PROMPT"

SCRIPT

chmod +x ~/scripts/pihole-narrator.sh
```

## Step 5: Test the Narrator

```bash
~/scripts/pihole-narrator.sh
```

You should get a friendly summary like:

> *Your network made 12,400 DNS queries today. Pi-hole blocked 2,100 of them
> (17%). Your Samsung TV was the chattiest device with 3,200 queries — 890
> went to tracking domains. The most-blocked domain was `ads.doubleclick.net`
> (340 hits). Nothing unusual this week.*

## Step 6: Schedule Weekly Reports

```bash
# Send digest every Sunday at 9am
(crontab -l 2>/dev/null; echo '0 9 * * 0 /home/$USER/scripts/pihole-narrator.sh') | crontab -
```

## Step 7: Optional — Daily Security Alerts

For a lighter daily check that only alerts on suspicious activity:

```bash
cat > ~/scripts/pihole-alert.sh <<'SCRIPT'
#!/bin/bash
# Quick daily check for suspicious domains

SUSPICIOUS=$(sqlite3 /etc/pihole/pihole-FTL.db \
  "SELECT domain FROM queries
   WHERE status NOT IN (1,4,5,9,10,11)
   AND timestamp > strftime('%s','now','-1 day')
   AND (domain LIKE '%.xyz' OR domain LIKE '%.top' OR domain LIKE '%.tk'
        OR domain LIKE '%crypto%' OR domain LIKE '%malware%')
   GROUP BY domain LIMIT 20;" 2>/dev/null)

if [ -n "$SUSPICIOUS" ]; then
  openclaw agent --message "SECURITY ALERT: These suspicious domains were queried from your network in the last 24 hours. Should I be concerned? Domains: $SUSPICIOUS"
fi

SCRIPT

chmod +x ~/scripts/pihole-alert.sh

# Run daily at 8am
(crontab -l 2>/dev/null; echo '0 8 * * * /home/$USER/scripts/pihole-alert.sh') | crontab -
```

> ### ✅ What Just Happened?
>
> You turned your Pi into a **network security narrator**. Pi-hole silently
> blocks ads and trackers across every device on your network. Your local AI
> reads those logs and sends you plain-language weekly summaries via Telegram —
> no data ever leaves your home. You'll know which devices are chattiest, what
> tracking domains they're hitting, and whether anything suspicious appeared.
> All running on a single $80 Raspberry Pi.

---

## Quick Validation Checklist

- [ ] Pi 5 running 64-bit OS (`uname -m` → `aarch64`)
- [ ] Swap configured (2GB+ visible in `free -h`)
- [ ] `node -v` is 22+
- [ ] Ollama service active (`systemctl status ollama`)
- [ ] Ollama tuned for Pi (`OLLAMA_NUM_PARALLEL=1`)
- [ ] `ollama run` works with starter model
- [ ] Tokens/sec baseline measured
- [ ] `openclaw onboard --install-daemon` completed
- [ ] `openclaw doctor` has no blocking errors
- [ ] `openclaw agent --message ...` returns a response
- [ ] Telegram bot created and token configured
- [ ] DM pairing approved
- [ ] One channel round-trip works end-to-end
- [ ] OpenClaw gateway auto-starts on reboot
- [ ] Pi-hole installed and blocking ads (`nslookup doubleclick.net` → `0.0.0.0`)
- [ ] Pi-hole narrator script returns a summary
- [ ] Weekly cron job scheduled

---

## References

- [OpenClaw repository](https://github.com/openclaw/openclaw)
- [OpenClaw docs](https://docs.openclaw.ai)
- [OpenClaw getting started](https://docs.openclaw.ai/start/getting-started)
- [OpenClaw onboarding wizard](https://docs.openclaw.ai/start/wizard)
- [OpenClaw security guide](https://docs.openclaw.ai/gateway/security)
- [Ollama](https://ollama.com)
- [Pi-hole](https://pi-hole.net)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
