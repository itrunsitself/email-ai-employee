# Guide: Gmail App Password + IMAP Credential 📧

The email agent watches an inbox over **IMAP**, and Gmail requires an **App Password** for that (your normal password won't work with IMAP clients). This takes about five minutes — it's the fiddliest part of the whole build, so here it is step by step.

## Step 0 — Use a dedicated Gmail account (recommended)

Create a **separate Gmail account just for the agent** rather than pointing it at your personal inbox, at least to start:

- You can forward only the mail you want triaged (or test with hand-sent emails) while you build trust in it.
- The App Password grants full mailbox access — nicer to scope that to a fresh account.
- If you ever want to reset, you delete one credential, not untangle your personal account.

Once it's earned your trust, pointing it at your real inbox is a two-minute credential swap.

## Step 1 — Enable 2-Step Verification

App Passwords only exist on accounts with 2-Step Verification turned on.

1. Go to [myaccount.google.com/security](https://myaccount.google.com/security)
2. Under **"How you sign in to Google"**, click **2-Step Verification**
3. Follow the setup (phone number is the quickest path) and finish the confirmation

## Step 2 — Create the App Password

1. Go to [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
   (If that page 404s, 2-Step Verification isn't fully enabled yet — finish Step 1 first.)
2. Give it a name like `n8n email agent`
3. Click **Create** — Google shows you a **16-character password**. Copy it now; it's shown exactly once.
4. Spaces in the displayed password are cosmetic — paste it with or without them.

## Step 3 — Make sure IMAP is enabled

In Gmail: **Settings (gear) → See all settings → Forwarding and POP/IMAP → Enable IMAP → Save**. (Newer Gmail accounts have IMAP on by default, but check.)

## Step 4 — Create the IMAP credential in n8n

In n8n: **Credentials → Add credential → IMAP**, then fill in:

| Field | Value |
|---|---|
| User | your full address, e.g. `you@example.com` |
| Password | the 16-character App Password from Step 2 |
| Host | `imap.gmail.com` |
| Port | `993` |
| SSL/TLS | **On** |

Save — n8n tests the connection and you should see a green check.

## Step 5 — Attach it to the workflow

Open the imported **Email Agent (real-time)** workflow, click the **"Email Trigger (IMAP)"** node (it carries a sticky note reminding you), and select your new IMAP credential. That's the agent's ears connected.

## Privacy note 🔒

This is the part of the build I like most: because the AI is a **local model running on your own machine** (Ollama), your mail content **never leaves your computer**. No email body is ever sent to a cloud AI API — the only thing that goes out is the Telegram message *you* configured, containing the summary the local model wrote.
