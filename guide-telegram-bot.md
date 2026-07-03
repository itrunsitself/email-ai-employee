# Guide: Telegram Bot + Chat ID 📱

Delivery is Telegram because it's free and it actually pushes to your phone. You need two things: a **bot token** and your **chat ID**. Three minutes, tops.

## Step 1 — Create the bot with @BotFather

1. In Telegram, search for **@BotFather** (the verified one — it's Telegram's official bot-creating bot)
2. Send: `/newbot`
3. Give it a display name — e.g. `Inbox Agent`
4. Give it a username ending in `bot` — e.g. `my_inbox_agent_bot`
5. BotFather replies with your **bot token**, which looks like:
   `1234567890:AAExAmPlEt0kEnDoNoTsHaReThIsAnYwHeRe`

**Treat the token like a password.** Anyone who has it can send messages as your bot.

## Step 2 — Get your chat ID

The bot can only message people who message it first, so:

1. Open your new bot in Telegram (BotFather's reply links to it) and send it any message — `hi` works
2. In a browser, open (with your real token substituted):

   ```text
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```

3. In the JSON response, find `"chat": {"id": ...}` — that number is your **chat ID**:

   ```json
   {"ok":true,"result":[{"message":{"chat":{"id":123456789, "first_name":"You", ...}}}]}
   ```

   (If `result` is empty, send the bot another message and refresh the page.)

## Step 3 — Create the Telegram credential in n8n

In n8n: **Credentials → Add credential → Telegram API**, paste the bot token, save. Green check means you're connected.

## Step 4 — Attach it in both workflows

There are **two Telegram nodes** across the two workflows, and both need the credential *and* your chat ID:

| Workflow | Node | What it sends |
|---|---|---|
| Email Agent (real-time) | **Telegram Urgent Alert** | Instant ping when an email is classified `urgent` |
| Morning Digest | **Telegram Digest** | The daily summary of everything processed |

For each node: select your Telegram credential, then replace the placeholder `YOUR_CHAT_ID` in the **Chat ID** field with the number from Step 2.

## Step 5 — Test it

Open the **Morning Digest** workflow and hit **Execute workflow**. Your phone should buzz within a couple of seconds (if no emails have been processed yet, the digest politely says so — that still proves the pipe works).
