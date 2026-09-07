# Governance Documents

Structured intake and audit checklists for operationalizing AI governance in regulated environments. These documents translate OWASP LLM Top 10 v2.0 risk controls into repeatable, evidence-generating processes aligned with OSFI Guideline E-23, SOC 2 Type II, NIST AI RMF 1.0, and PCI-DSS v4.0.

Unlike prompt patterns, which operate at inference time, governance documents address the organizational controls around AI systems: what gets evaluated before a model is deployed, what gets audited after, and what records need to exist for a model risk management examination.

---

## Contents

| File | OWASP Risk | Purpose |
|------|-----------|---------|
| [llm03-model-intake-assessment.md](llm03-model-intake-assessment.md) | LLM03: Supply Chain | Structured checklist for vetting a new model or AI component before approving it for production use |
| [llm07-system-prompt-audit.md](llm07-system-prompt-audit.md) | LLM07: System Prompt Leakage | Structured audit checklist for reviewing a deployed or candidate system prompt for credentials, logic, and disclosure risk |

---

## Who These Are For

**Model risk managers and compliance teams** running governance programs under OSFI E-23 or equivalent: these checklists provide the structured evidence trail that model risk management examinations look for. Each document identifies the specific regulatory requirements it supports.

**Security teams** reviewing AI deployments before go-live: the intake assessment covers vendor security posture, data processing agreements, and integration risk. The system prompt audit includes active test prompts and a pass/fail scoring structure.

**Engineering and product teams** who need a governance handoff process: the approval records in each document create a clear chain of accountability from intake to deployment to periodic review.

---

## Usage Notes

Complete one intake assessment per model or AI component. Retain the completed document in your model inventory. The system prompt audit should be run on schedule (see the Audit Schedule table in that document) and retained as part of your audit evidence.

These documents are starting points, not certified compliance frameworks. Adapt field names and approval workflows to match your organization's processes. Organizations in regulated industries should engage qualified legal and compliance professionals before treating any checklist here as sufficient for regulatory purposes. The disclaimer at the bottom of each document applies.

---

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
