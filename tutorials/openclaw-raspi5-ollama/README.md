# OpenClaw + Raspberry Pi 5 + Ollama: A Practical Local Assistant Tutorial

> **Run a personal AI assistant on Raspberry Pi 5 with OpenClaw as the control plane and Ollama as the local model runtime.**

---

## Who Is This For?

You want a local-first assistant and you are okay with Raspberry Pi limits (CPU-only inference, smaller models, slower responses than desktop GPUs).

**What we're building:** an always-on OpenClaw gateway on Pi 5, backed by local Ollama models, with one messaging channel connected end-to-end.

---

## Feasibility (Short Answer)

Yes, this is feasible on Pi 5 and works well for lightweight assistant tasks, especially with small quantized models.

### What to Expect on Pi 5

| Area | Realistic Expectation |
|---|---|
| Model size | Best with 1.5B–3B models; 7B is possible but slower |
| Latency | Good for short prompts; long contexts get noticeably slower |
| Hardware | Active cooling is strongly recommended |
| Stability | Good if you limit concurrency and keep context size moderate |

---

## Prerequisites

| Requirement | Suggested Version | Notes |
|---|---|---|
| Raspberry Pi 5 | 8GB+ preferred | 4GB can work with stricter limits |
| OS | Raspberry Pi OS 64-bit (Bookworm) | Keep system fully updated |
| Node.js | 22+ | OpenClaw runtime requirement |
| npm/pnpm | latest | Needed for OpenClaw install |
| Ollama | latest | Local model runtime |
| Network | stable LAN/Wi-Fi | Required for onboarding and channel setup |

---

## Part 0: Architecture You’re Building

```text
Messaging Channel (Telegram/Discord/etc)
                |
                v
          OpenClaw Gateway (Pi 5)
                |
                v
          Ollama local model(s)
```

OpenClaw manages channels, routing, tools, and session behavior. Ollama runs the LLM locally.

---

# Part 1: Prepare Raspberry Pi 5

## Step 1: Update System

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

## Step 2: Install Core Utilities

```bash
sudo apt install -y curl git build-essential jq htop
```

## Step 3: Install Node.js 22+

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

Checkpoint: Node major version should be `v22` or newer.

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

## Step 3: Pull a Starter Model

Start small first:

```bash
ollama pull qwen2.5:1.5b
```

Optional upgrade after baseline is stable:

```bash
ollama pull qwen2.5:3b
```

## Step 4: Sanity Test

```bash
ollama run qwen2.5:1.5b "Say hello in one short sentence."
```

Checkpoint: the model returns a response reliably without crashing.

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

During onboarding:
- Select local model usage.
- Choose Ollama as the provider/runtime when prompted.
- Keep defaults conservative for first run.

## Step 3: Run Health Check

```bash
openclaw doctor
```

Checkpoint: doctor should not report blocking errors.

---

# Part 4: Start Gateway and Validate Local Agent Path

## Step 1: Start Gateway (Foreground Debug Run)

```bash
openclaw gateway --port 18789 --verbose
```

Keep this terminal open for now.

## Step 2: Send Local Agent Prompt (Second Terminal)

```bash
openclaw agent --message "Give me a 3-item checklist for backing up a Raspberry Pi."
```

Checkpoint: response returns successfully and gateway logs show the request path.

---

# Part 5: Connect One Channel Only (MVP)

Do one channel first (Telegram or Discord is easiest for most users).

1. Configure channel credentials in onboarding/config.
2. Send a test message to your assistant.
3. Confirm the reply appears in that same channel.

Checkpoint: one clean round-trip from channel -> OpenClaw -> Ollama -> channel.

---

# Part 6: Hardening for 24/7 Pi Usage

## Step 1: Keep Thermal Headroom

- Use active cooling (fan + heatsink).
- Avoid fully saturating the CPU for long periods.

Useful commands:

```bash
vcgencmd measure_temp
vcgencmd get_throttled
```

## Step 2: Confirm Memory Headroom

```bash
free -h
htop
```

If memory pressure is high, step down to a smaller model.

## Step 3: Reduce Concurrency and Prompt Size

- Keep to one active channel while validating.
- Keep prompts short and practical.
- Avoid large context usage until baseline is stable.

---

# Part 7: Performance Tuning Playbook

Start from this order:

1. **Model size first**: 1.5B -> 3B only if stable.
2. **Prompt size second**: shorter prompts and smaller history windows.
3. **Load pattern third**: serial requests over parallel fan-out.

If responses become too slow:
- drop back to smaller model,
- trim conversation context,
- reduce background assistant activity.

---

# Part 8: Troubleshooting

## Ollama fails to respond

```bash
sudo systemctl status ollama --no-pager
sudo journalctl -u ollama -n 100 --no-pager
```

## OpenClaw startup/config issues

```bash
openclaw doctor
openclaw gateway --verbose
```

## High latency

- Switch to smaller model (e.g., 1.5B).
- Reduce prompt size.
- Check CPU temp and throttling.

## Memory/OOM symptoms

- Move down a model tier.
- Restart long-running services.
- Keep only one channel active until stable.

---

# Part 9: Suggested “Next Step” Expansions

1. Add a second channel after the first is stable.
2. Add specific skills/tools for your daily workflows.
3. Add remote admin and backups of OpenClaw config/workspace.
4. Move to a larger host later while keeping the same OpenClaw workflow.

---

## Quick Validation Checklist

- [ ] Pi fully updated and stable after reboot
- [ ] `node -v` is 22+
- [ ] Ollama service active
- [ ] `ollama run` works with starter model
- [ ] `openclaw onboard --install-daemon` completed
- [ ] `openclaw doctor` has no blocking errors
- [ ] `openclaw agent --message ...` returns a response
- [ ] One channel round-trip works end-to-end

---

## References

- [OpenClaw repository](https://github.com/openclaw/openclaw)
- [OpenClaw docs](https://docs.openclaw.ai)
- [OpenClaw getting started](https://docs.openclaw.ai/start/getting-started)
- [OpenClaw onboarding wizard](https://docs.openclaw.ai/start/wizard)
- [Ollama](https://ollama.com)
