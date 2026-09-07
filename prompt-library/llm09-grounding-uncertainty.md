# LLM09: Grounding and Uncertainty System Prompt Pattern

**OWASP Risk:** LLM09:2025 Misinformation  
**Source:** [OWASP LLM Top 10 v2.0](../reference/owasp-llm-top10-v2.0.md)  
**Mitigations addressed:** Source grounding, explicit uncertainty communication, overreliance prevention, RAG accuracy constraints

---

## The Risk

LLMs generate plausible-sounding responses even when they have no reliable basis for an answer. This is not deception; it is how language models work: they predict probable continuations of text, which produces confident-sounding output regardless of underlying accuracy.

The harm comes from overreliance: users treating model output as authoritative when it is not. In low-stakes consumer contexts, this is an annoyance. In regulated professional contexts, it creates material liability.

In a financial services context: a client-facing chatbot is asked "What is the current penalty for early RRSP withdrawal?" The model's training data is two years old. Regulations have changed. The model answers confidently, correctly describing the old rule. The client acts on it. Under CIRO guidance, providing inaccurate regulatory information, even via an automated system, is a supervision and suitability concern. The firm bears liability.

A second scenario: a legal research assistant confidently cites a case that does not exist, a well-documented failure mode (hallucinated citations). An attorney relies on it without verification. The citation is filed in court. This is not hypothetical: it happened in Mata v. Avianca (2023).

---

## Defensive System Prompt Pattern

```
KNOWLEDGE BOUNDARY
Your knowledge has a training cutoff of [date]. For time-sensitive information (current rates, regulatory updates, market data, recent events), treat your knowledge
as potentially outdated and say so explicitly.

When answering a time-sensitive question without access to current data:
"My information on this may be out of date. As of my training, [answer]. Please verify
with [authoritative source] before acting on this."

SOURCE GROUNDING
Answer questions based only on:
[Select what applies:
  - The documents or context provided in this conversation
  - The knowledge base retrieved and shown to you
  - Your training, clearly qualified as such]

If you cannot support a claim with information from the above sources, say so.
Do not generate supporting facts to fill gaps. Do not cite sources you cannot verify exist.

UNCERTAINTY EXPRESSION
Use explicit uncertainty markers when you are not confident:
- "I believe..." / "I'm not certain, but..."
- "You should verify this with [source]"
- "I don't have reliable information on this"

Do not use hedging language to soften a false claim while still making it.
If you do not know, say you do not know.

CITATION INTEGRITY
If you cite a specific document, case, regulation, or data point, you must be able
to point to it in the context provided to you. Do not generate citations from memory.
If a user asks for a source and you cannot provide a verifiable reference, respond:
"I don't have a source I can verify for that. Please check [authoritative reference]."

PROFESSIONAL ADVICE BOUNDARY
This assistant provides [describe: general information / research support / document
summaries]. It does not provide [legal / financial / medical / compliance] advice.
For decisions in these domains, direct users to qualified professionals.
Do not soften this boundary by providing substantive guidance that amounts to advice
while nominally disclaiming it.
```

---

## What Each Section Defends Against

| Section | Risk it mitigates |
|---------|------------------|
| Knowledge Boundary | Stale information presented as current: regulatory, market, or policy changes |
| Source Grounding | Hallucinated facts: model inventing supporting details with no basis |
| Uncertainty Expression | False confidence: hedged language masking an unsupported claim |
| Citation Integrity | Hallucinated citations: non-existent cases, articles, or regulations cited as real |
| Professional Advice Boundary | Unauthorized professional advice: model providing substantive guidance in regulated domains |

---

## Usage Notes

**Grounding is an architecture problem, not a prompt problem.** The most effective mitigation for LLM09 is Retrieval-Augmented Generation (RAG): retrieve verified, current documents and inject them into the model context. A model answering from a pinned, auditable knowledge base hallucinates far less than one answering from training alone. This prompt handles the cases where RAG is absent or incomplete.

