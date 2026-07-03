# The Prompts 🧠

Every prompt from the build, exactly as it runs in production. There is only one AI call in the whole system — the classification step — and that's on purpose: one well-tuned prompt beats five mediocre ones.

## The classification prompt

This lives in the **"Ollama Classify"** node of `workflow-email-agent.json` (an HTTP Request node that POSTs to Ollama's `/api/generate`). The static instruction, followed by the live email fields that n8n injects:

```text
You are an email triage assistant. Classify the following email into exactly one category: "urgent" (time-sensitive, blocking, financial or critical issues), "reply-needed" (a human should respond soon), or "ignore" (newsletters, promotions, automated notifications). Respond with STRICT JSON only, exactly this shape: {"category": "urgent|reply-needed|ignore", "summary": "one line summary", "suggested_reply": "2-3 sentence reply, or null if none needed" }.

FROM: <email "from" field>
SUBJECT: <email subject>
BODY:
<first 3,000 characters of the email body>
```

In the node, the email fields are appended with n8n expressions:

```js
'...prompt text...'
  + '\n\nFROM: '    + String($json.from || '')
  + '\nSUBJECT: '   + String($json.subject || '')
  + '\nBODY:\n'     + String($json.textPlain || $json.textHtml || '').slice(0, 3000)
```

The body is truncated to 3,000 characters — plenty for triage, and it keeps the small model fast and focused.

## The output schema

The model must return **strict JSON** in exactly this shape (Ollama's `format: "json"` mode enforces valid JSON):

| Field | Type | Meaning |
|---|---|---|
| `category` | `"urgent"` \| `"reply-needed"` \| `"ignore"` | `urgent` = time-sensitive, blocking, financial or critical. `reply-needed` = a human should respond soon. `ignore` = newsletters, promotions, automated notifications. |
| `summary` | string | One-line, plain-English summary of the email. |
| `suggested_reply` | string \| `null` | A 2–3 sentence draft reply, or `null` if no reply is needed. **Never auto-sent** — it appears in your digest for a human to review. |

The **"Parse & Validate"** node downstream is the safety net: if the model ever returns a category outside the allowed three, it defaults to `reply-needed` — the agent fails toward *showing you* an email, never toward silently ignoring one.

## Model & settings

| Setting | Value | Why |
|---|---|---|
| Model | `qwen2.5:3b-instruct` | Small, free, runs locally in ~2 GB RAM, and reading email is not rocket science. |
| Temperature | `0.2` | Triage is a classification job — low temperature keeps it consistent and boring (boring is good). |
| `format` | `json` | Ollama's JSON mode — guarantees parseable output. |
| `stream` | `false` | We want one complete response, not tokens. |
| HTTP timeout | 180 s | First call after a cold start loads the model into memory — give it room. |

## Tuning lessons (learned the hard way)

**Newsletters never need replies.** The single biggest early failure: the model kept drafting polite little replies to newsletters and promo blasts. Nobody replies to a newsletter. The fix is baked directly into the prompt above — `"ignore" (newsletters, promotions, automated notifications)` names them explicitly as the ignore category, and `suggested_reply` is allowed to be `null`. After that one change, the noise disappeared from the digest.

**That's the real workflow with these agents.** They're not magic — they're trainable. When yours misclassifies something (mine once tagged a payment reminder as low priority when I'd call it urgent), you don't retrain a model or write code. You add one plain-English sentence to the prompt — e.g. *"anything involving money owed is urgent"* — and the fix sticks for every email after that. Correct it once, in plain English, and move on.

**Tune it to your life.** The categories in this prompt are the ones that matter to me. Edit the parentheticals — that's where the behavior lives. Examples: *"emails from my accountant are always urgent"*, *"anything mentioning invoices needs a reply"*, *"GitHub notifications are ignore"*.
