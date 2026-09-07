# LLM03: Model Intake Assessment

**OWASP Risk:** LLM03:2025 Supply Chain  
**Source:** [OWASP LLM Top 10 v2.0](../reference/owasp-llm-top10-v2.0.md)  
**Frameworks referenced:** OSFI Guideline E-23, NIST AI RMF 1.0, OWASP CycloneDX ML-BOM

---

## Purpose

This document is a structured intake checklist for evaluating a new LLM or AI model component before it is approved for use in a production system. It addresses the supply chain risks that arise when organizations consume third-party models, fine-tuning adapters, datasets, or inference platforms without systematic vetting.

Use this checklist when:
- Adopting a new foundation model (API or self-hosted)
- Integrating a fine-tuned or quantized model variant from a public repository
- Onboarding a third-party AI vendor or managed inference service
- Adding a new training dataset or embedding model to an existing pipeline

Complete one assessment per model or component. Retain the completed assessment in your model inventory.

---

## Section 1: Model Identity and Provenance

| Field | Value |
|-------|-------|
| Model name and version | |
| Provider / source | |
| Source URL or repository | |
| Date retrieved | |
| File hash (SHA-256) | |
| License type | |
| Training data disclosed? | ☐ Yes ☐ No ☐ Partial |
| Fine-tuned from base model? | ☐ Yes (specify base) ☐ No |
| Published model card? | ☐ Yes (URL) ☐ No |
| Release channel (API / Hugging Face / self-hosted / other) | |

**Provenance minimum bar:** Model source must be verifiable. File hash must match the provider's published hash. Models from unverifiable sources (no organization, no hash, no license) must not proceed past this section without escalation.

---

## Section 2: Vendor Security Posture

Complete this section for any third-party provider whose infrastructure you will rely on (API providers, managed fine-tuning services, inference platforms).

| Question | Response |
|----------|----------|
| Does the vendor have a published security policy? | ☐ Yes (URL) ☐ No |
| SOC 2 Type II report available? | ☐ Yes ☐ No ☐ Not applicable |
| Data processing agreement (DPA) executed? | ☐ Yes ☐ No ☐ Pending |
| Zero data retention available and confirmed? | ☐ Yes ☐ No ☐ Not applicable |
| Incident disclosure policy published? | ☐ Yes ☐ No |
| Is your data used to train or improve the vendor's model? | ☐ No ☐ Yes ☐ Unknown |
| Penetration test results available (last 12 months)? | ☐ Yes ☐ No ☐ Not applicable |

**Minimum bar for regulated deployments:** DPA must be executed before any personal data reaches the vendor. Zero data retention should be confirmed for any client-facing or sensitive-data use case. If the vendor uses your data for training without explicit consent, the deployment is non-compliant with PIPEDA and may trigger GDPR obligations.

---

## Section 3: Training Data Assessment

| Question | Response |
|----------|----------|
| Training data sources disclosed? | ☐ Yes ☐ Partial ☐ No |
| Does disclosed data include web scrapes? | ☐ Yes ☐ No ☐ Unknown |
| Does disclosed data include user-generated content? | ☐ Yes ☐ No ☐ Unknown |
| Known copyright or licensing concerns with training data? | ☐ Yes ☐ No ☐ Unknown |
| Red team or bias evaluation published? | ☐ Yes (URL) ☐ No |
| Known failure modes or hazardous outputs documented? | ☐ Yes ☐ No |

**Note:** If training data sources are not disclosed, document this as a known risk. Models trained on unverified web data may have absorbed biased, inaccurate, or poisoned content. This increases hallucination risk (LLM09) and potential for biased outputs in credit, hiring, or client-facing contexts.

---

## Section 4: Intended Use Fit

| Question | Response |
|----------|----------|
| Intended use case in our environment | |
| Does the model card prohibit this use case? | ☐ Yes ☐ No ☐ No model card |
| Is the model designed for this domain (e.g., financial, medical, legal)? | ☐ Yes ☐ No ☐ General-purpose |
| Language(s) required | |
| Language(s) supported | |
| Context window sufficient for intended input size? | ☐ Yes ☐ No |
| Output format matches application requirements? | ☐ Yes ☐ No (describe gap) |

