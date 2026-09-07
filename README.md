# AI Governance & LLM Guardrails
### OWASP LLM Top 10 Implementations for Regulated Industries

AI security controls and governance patterns mapped directly to the [OWASP Top 10 for Large Language Model Applications v2.0 (2025)](reference/owasp-llm-top10-v2.0.md). Each artifact addresses a specific risk with a runnable n8n workflow, a defensive prompt pattern, or a governance document, not documentation for its own sake.

Built for teams deploying AI in regulated environments: financial services, healthcare, legal, and enterprise.

> **Build status:** Complete. All 10 OWASP LLM risks covered across 5 workflows, 6 prompt library entries, and 2 governance documents.

---

## Coverage

| Risk | Name | Artifact | Status |
|------|------|----------|--------|
| LLM01 | Prompt Injection | Workflow + Prompt | ✓ Done |
| LLM02 | Sensitive Information Disclosure | Workflow + Prompt | ✓ Done |
| LLM03 | Supply Chain | Governance | ✓ Done |
| LLM04 | Data and Model Poisoning | Workflow | ✓ Done |
| LLM05 | Improper Output Handling | Prompt | ✓ Done |
| LLM06 | Excessive Agency | Workflow + Prompt | ✓ Done |
| LLM07 | System Prompt Leakage | Governance + Prompt | ✓ Done |
| LLM08 | Vector and Embedding Weaknesses | Workflow | ✓ Done |
| LLM09 | Misinformation | Prompt | ✓ Done |
| LLM10 | Unbounded Consumption | Workflow | ✓ Done |

### Why LLM04 and LLM08 have no prompt library entry

Six of the ten risks have defensive system prompt patterns. LLM04 and LLM08 do not, and that is intentional.

A system prompt runs inside a model at inference time. It can constrain what the model says, how it handles untrusted input, and what actions it will take. What it cannot do is validate or sanitize data that was already embedded into a vector store before the conversation began.

LLM04 (Data and Model Poisoning) and LLM08 (Vector and Embedding Weaknesses) are **pre-inference pipeline risks**. Poisoning attacks corrupt training data or RAG knowledge bases during ingestion. Embedding weaknesses (unauthorized access, cross-tenant leakage, embedding inversion) are access control and infrastructure failures in the vector database layer. By the time a model processes a query, both risks have already materialized or been prevented. A prompt instruction has no surface to act on.

The correct control layer for both risks is the ingestion pipeline: validate sources, sanitize documents, enforce access partitioning, and log retrievals before content reaches the model. That is what the [RAG Security Pipeline](workflows/llm04-llm08-rag-security-pipeline/) workflow addresses, combining both risks because they share the same intervention point.

This separation reflects a broader principle: **match the control to the attack surface**. Applying a prompt-layer defense to a pipeline-layer risk produces the illusion of coverage without the substance.

---

## Repo Structure

```
ai-governance/
├── reference/                          # Pinned source documents
│   └── owasp-llm-top10-v2.0.md        # OWASP LLM Top 10 v2.0, March 2025
├── prompt-library/                     # Defensive system prompt patterns
│   ├── llm01-anti-injection.md
│   ├── llm02-pii-non-disclosure.md
│   ├── llm05-output-sanitization.md
│   ├── llm06-minimal-agency.md
│   ├── llm07-prompt-security-design.md
│   └── llm09-grounding-uncertainty.md
├── governance/                         # Model intake and audit documents
│   ├── llm03-model-intake-assessment.md
│   └── llm07-system-prompt-audit.md
└── workflows/                          # Runnable n8n workflow JSON + docs
    ├── llm01-prompt-injection-scanner/
    ├── llm02-pii-detector/
    ├── llm04-llm08-rag-security-pipeline/
    ├── llm06-hitl-approval-gate/
    └── llm10-rate-limiter/
```

---

## Methodology

All implementations are built against [OWASP LLM Top 10 v2.0](reference/owasp-llm-top10-v2.0.md), published March 12, 2025, retrieved and pinned on 2026-09-06. The reference document will be updated when OWASP publishes a new major version; git history tracks which version each artifact was built against.

**Note on naming:** This repository implements the *OWASP Top 10 for Large Language Model Applications*, a distinct list from the *OWASP Top 10 for Web Applications* (A01 Broken Access Control, A02 Security Misconfiguration, etc.). Both are published by OWASP but address different risk surfaces. This repository covers the LLM-specific list only.

Workflows are built in [n8n](https://n8n.io) and import directly into any n8n instance. Prompt library entries are framework-agnostic and work with any LLM.

Supporting frameworks referenced throughout:
- NIST AI Risk Management Framework (AI RMF 1.0)
- OSFI Guideline E-23: Model Risk Management
- SOC 2 Type II Trust Service Criteria
- PCI-DSS v4.0
- SEC/FINRA communication and supervision requirements

**Reference labeling:** Real-world references throughout this repository are labeled to distinguish their verification level. **[Confirmed]**: a documented public incident with a verifiable outcome. **[Technique]**: an attack method described in security research, not attributed to a single named incident. **[Scenario]**: a constructed example based on known failure modes in the domain, used to illustrate a risk class.

---

## License

Original content in this repository is licensed under the [MIT License](LICENSE).  
`reference/owasp-llm-top10-v2.0.md` is adapted from the OWASP Foundation under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

---

## About

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign), AI Product Manager and Builder at [Distinct AI](https://www.fromus.ca/ai-builds).
