# SOC 2 Type II Compliance

**Framework:** SOC 2 Type II Trust Service Criteria  
**Relevance:** Any SaaS or enterprise AI deployment requiring third-party audit evidence of security, availability, and confidentiality controls

---

## Artifacts

| Artifact | Type | Status |
|----------|------|--------|
| AI interaction audit log pipeline | Workflow | In development |

---

## Why SOC 2 matters for LLM deployments

SOC 2 Type II audits evaluate controls over time, not just at a point in time. For AI systems, the critical gap is often auditability: can you prove what the model was asked, what it returned, who asked it, and when? The AI interaction audit log pipeline addresses this by writing every prompt and response to a tamper-evident log in Supabase — including a content hash, timestamp, and user ID — creating the non-repudiation trail an auditor needs.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign)
