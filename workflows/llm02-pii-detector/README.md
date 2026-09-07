# LLM02: PII Detector

**OWASP Risk:** LLM02:2025 Sensitive Information Disclosure  
**Source:** [OWASP LLM Top 10 v2.0](../../reference/owasp-llm-top10-v2.0.md)  
**Complexity:** Medium

---

## What it does

Scans raw LLM output for personally identifiable information before the response reaches end users, and replaces each match with a `[REDACTED:category]` placeholder. The caller receives the sanitized output alongside a redaction summary that reports how many instances of each PII category were found and replaced.

This is a post-inference control: it intercepts the model's response after generation and before delivery. A model cannot reliably self-censor PII — it may reproduce sensitive patterns from its training data or context window regardless of prompt instructions.

---

## Who it's for

Teams deploying LLM-powered APIs who need to prevent PII from leaking to end users. Applies to any system where user-submitted context (documents, emails, support tickets) flows into the model and the response could reproduce sensitive fragments.

Beginner to intermediate n8n users. No credentials or external services required.

---

## Nodes used

- **Webhook** — receives POST requests with `user_id`, `session_id`, and `llm_output` (the raw model response)
- **Edit Fields (Set)** — normalizes and defaults request body fields
- **Code** — scans `llm_output` against 7 PII regex patterns and replaces each match in-place; returns `sanitized_output` and a `redaction_summary` with per-category counts
- **Respond to Webhook (200)** — returns the sanitized output with `X-PII-Scan` and `X-Redaction-Count` headers

---

## PII categories detected

| Category | Pattern | Examples |
|----------|---------|----------|
| `email` | RFC 5321 local-part + domain | `john.doe@example.com` |
| `phone_ca_us` | North American 10-digit with optional country code | `416-555-0123`, `+1 (800) 555-1234` |
| `ssn_us` | US Social Security Number (dashed format) | `123-45-6789` |
| `sin_ca` | Canadian Social Insurance Number (spaced/dashed) | `123 456 789`, `123-456-789` |
| `credit_card` | Visa, Mastercard, Amex, Discover (with or without separators) | `4111-1111-1111-1111` |
| `postal_code_ca` | Canadian postal code | `M5V 2T6`, `K1A0B1` |
| `ip_address` | IPv4 address | `192.168.1.1` |

---

## Requirements

- n8n instance (self-hosted or cloud)
- No external credentials required

---

## How to import

1. Download `workflow.json` from this folder
2. Open your n8n instance
3. Go to **Workflows** and click **Add workflow**
4. Select **Import from file** and choose `workflow.json`

---

## Setup after import

1. Activate the workflow — no credentials to configure

**Test (clean output — no redaction):**
```bash
curl -X POST http://localhost:5678/webhook/llm-pii-detector \
  -H "Content-Type: application/json" \
  -d '{"user_id":"u1","llm_output":"The capital of France is Paris."}'
```

**Test (PII in output — redaction triggered):**
```bash
curl -X POST http://localhost:5678/webhook/llm-pii-detector \
  -H "Content-Type: application/json" \
  -d '{"user_id":"u1","llm_output":"Contact John at john.doe@example.com or call 416-555-0123. His SIN is 123 456 789."}'
```

**Expected response with PII:**
```json
{
  "status": "ok",
  "user_id": "u1",
  "sanitized_output": "Contact John at [REDACTED:email] or call [REDACTED:phone_ca_us]. His SIN is [REDACTED:sin_ca].",
  "redaction_summary": {
    "total_redactions": 3,
    "pii_detected": true,
    "categories": {
      "email": 1,
      "phone_ca_us": 1,
      "sin_ca": 1
    }
  }
}
```

Response headers include `X-PII-Scan: redacted` and `X-Redaction-Count: 3`.

---

## Customization

**Add PII categories:** Add new entries to the `patterns` array in the **Detect and Redact PII** code node. Each entry needs a `name` (used as the redaction label) and a `regex` with the global flag (`/pattern/g`).

**Change the redaction label:** Edit the replacement string in the `.replace()` call. Default is `` `[REDACTED:${pattern.name}]` ``. Change to `***`, `<redacted>`, or any format your downstream system expects.

**Log redaction events:** Add a Postgres or Supabase write node (or HTTP Request to a logging service) after the Code node to record every redaction event — user ID, session ID, categories detected, and timestamp. This creates an audit trail for PII governance.

**Wire into an existing LLM pipeline:** Instead of receiving `llm_output` from a webhook, call this workflow as a subworkflow from your main LLM pipeline using the Execute Workflow node. Pass the model's raw response as `llm_output` and use the `sanitized_output` field as the response to your end user.

---

## Known limitations

- Pattern matching is regex-based. It will not detect PII expressed in unusual formats (e.g., written-out phone numbers: "four one six five five five zero one two three"), obfuscated values, or non-standard encodings.
- The `sin_ca` pattern (three groups of three digits separated by spaces or dashes) may produce false positives for other numeric codes in similar formats. For production, supplement with a confidence threshold or NER-based validation.
- No de-duplication of redaction labels: if the same email appears three times, each is redacted individually and counted separately.
- This workflow processes one LLM output per request. For batch processing, restructure using a Split In Batches loop or implement batch input/output.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).
