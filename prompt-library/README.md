# Prompt Library

Defensive system prompt patterns for six of the ten OWASP LLM Top 10 v2.0 risks. Each pattern is a drop-in system prompt block with explanatory notes on what each section defends against, usage guidance, known limitations, and real-world references.

These patterns are framework-agnostic. They work with Claude, GPT-4, Gemini, and any instruction-following model. Copy the prompt block, adapt the bracketed placeholders to your deployment, and review the usage notes before treating any pattern as a production control.

**Important:** Prompt-layer patterns are probabilistic, not deterministic. A sufficiently crafted adversarial input can cause a model to deviate from its instructions. For high-stakes deployments, pair every pattern here with application-layer enforcement: output filters, authorization middleware, schema validators, and audit logs. The usage notes in each file identify where prompt constraints are insufficient and what must be enforced externally.

LLM04 and LLM08 are not in this library because they are pre-inference pipeline risks with no prompt-layer intervention point. See the [repo README](../README.md#why-llm04-and-llm08-have-no-prompt-library-entry) for the full rationale.

---

## Contents

| File | OWASP Risk | What it defends |
|------|-----------|-----------------|
| [llm01-anti-injection.md](llm01-anti-injection.md) | LLM01: Prompt Injection | Untrusted input treated as instructions; indirect injection via retrieved content |
| [llm02-pii-non-disclosure.md](llm02-pii-non-disclosure.md) | LLM02: Sensitive Information Disclosure | PII, credentials, and confidential data repeated or inferred from context |
| [llm05-output-sanitization.md](llm05-output-sanitization.md) | LLM05: Improper Output Handling | Unsafe content passed downstream; structured output format violations |
| [llm06-minimal-agency.md](llm06-minimal-agency.md) | LLM06: Excessive Agency | Agentic models taking unauthorized or irreversible actions without confirmation |
| [llm07-prompt-security-design.md](llm07-prompt-security-design.md) | LLM07: System Prompt Leakage | Credentials and logic embedded in prompts; privilege escalation via self-assertion |
| [llm09-grounding-uncertainty.md](llm09-grounding-uncertainty.md) | LLM09: Misinformation | Hallucinated facts and citations; overreliance on stale or unsupported model output |

---

## How to Read Each File

Each document follows the same structure:

1. **The Risk** — what failure looks like in a regulated context, with a concrete scenario
2. **Defensive System Prompt Pattern** — the copy-paste prompt block
3. **What Each Section Defends Against** — table mapping each prompt section to the attack it mitigates
4. **Usage Notes** — deployment guidance, known limitations, and what must be enforced outside the prompt
5. **Real-world References** — labeled [Confirmed], [Technique], or [Scenario] per the labeling key in the repo README

---

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
