# LLM02: PII Non-Disclosure System Prompt Pattern

**OWASP Risk:** LLM02:2025 Sensitive Information Disclosure  
**Source:** [OWASP LLM Top 10 v2.0](../reference/owasp-llm-top10-v2.0.md)  
**Mitigations addressed:** Input validation for sensitive data, least privilege data access, redaction before processing, limit user ability to extract system preamble

---

## The Risk

LLMs expose sensitive data in two ways: they repeat back sensitive information users share in conversation, or they surface data from training that should never have been included. Either path creates regulatory exposure.

In a financial services context: a user pastes their credit card number into a Wealthsimple support chat to ask about a transaction dispute. Without controls, the model echoes it back in its reply; now the card number appears in a chat log, a support ticket, and potentially a model context that persists across sessions. Under PCI-DSS v4.0, cardholder data must never be stored or transmitted in non-compliant systems. The chat interface likely isn't one.

A second scenario: a user asks a Raymond James AI assistant "what do you know about me?" The model, trained or fine-tuned on internal data, surfaces account details, advisor notes, or another client's information, a breach under CIRO client confidentiality rules and PIPEDA.

---

## Defensive System Prompt Pattern

```
DATA HANDLING POLICY
Do not repeat sensitive information shared during this conversation.
If a user shares sensitive data (including payment card numbers, government-issued ID numbers,
passwords, account credentials, medical information, or full names combined with financial
details), acknowledge the request without repeating the sensitive data and instruct the user
to use the appropriate secure channel instead.

Example response when sensitive data is detected:
"I can see you've shared some sensitive information. For your security, I won't repeat it here.
Please [specific secure channel or action]. I'm happy to help once that's done."

SCOPE OF KNOWLEDGE
You answer questions based only on [specify: approved knowledge base / this conversation /
provided documents]. You do not speculate about, infer, or reconstruct information you were
not explicitly given in this session. If asked what you "know" about a user, person, or
account, respond only with what has been provided to you in the current context.

TRAINING DATA BOUNDARY
You may not have been trained on data relevant to this deployment. Do not attempt to recall
or reconstruct information from training. If you are uncertain whether information is accurate,
say so explicitly rather than generating a plausible-sounding response.

OUTPUT REDACTION
Before responding, check whether your reply repeats any sensitive data the user shared in this conversation.
If it does, remove or mask it before replying. Use [REDACTED] as the placeholder.

THIRD-PARTY DATA
Do not share information about one user with another. If a request could expose another
person's data, even indirectly, decline and escalate to [human escalation path].
```

---

## What Each Section Defends Against

| Section | Risk it mitigates |
|---------|------------------|
| Data Handling Policy | User-shared PII echoed in responses or persisted in logs |
| Scope of Knowledge | Model fabricating or inferring account/personal details it wasn't given |
| Training Data Boundary | Training data regurgitation: model surfacing data from fine-tuning |
| Output Redaction | Sensitive data appearing in the model's own generated response |
| Third-Party Data | Cross-user data leakage in multi-tenant or session-shared deployments |

---

## Usage Notes

**This prompt is a last line of defense, not the first.** The right architecture scans and redacts sensitive data *before* it reaches the LLM (see [llm02-pii-detector workflow](../workflows/llm02-pii-detector/)). A prompt instruction can be bypassed via injection (LLM01). A workflow-level redaction step cannot.

**PCI-DSS scope boundary:** If your application can receive payment card data in any input field, the entire system, including the LLM and its context window, may be considered in-scope for PCI-DSS v4.0. The safest architecture excludes cardholder data from reaching the model entirely. This prompt addresses the case where that exclusion is not guaranteed.

**PIPEDA and provincial privacy law (Canada):** Personal information collected during an AI interaction may trigger retention, consent, and breach notification obligations under PIPEDA and Quebec Law 25. This prompt does not satisfy those obligations; it reduces exposure. Consult legal counsel for full compliance design.

**Do not list data types exhaustively in the prompt.** Enumerating every sensitive data type (e.g., "do not repeat SIN numbers, passport numbers, OHIP numbers...") teaches attackers exactly what you're protecting. Keep the list general; handle specifics in the workflow layer.

**Known limitations of this pattern.** This prompt instructs the model not to repeat sensitive data in its outputs. It cannot prevent a determined attacker from using prompt injection (LLM01) to cause the model to reveal data already present in the context window. It also has no effect on infrastructure-level logging; conversation logs, API request logs, and model context traces are captured by the application layer regardless of this instruction. PII that enters the model context is not protected by this prompt alone.

---

## Real-World Reference

- **[Confirmed] Samsung ChatGPT data leak (2023)**: Engineers pasted proprietary source code and meeting notes into ChatGPT, raising concerns that it could be incorporated into OpenAI's training pipeline. Widely reported in April 2023; Samsung subsequently restricted internal use of external AI tools. No prompt instruction could have prevented this; the architectural control (blocking sensitive data before it reaches the model) was absent. Prompt-layer controls are supplementary to, not a substitute for, workflow-layer redaction.
- **[Technique] Training data extraction**: Research by Carlini et al. (2021, "Extracting Training Data from Large Language Models") demonstrated that verbatim training data, including personally identifiable information such as names, email addresses, and phone numbers, could be recovered from GPT-2 through targeted probing of model outputs. This technique applies to any model trained without differential privacy on data containing PII. Mitigation: differential privacy during training and strict scope-of-knowledge constraints at inference time.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign).  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

---

## Disclaimer

This document is provided for educational and informational purposes only. The prompt patterns and guidance presented here are general-purpose starting points and do not constitute legal, compliance, or security advice. They have not been certified or validated against any specific regulatory framework, audit standard, or threat model.

Organizations deploying LLMs in regulated environments should conduct independent security assessments and consult qualified legal, compliance, and cybersecurity professionals before relying on any system prompt pattern as a compliance control.
