# LLM06: HITL Approval Gate

**OWASP Risk:** LLM06:2025 Excessive Agency  
**Source:** [OWASP LLM Top 10 v2.0](../../reference/owasp-llm-top10-v2.0.md)  
**Complexity:** Medium

---

## What it does

Enforces a human-in-the-loop approval step for high-impact agentic actions before they execute. An AI agent submits an action request via webhook. A Code node classifies the action as low or high severity based on the action type. Low-impact actions (reads, searches, queries) execute immediately. High-impact actions (deletes, writes, sends, submissions) pause and send an approval request to a Slack channel. A human approves or declines using Slack buttons. The workflow resumes with the decision and either executes the action or returns a 403.

This pattern enforces the principle from the [LLM06 prompt pattern](../../prompt-library/llm06-minimal-agency.md): confirmation for irreversible actions must be enforced at the application layer, not by prompt instructions the model can be argued out of.

---

## Who it's for

Teams building AI agents that take real-world actions: sending emails, modifying records, calling APIs, submitting forms. Any deployment where an injected or misinterpreted prompt could trigger an irreversible action that should require human sign-off.

Intermediate n8n users. Requires a Slack workspace with an n8n OAuth2 app configured.

---

## Nodes used

- **Webhook** — receives POST requests from agent systems
- **Edit Fields (Set)** — normalizes action fields from the request body
- **Code** — classifies action type as low or high impact based on prefix matching
- **If (Requires Human Approval?)** — gates high-impact actions to the approval path
- **Slack (sendAndWait)** — sends an approval request with Approve/Decline buttons and pauses until a human responds (30-minute timeout)
- **If (Approved?)** — routes based on the human's decision
- **HTTP Request** — executes the approved action (placeholder endpoint)
- **Respond to Webhook (200 Executed Directly)** — low-impact path response
- **Respond to Webhook (200 Approved and Executed)** — high-impact approved response
- **Respond to Webhook (403 Action Declined)** — human declined or timed out

---

## Requirements

- n8n instance (self-hosted or cloud)
- Slack workspace with an n8n OAuth2 app (scope: `chat:write`)
- An action execution endpoint (replace the placeholder in "Execute Approved Action")

---

## How to import

1. Download `workflow.json` from this folder
2. Open your n8n instance
3. Go to **Workflows** and click **Add workflow**
4. Select **Import from file** and choose `workflow.json`

---

## Setup after import

1. Create a Slack OAuth2 credential in n8n with the `chat:write` scope
2. Open **Request Human Approval** and replace `YOUR_SLACK_CHANNEL_ID` with your approvals channel ID (format: `C01234ABCDE`)
3. Link your Slack credential to the **Request Human Approval** node
4. Open **Execute Approved Action** and replace the URL placeholder with your action execution endpoint
5. Activate the workflow

**Test it (low impact, executes immediately):**
```bash
curl -X POST http://localhost:5678/webhook/llm-hitl-gate \
  -H "Content-Type: application/json" \
  -d '{"action_type":"search_records","action_payload":{"query":"invoices"},"user_id":"u1","agent_id":"a1"}'
```

**Test it (high impact, triggers Slack approval):**
```bash
curl -X POST http://localhost:5678/webhook/llm-hitl-gate \
  -H "Content-Type: application/json" \
  -d '{"action_type":"delete_record","action_payload":{"record_id":"rec_789"},"user_id":"u1","agent_id":"a1"}'
```

---

## Customization

**Change which actions require approval:** Edit the `lowImpact` array in the **Classify Action Severity** code node. Any action type that starts with a listed prefix executes without approval. Everything else routes through Slack.

```javascript
const lowImpact = ['search', 'read', 'get', 'list', 'fetch', 'query', 'find'];
```

**Change the approval timeout:** Open **Request Human Approval** and adjust `resumeAmount` (default: 30 minutes). After timeout, the workflow resumes with `approved: false` and the caller receives a 403. This prevents orphaned approvals from blocking indefinitely.

**Add an audit log:** Wire a database write (Postgres, Supabase) or an HTTP Request to a logging service between the approval decision and the response nodes to record every approval and rejection with a timestamp and approver identity.

**Capture the approver's identity:** Enable `captureResponder` in the Slack sendAndWait node to record which Slack user clicked Approve or Decline. Requires Slack Interactivity to be configured on the n8n app and `users:read` scope.

---

## Known limitations

- The action execution node is a placeholder. Replace it with the actual integration nodes for your system (database write, API call, email send, etc.).
- Slack sendAndWait requires the n8n instance to be accessible from Slack's servers for the button callback. Self-hosted n8n behind a firewall requires a tunnel or reverse proxy.
- If the Slack message is deleted or the bot is removed from the channel, the workflow times out after the configured interval.
- The severity classification is prefix-based and may misclassify action types that do not follow naming conventions. For production use, replace the prefix list with an explicit allowlist mapping action types to severity levels.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
