# Workflows

Runnable n8n workflows implementing pipeline-layer controls for OWASP LLM Top 10 v2.0 risks. Each workflow addresses risks that operate at the infrastructure and data layer, where prompt patterns have no surface to act on, or demonstrates a control pattern that must be enforced by application code rather than model instructions.

Import any workflow directly into n8n: open the workflow folder, download `workflow.json`, and use **File > Import from file** in n8n. Credentials are intentionally blank; connect your own before activating.

> **Status:** Workflows in development. Folders are scaffolded; `workflow.json` and per-workflow READMEs will be added as each build is completed.

---

## Contents

| Folder | OWASP Risk(s) | Description | Complexity |
|--------|--------------|-------------|------------|
| [llm01-prompt-injection-scanner/](llm01-prompt-injection-scanner/) | LLM01: Prompt Injection | Scans incoming user input for injection patterns before it reaches the model | Medium |
| [llm02-pii-detector/](llm02-pii-detector/) | LLM02: Sensitive Information Disclosure | Detects and redacts PII in model outputs before they reach end users | Medium |
| [llm04-llm08-rag-security-pipeline/](llm04-llm08-rag-security-pipeline/) | LLM04: Data Poisoning + LLM08: Embedding Weaknesses | Validates, sanitizes, and audits document ingestion into a RAG vector store | High |
| [llm06-hitl-approval-gate/](llm06-hitl-approval-gate/) | LLM06: Excessive Agency | Human-in-the-loop confirmation gate for irreversible agentic actions | Medium |
| [llm10-rate-limiter/](llm10-rate-limiter/) | LLM10: Unbounded Consumption | Per-user and per-session rate limiting on model API calls | Low |

---

## Why Workflows for LLM04 and LLM08

LLM04 (Data and Model Poisoning) and LLM08 (Vector and Embedding Weaknesses) have no prompt library entries because they are pre-inference risks. By the time a model processes a query, these attacks have already materialized or been prevented in the ingestion pipeline.

The RAG Security Pipeline workflow addresses both risks at their shared intervention point: the document ingestion process. It validates sources, sanitizes content, enforces access partitioning, and logs retrievals before anything reaches a vector store or model context.

See the [repo README](../README.md#why-llm04-and-llm08-have-no-prompt-library-entry) for the full architectural rationale.

---

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
