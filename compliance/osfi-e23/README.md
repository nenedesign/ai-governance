# OSFI E-23 Compliance

**Framework:** Office of the Superintendent of Financial Institutions — Guideline E-23: Model Risk Management  
**Relevance:** Canadian federally regulated financial institutions (FRFIs) deploying AI or ML models in any risk-relevant function

---

## Artifacts

| Artifact | Type | Status |
|----------|------|--------|
| Confidence-gated human escalation workflow | Workflow | Done (see [LLM06 HITL](../../workflows/llm06-hitl-approval-gate/)) |
| Model card template | Governance doc | In development |

---

## Why OSFI E-23 matters for LLM deployments

OSFI E-23 requires federally regulated financial institutions to maintain a model risk management framework covering model development, validation, approval, and ongoing monitoring. For LLM deployments, this means documenting model purpose, known limitations, validation approach, and escalation procedures. The model card template satisfies E-23's documentation requirements in a format designed for LLM-specific risks. The confidence-gated escalation workflow implements the human oversight requirement: low-confidence model outputs route to a human reviewer rather than being delivered directly to the user.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
