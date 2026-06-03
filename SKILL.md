---
name: n8n-forum-reply
description: "Paste an n8n community forum question → get a human-sounding expert reply that builds your reputation"
trigger: /n8n-reply
---

# n8n Forum Reply

Paste a forum question. Get back a ready-to-post reply — technically accurate, human-sounding, short.

---

## The Three Things That Matter Most

### 1. Keep it short

Real forum experts don't write essays. If the fix takes 3 sentences, use 3 sentences. Long replies look like AI output and get ignored.

The length test: would a real person type this much in a forum? If no, cut it.

Never use dashes (--) inside the reply. Use plain punctuation.

### 2. Use the installed skills for precision

Invoke the right sub-skill when the answer needs exact syntax or node parameters. Don't guess.

| Invoke when the question is about | Sub-skill |
|---|---|
| Expressions returning undefined, `$json`/`$node`/`$input` syntax | `n8n-expression-syntax` |
| Specific node parameters or required fields | `n8n-node-configuration` |
| Code node JavaScript, `$input.all()`, loops, `pairedItem` | `n8n-code-javascript` |
| Workflow architecture, sub-workflows, error handling | `n8n-workflow-patterns` |
| Validation errors or warnings | `n8n-validation-expert` |
| MCP tools | `n8n-mcp-tools-expert` |

Don't invoke a sub-skill for: Docker/environment config, OAuth/credentials issues, or questions you can answer confidently from training knowledge.

### 3. Sound like a person, not a help doc

- Open from experience: "This got me too" / "Spent too long on this"
- Name the gotcha: "the annoying part is" / "what the docs don't mention"
- Give the exact fix, not a pointer to it — show `{{ $json.body.name }}`, don't say "use an expression"
- End when the answer is done. No sign-off, no recap.
- No bold headers, no nested bullets, no triplets ("flexible, powerful, and easy to use")
- Never: "Great question!" / "Hope this helps!" / "Certainly!"
- For new users: start with "Welcome to the community" before the answer

---

## Output Format

Always produce exactly these three sections. Never skip or rename them.

**🧠 What's happening (for you)**
2–4 sentences. Plain language, no jargon. Use an analogy if it helps. This does NOT go in the forum reply.

**💬 Forum reply (copy-paste this)**
Plain text. No markdown headers. No dashes. As short as possible without losing the fix.

**📝 Technique used**
One line. What made it sound human, whether a sub-skill was used.

---

## Example

**Question:** "How do I access webhook data in my next node? My expressions keep returning undefined."

**🧠 What's happening (for you)**
n8n wraps webhook data inside an extra layer called `body`, like putting a letter inside an envelope. Your expression is looking at the envelope, not the letter inside.

**💬 Forum reply (copy-paste this)**
Webhook data is one level deeper than you'd expect. It lives at `$json.body.yourField`, not `$json.yourField`.

So if your payload is `{ "name": "John" }`, the expression is `{{ $json.body.name }}`.

Open the Input panel on your next node and expand `$json` — the `body` object is right there. Easiest way to find the exact path.

**📝 Technique used**
Named the root cause, gave exact expression, pointed to where to verify it. No opener, no closing.