**"I don't know" must be a permitted output.** Some deployments implicitly penalize model uncertainty by training on data that rewards confident, complete-sounding answers. If your use case is high-stakes, explicitly test whether the model says "I don't know" when it should. If it cannot, the grounding architecture is insufficient.

**Disclaimers are not a defense.** A prompt that says "I am not a financial advisor" followed by specific investment guidance is still giving investment advice. CIRO and SEC/FINRA enforcement actions have addressed AI-generated communications that disclaim advice while delivering substantive guidance. The disclaimer section of this prompt must be enforced behaviorally, not just stated.

**For RAG systems:** Pair this prompt with a source-citation requirement in the retrieval layer. Each retrieved chunk should carry provenance metadata (document name, date, section). The model should be constrained to cite only what was retrieved, and the application layer should validate that citations appear in the retrieved context before they reach the user.

**Test for hallucination rate before deployment.** For any high-stakes use case, measure your model's hallucination rate on a domain-specific evaluation set before deployment. "Confident and wrong" is not acceptable in regulated contexts regardless of disclaimers.

**Multi-model cross-checking for high-stakes outputs.** For legal, healthcare, and financial use cases where a single model's output could cause material harm (a cited regulation, a dosing guideline, a tax rule), consider routing critical claims through a second model for independent verification before surfacing them to users. This does not eliminate hallucination but reduces the probability that the same error appears in both outputs. It is a supplement to human review, not a replacement.

**Known limitations of this pattern.** These instructions reduce hallucination rate; they do not eliminate it. A model can produce confident false outputs even when instructed to express uncertainty, particularly in specialized domains where its training data is sparse or outdated. The CITATION INTEGRITY section asks the model not to cite sources it cannot verify; however, a model cannot reliably distinguish between a citation it retrieved and one it confabulated. Application-layer citation verification, confirming that a cited source appears in the retrieved context, is required for any deployment where citation accuracy is material.

---

## Real-World Reference

- **[Confirmed] Moffatt v. Air Canada (2024)**: Air Canada's chatbot provided a passenger with incorrect information about bereavement fare eligibility. Air Canada's defense was that it bore no responsibility for the chatbot's inaccurate statements, as the information should have been verified on the live website. The British Columbia Civil Resolution Tribunal rejected this: Air Canada was liable for all content on its platform, including its AI's output. The case established that deploying an AI system does not transfer responsibility for its errors to the user. Accurate grounding and an explicit "verify with an agent" fallback would have prevented the liability.
- **[Confirmed] Mata v. Avianca (2023)**: Attorneys submitted a legal brief in the US District Court for the Southern District of New York citing six AI-generated case citations. None of the cases existed. The court sanctioned counsel for failing to verify the citations before filing. The model produced confident, plausible-sounding case names and summaries with no factual basis. Mitigation: citation integrity controls and mandatory human verification of any cited legal authority.
- **[Technique] Healthcare chatbot overconfidence**: Academic research (including studies published in journals such as JAMA Network Open and Nature Medicine) documents cases where AI systems deployed in healthcare contexts expressed high-confidence responses that contradicted clinical guidelines or contained dosing errors. These are research findings rather than a single attributed incident. Mitigation: explicit uncertainty thresholds, mandatory escalation to human clinicians for treatment-adjacent questions, and confidence score thresholds below which the system must defer.

---

## Attribution

Built by [Neville Ko](https://www.linkedin.com/in/nevilleko/) · [GitHub](https://github.com/nenedesign).  
Derived from [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, published March 12, 2025, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

---

## Disclaimer

This document is provided for educational and informational purposes only. The prompt patterns and guidance presented here are general-purpose starting points and do not constitute legal, compliance, or security advice. They have not been certified or validated against any specific regulatory framework, audit standard, or threat model.

Organizations deploying LLMs in regulated environments should conduct independent security assessments and consult qualified legal, compliance, and cybersecurity professionals before relying on any system prompt pattern as a compliance control.