---

## Section 5: Integration Risk

| Question | Response |
|----------|----------|
| Will the model have access to production data? | ☐ Yes ☐ No |
| Will the model be able to take real-world actions (agentic use)? | ☐ Yes ☐ No |
| Are output validation controls in place before results reach users? | ☐ Yes ☐ No ☐ In progress |
| Is rate limiting enforced on model calls? | ☐ Yes ☐ No |
| Will model outputs be logged and retained? | ☐ Yes ☐ No |
| Retention period for model outputs | |
| Human review required before outputs reach end users? | ☐ Yes ☐ No ☐ For some outputs |

---

## Section 6: Risk Classification

After completing Sections 1–5, assign a risk tier:

| Tier | Criteria | Required action |
|------|----------|----------------|
| **Low** | General-purpose, read-only, no PII access, outputs reviewed before use | Document and proceed |
| **Medium** | Client-facing, accesses non-sensitive internal data, outputs semi-automated | Security review + monitoring plan required |
| **High** | Accesses PII or financial data, agentic (takes actions), outputs reach users without review | CISO approval + formal risk acceptance + monitoring |
| **Critical** | Influences credit, investment, or compliance decisions; handles cardholder data | Legal + compliance + CISO sign-off; consider advance engagement with relevant regulators (OSFI, CIRO) before deployment |

**Assigned tier:** _______________  
**Rationale:** _______________

---

## Section 7: Approval Record

| Field | Value |
|-------|-------|
| Assessment completed by | |
| Date completed | |
| Reviewed by (risk/security) | |
| Approved for use | ☐ Yes ☐ No ☐ Conditional |
| Conditions (if conditional) | |
| Next review date | |

---

## ML Bill of Materials Entry

Once approved, add this model to your ML-BOM. Minimum required fields (OWASP CycloneDX):

```
component:
  type: ml-model
  name: [model name]
  version: [version or commit hash]
  supplier: [provider name]
  purl: [package URL if available]
  hashes:
    - alg: SHA-256
      content: [file hash]
  licenses:
    - license:
        id: [SPDX identifier or "proprietary"]
  externalReferences:
    - type: model-card
      url: [model card URL or "not published"]
  properties:
    - name: intake-assessment-date
      value: [YYYY-MM-DD]
    - name: risk-tier
      value: [Low / Medium / High / Critical]
    - name: approved-by
      value: [approver name]
```

The `purl` field follows the Package URL (purl) specification. For models hosted on Hugging Face, the format is `pkg:huggingface/{owner}/{model-name}@{version-or-commit}`, for example, `pkg:huggingface/meta-llama/Llama-2-7b@main`. For API-only models without a public repository, omit the field and note "API: no package URL available."

---

## Regulatory Context

**OSFI Guideline E-23 (Model Risk Management):** Requires that federally regulated financial institutions maintain a model inventory, assess model risk before deployment, and subject high-risk models to independent validation. This assessment satisfies the intake stage of that requirement. Independent validation (Section 6, High/Critical tiers) must be completed separately.

**NIST AI RMF 1.0 (GOVERN, MAP):** This checklist addresses the MAP function (identifying and analyzing AI risks in context) and contributes to the GOVERN function (establishing policies and accountability for AI risk management).

**PCI-DSS v4.0:** If the model will process or generate output containing cardholder data, confirm scope with your QSA before deployment. The entire system, including model context windows, may be considered in-scope.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign).  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

---

## Disclaimer

This document is provided for educational and informational purposes only and does not constitute legal, compliance, regulatory, or security advice. It has not been reviewed or validated against any specific regulatory framework, audit standard, or organizational risk policy.

Organizations subject to OSFI, CIRO, FINRA, PCI-DSS, PIPEDA, or other regulatory requirements should engage qualified legal, compliance, and cybersecurity professionals to design intake and governance processes appropriate to their specific risk profile and regulatory obligations.
