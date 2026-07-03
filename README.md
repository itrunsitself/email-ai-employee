# The Free AI Email Employee 📬🤖

This is the **exact build** from the video — a free AI employee that reads your email while you sleep, classifies everything with a local AI model, pings your phone the moment something urgent lands, and hands you a morning digest with suggested replies already written.

▶️ **Watch the build:** [VIDEO_URL]

Everything runs on your own machine: **n8n** for the automation, **Ollama** running `qwen2.5:3b-instruct` for the brain. No subscriptions, no API keys, and your mail content never leaves your computer.

## How it works

Two small workflows — and that separation is the trick most tutorials skip:

1. **Email Agent (real-time)** — an IMAP trigger watches your inbox 24/7. Every new email goes to the local AI, which returns strict JSON: a category (`urgent` / `reply-needed` / `ignore`), a one-line summary, and a suggested reply. Everything is appended to a log file. If it's urgent, you get a Telegram alert immediately.
2. **Morning Digest** — once a day, a schedule trigger reads the log, builds one clean message (counts at the top, every email one line, suggested replies underneath), sends it to your Telegram, and archives the log so tomorrow starts fresh.

## Quick start (5 steps)

1. **Get n8n + Ollama running** — follow [`docker-n8n.md`](docker-n8n.md). One `docker run` for n8n, one install line for Ollama, one `ollama pull` for the model.
2. **Set up the email account** — follow [`guide-gmail-app-password.md`](guide-gmail-app-password.md) to create a Gmail App Password and the n8n IMAP credential. (Strongly recommended: a dedicated Gmail account for the agent, not your personal one.)
3. **Create your Telegram bot** — follow [`guide-telegram-bot.md`](guide-telegram-bot.md) to get a bot token from @BotFather and find your chat ID.
4. **Import the workflows** — in n8n: *Workflows → Import from File*, once for [`workflow-email-agent.json`](workflow-email-agent.json) and once for [`workflow-morning-digest.json`](workflow-morning-digest.json). Attach your credentials (each node that needs one carries a note telling you which), and replace `YOUR_CHAT_ID` in the two Telegram nodes with your real chat ID.
5. **Test, then activate** — send yourself a test email with "URGENT" in the subject, watch the execution light up green, then flip both workflows to **Active**. Done.

> **One config note:** the workflows call Ollama at `http://172.17.0.1:11434` — that's the Docker bridge address, so n8n running *inside* Docker can reach Ollama running on the host. If your n8n is **not** in Docker, change the URL in the "Ollama Classify" node to `http://localhost:11434`. Details in [`docker-n8n.md`](docker-n8n.md).

> **Digest time:** the schedule node ships as `0 10 * * *` (10:00 UTC). Edit the cron expression — or set `GENERIC_TIMEZONE` in Docker — so the digest lands at *your* 7 AM.

## What's in this repo

| File | What it is |
|---|---|
| [`workflow-email-agent.json`](workflow-email-agent.json) | The real-time agent: IMAP trigger → AI classify → parse → log + urgent Telegram alert |
| [`workflow-morning-digest.json`](workflow-morning-digest.json) | The daily digest: schedule → read log → build message → Telegram → archive log |
| [`prompts.md`](prompts.md) | The exact classification prompt, the output JSON schema, model settings, and the tuning lessons |
| [`guide-gmail-app-password.md`](guide-gmail-app-password.md) | Gmail App Password + IMAP credential, step by step |
| [`guide-telegram-bot.md`](guide-telegram-bot.md) | Telegram bot token + chat ID, step by step |
| [`docker-n8n.md`](docker-n8n.md) | The exact Docker and Ollama commands, with explanations |
| [`LICENSE`](LICENSE) | MIT — use it, modify it, ship it |

## Requirements

- **Docker** (for n8n) — any machine that can stay on: an old laptop, a Raspberry Pi-class box, or a ~$5/month VPS
- **~2 GB of RAM** free for the `qwen2.5:3b-instruct` model (Ollama)
- A **Gmail account** for the agent (dedicated account recommended) with 2-Step Verification, for the App Password
- A free **Telegram** account for alerts and the digest

## The most important design decision 🧑‍⚖️

**This agent never sends email.** The AI *drafts* suggested replies into your digest and alerts — a human always reviews, edits, and sends. Rule number one of email automation: the human presses send. Always. The AI does 90% of the work; you do the 10% that protects your reputation.

## Transparency

A few honest things before you build, same as in the video:

- **It's free, but not effortless** — n8n and Ollama cost nothing, but your machine has to stay on (or you run it on a cheap server).
- **The AI will occasionally misjudge an email.** That's exactly why it drafts and never auto-sends. When it gets one wrong, you fix it with one plain-English sentence in the prompt — see [`prompts.md`](prompts.md).
- **Your mail stays local.** The model runs on your own hardware; email content is never sent to any third-party AI API.
- This repo is the exact production workflow from the video, sanitized only to remove personal credentials and IDs.

## License

MIT — see [`LICENSE`](LICENSE). Build on it, remix it, teach with it.

---

Set it once — it runs itself. 🔁
