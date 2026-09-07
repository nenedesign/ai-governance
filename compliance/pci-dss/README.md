# PCI-DSS v4.0 Compliance

**Framework:** Payment Card Industry Data Security Standard v4.0  
**Relevance:** Any system that processes, stores, or transmits cardholder data — or where an LLM could be exposed to it

---

## Artifacts

| Artifact | Type | Status |
|----------|------|--------|
| Cardholder data detector | Workflow | In development |
| PCI scope boundary enforcement | System prompt | In development |

---

## Why PCI-DSS matters for LLM deployments

LLMs present a novel PCI-DSS risk: a user can paste card numbers, CVVs, or account data directly into a prompt. If that input reaches the model unfiltered, it enters the LLM's context window — potentially logged, cached, or reproduced in output. PCI-DSS v4.0 Requirement 3 prohibits storing sensitive authentication data after authorization. A pre-inference scan that detects and masks cardholder data before the LLM call enforces this boundary at the application layer.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
