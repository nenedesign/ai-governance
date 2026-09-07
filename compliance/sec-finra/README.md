# SEC/FINRA Compliance

**Framework:** U.S. Securities and Exchange Commission · Financial Industry Regulatory Authority  
**Relevance:** Any AI system operating in a registered investment advisory, broker-dealer, or financial communications context

---

## Artifacts

| Artifact | Type | Status |
|----------|------|--------|
| Investment advice guardrail prompt library | System prompt | In development |
| Personalized recommendation output scanner | Workflow | In development |

---

## Why SEC/FINRA matters for LLM deployments

SEC and FINRA rules on investment advice, suitability, and communications apply when an AI system could be construed as providing personalized financial guidance. An LLM that says "you should buy X" to a specific user may trigger investment adviser registration requirements or FINRA communication rules. The guardrail prompt library defines what the model can and cannot say — with regulatory rationale for each boundary — and the output scanner flags responses that cross into personalized recommendation territory before they reach the user.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
