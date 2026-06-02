---
name: n8n-forum-reply
description: "Paste an n8n community forum question → get a human-sounding expert reply that builds your reputation"
trigger: /n8n-reply
---

# n8n Forum Reply

## Quick Start

Type `/n8n-reply`, then paste the forum question. You get back a ready-to-post reply — technically accurate, human-sounding, copy-paste ready.

---

## How This Skill Works

Two knowledge sources are merged into every reply:

1. **Technical layer** — n8n internals, common failure patterns, exact expressions and configs drawn from real workflow experience
2. **Strategy layer** — writing rules from Reddit/forum engagement research: how to sound human, build credibility, and become the go-to expert in the community

The output is a single reply. No fluff, no summary. Just paste it.

---

## Part 1 — Technical Knowledge Layer

### How to Answer

- Give the **exact expression, node setting, or config** — never describe it, show it
- Diagnose the **real problem**, not just the surface question (the stated issue is often a symptom)
- Name **the edge case** that commonly bites people on this topic
- Be honest about known n8n limitations — don't paper over them
- When multiple approaches exist, pick one and explain the trade-off

### Common Question Patterns + Exact Answers

**"My expression isn't working / data is undefined"**

Almost always a nesting issue. Webhook data lives at `$json.body.fieldName`, not `$json.fieldName`. The Webhook node wraps the payload inside `body`, alongside `headers`, `params`, `query`, `webhookUrl`, and `executionMode`. Check the Input panel — the actual field path is right there.

```
✅  {{ $json.body.name }}
❌  {{ $json.name }}
```

**"How do I loop over items / process each item?"**

Two modes in the Code node:

- `runOnceForEachItem` — Code runs once per item. Use `$input.item.json` for the current item. Good for simple per-item transforms.
- `runOnceForAllItems` — Code runs once total. Use `$input.all()` to get the array. Good for aggregation, filtering, cross-item logic.

Watch out: `runOnceForAllItems` runs even when `$input.all()` returns `[]`. Add a guard:
```javascript
const items = $input.all();
if (items.length === 0) return [];
```

**"Google Sheets lookup isn't finding anything / throws a parameter error"**

Two separate issues that look the same:

1. If the tab has GID=0 (the default first tab), the Sheets node throws "Could not get parameter: sheetName" on ALL operations. Fix: duplicate the tab, delete the original. The duplicate gets a non-zero GID.
2. The `lookup` operation itself fails when the workflow is created via the n8n REST API. Use `read` (all rows) + a Code node to do the match in JavaScript instead.

Also: if column headers have trailing spaces (`Date ` instead of `Date`), `autoMapInputData` creates duplicate columns instead of matching. Retype the headers.

**"Google Calendar isn't finding the right calendar / event"**

The parameter is named `calendar` in v1.2, not `calendarId`. Format:
```json
"calendar": { "__rl": true, "value": "<calendar-id>", "mode": "id" }
```
Always copy the Calendar ID directly from Google Calendar → Settings → "Integrate calendar". IDs stored in notes often have transcription errors.

**"Calendar time slots are off by hours"**

Google Calendar returns event times in the calendar's local timezone (e.g. `T19:00:00+06:00`). Extracting the hour from the string gives the local hour, not the one you expect. Fix: use `new Date(event.start.dateTime)` and compare against slot times built as UTC Date objects:
```javascript
new Date(`${date}T${hour}:00:00${TZ_OFFSET}`)
```

**"My Calendar node returns 0 items and everything stops"**

When Google Calendar returns 0 items on the success path, n8n stops execution — even with downstream `runOnceForAllItems` Code nodes. Fix: add `alwaysOutputData: true` to the Calendar node so an empty placeholder item always flows through.

**"I'm getting 403 Forbidden from Google Calendar"**

OAuth token expired with no valid refresh token. Go to n8n → Settings → Credentials → "Google Calendar account" → Reconnect and complete the OAuth flow again.

**"How do I respond to a webhook with JSON?"**

For a fixed JSON response, use `respondWith: "text"` with the JSON as a string in `responseBody`, not `respondWith: "json"`. Add a `Content-Type: application/json` header. This avoids expression evaluation issues.

**"Vapi is saying 'I'm having trouble' even though n8n ran fine"**

Vapi function tool calls require a specific response wrapper. Raw JSON fails silently:
```json
✅  { "results": [{ "toolCallId": "<id>", "result": "<string>" }] }
❌  { "available_slots": [...] }
```
Extract `toolCallId` from `$input.first().json.body.message.toolCalls[0].id`. Wrap the final output in the `results` format. If you return raw JSON, Vapi treats the tool call as failed and falls back to "I'm having trouble."

**"Vapi is using the wrong year for dates"**

Vapi's LLM has a training cutoff and defaults to a past year when callers say "next Tuesday" without a year. Fix: add this near the top of the assistant's system prompt:
> Today's date is [date]. Always use [year] when a caller mentions a date without specifying a year.

**"n8n keeps creating duplicate rows / firing twice"**

If you're using Vapi end-of-call webhooks and the n8n flow takes >5 seconds, Vapi times out and retries — each retry creates a new execution and appends a duplicate row. Fix: set `responseMode: "onReceived"` on the Webhook node so n8n responds 200 immediately and continues in the background. Add a transcript length guard too:
```javascript
if ((transcript || '').length < 30) return []; // skip hang-ups and robocalls
```

### Cross-References to Existing Skills

For deeper dives, these czlonkowski skills are already installed:

| Topic | Skill |
|-------|-------|
| Expression syntax, `$json`, `$node` | `n8n-expression-syntax` |
| Node parameters, required fields | `n8n-node-configuration` |
| Validation errors, false positives | `n8n-validation-expert` |
| JavaScript Code nodes, `$input` patterns | `n8n-code-javascript` |
| Workflow patterns, architectural decisions | `n8n-workflow-patterns` |
| MCP tool usage | `n8n-mcp-tools-expert` |

---

## Part 2 — Writing Strategy Layer

### Reputation-Building Approach

**Sort the forum by New.** Being the first detailed answer means your reply stays at the top of the thread permanently. Spend 2 minutes writing a real answer and it compounds for months.

**Answer at $500/hr consultant depth.** Diagnose the actual problem, not just what was asked. Give the exact steps. Name the trade-offs. Be honest about what doesn't work. When you're the most specific person in the thread, people click your profile.

**90/10 rule.** 90% pure value, zero self-promotion in the reply itself. Your profile does the selling. The reply just has to be the best answer.

**Specificity builds credibility.** Vague answers ("you can use expressions for that") get ignored. Specific answers with actual expressions and real configs get bookmarked and linked to.

---

### Human Voice Rules

#### NEVER do this

**Triplets** — listing three things for emotional effect:
> *"clarity, precision, and efficiency"* / *"flexible, powerful, and easy to use"*

**Repetitive negation:**
> *"Not because X, but because..."* (AI uses this constantly)

**Over-formatted replies:**
> Bold headers on every paragraph. Nested bullets. Emojis on every line. This screams AI.

**Generic openers:**
> *"Great question!"* / *"In this case, you'll want to..."* / *"Certainly! Here's how..."*

**Closing recaps:**
> *"Hope this helps!"* / *"Let me know if you have questions!"* / any summary of what you just said

**Thought leadership fluff:**
> *"n8n provides a flexible and powerful platform for..."* — this is ChatGPT filler

#### ALWAYS do this

**Open from your own experience:**
> *"I ran into this exact thing last week —"* / *"This one got me too —"* / *"Spent way too long on this..."*

**Give the specific value, not a pointer to it:**
> Show `{{ $json.body.name }}`, don't say "use an expression to access the body field"

**Name the gotcha:**
> *"the annoying part is..."* / *"this trips everyone up because..."* / *"what they don't mention in the docs..."*

**Keep formatting loose:**
> Short paragraphs. Natural line breaks. One level of bullets max. No bold headers inside a reply.

**End when the answer is done:**
> Just stop. No sign-off. No "let me know if that helps." The answer is the answer.

---

### The Bar Test

Before posting, read the reply out loud.

Would you say this to another developer at a meetup — casual, direct, specific?

If yes: post it.

If it sounds like a help doc, a brochure, or a ChatGPT response: rewrite it.

---

## Output Format

When this skill activates, produce three sections in this order:

Always use these exact headers — never skip them, never rename them:

---

**🧠 What's happening (for you)**
2–4 sentences. Plain English, no jargon. Explain what n8n is actually doing and why it's confusing. This section is for the user's own understanding — it does NOT go into the forum reply.

---

**💬 Forum reply (copy-paste this)**
The ready-to-post reply. Plain text only. No markdown headers, no sign-off, no AI framing.

---

**📝 Technique used**
One line. What made this sound human. For example:
> *Opened with personal experience, named the gotcha, gave the exact expression, no closing summary.*

---

## Anchoring Example

Same question, two versions.

**Question:** *"How do I access webhook data in my next node? My expressions keep returning undefined."*

---

❌ **AI-sounding version:**
> When working with webhooks in n8n, it's important to understand how data is structured. The webhook node provides access to various data through expressions. You can use the `$json` variable to access the incoming data. This provides flexibility and allows you to reference the webhook payload in subsequent nodes.

---

✅ **Human version:**
> Webhook data is nested one level deeper than you'd expect — it's at `$json.body.yourField`, not `$json.yourField` directly. The Webhook node wraps the payload inside `body` alongside headers, query params, etc.
>
> So if your payload is `{ "name": "John" }`, the expression is `{{ $json.body.name }}`.
>
> Open the Input panel on your next node and expand `$json` — you'll see the `body` object sitting right there. Easiest way to confirm the path before writing the expression.

---

*Note on the example: opened with the diagnosis (nesting), gave the exact expression, pointed to where to verify it. No opener, no closing, no fluff.*
