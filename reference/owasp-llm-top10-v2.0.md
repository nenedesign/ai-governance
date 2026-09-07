# OWASP Top 10 for Large Language Model Applications: v2.0 (2025)

**Source:** https://genai.owasp.org/llm-top-10/  
**Published:** March 12, 2025  
**Retrieved:** 2026-09-06  
**Version pinned:** 2.0 (2025); update this file when OWASP publishes v3.0

All prompt library entries and n8n workflows in this repository are built against this version.

---

**Attribution:** This document is an adaptation of the [OWASP Top 10 for Large Language Model Applications v2.0](https://genai.owasp.org/llm-top-10/), copyright © OWASP Foundation, used under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). Content has been summarized, reformatted, and annotated for use as a pinned reference in this repository. The original work has not been reproduced verbatim in its entirety.

---

## LLM01:2025: Prompt Injection

**What it is:** User prompts alter the LLM's behavior or output in unintended ways, either by directly hijacking instructions (direct injection) or by embedding malicious instructions in external content the LLM processes (indirect injection via websites, files, RAG documents).

**Distinct from jailbreaking:** Injection manipulates responses through specific inputs. Jailbreaking causes the model to disregard safety protocols entirely.

**Key impacts:** Sensitive information disclosure, unauthorized function access, arbitrary command execution in connected systems, manipulation of critical decisions.

**Mitigations:**
1. Constrain model behavior through specific role definitions and context limits
2. Define expected output formats with validation rules
3. Filter inputs and outputs using semantic and string-checking methods
4. Enforce least-privilege access: model accesses only what it needs
5. Require human approval for high-risk operations
6. Segregate and clearly label external/untrusted content
7. Conduct regular adversarial testing and red-teaming
8. Ongoing model training updates addressing emerging jailbreak techniques

**Notable attacks:** CVE-2024-5184 (email assistant), resume payload splitting, multimodal injection (hidden prompts in images), Base64/emoji obfuscation.

---

## LLM02:2025: Sensitive Information Disclosure

**What it is:** LLMs expose sensitive data (PII, financial details, health records, credentials, legal documents, proprietary model information) through insufficiently sanitized outputs or training data regurgitation.

**Key insight:** System prompt restrictions may not always be honored and can be bypassed via prompt injection.

**Mitigations:**
1. Scrub or mask sensitive content before training data inclusion
2. Apply strict input validation to detect and filter sensitive data
3. Enforce least privilege: limit model access to external data sources
4. Use federated learning to minimize centralized data collection
5. Apply differential privacy: add noise to prevent reverse-engineering
6. Maintain clear data retention, usage, and deletion policies
7. Conceal system preamble: limit user ability to access initial settings
8. Use tokenization and redaction to detect and remove confidential content before processing
9. Apply homomorphic encryption where feasible

**Notable incidents:** Samsung ChatGPT data leak; CVE-2019-20634 (Proof Pudding: training data extraction enabling model inversion).

---

## LLM03:2025: Supply Chain

**What it is:** Vulnerabilities in the LLM supply chain (training data, pre-trained models, fine-tuning adapters, third-party packages, and deployment platforms) that can introduce backdoors, biases, or malicious code.

**Unique to ML:** Unlike traditional software, risks extend to third-party datasets and pre-trained models that can be compromised through tampering or poisoning.

**Mitigations:**
1. Thoroughly vet data sources and suppliers: audit security posture and access controls
2. Apply OWASP Top Ten A06:2021 (Vulnerable and Outdated Components) controls
3. Conduct AI red teaming when selecting third-party models
4. Maintain a Software Bill of Materials (SBOM); consider AI BOM / ML SBOM using OWASP CycloneDX
5. Create comprehensive license inventories and conduct regular audits
6. Only deploy models from verifiable sources; use signing and file hashes
7. Monitor collaborative model development environments for abuse
8. Implement anomaly detection and adversarial robustness tests on supplied models
9. Ensure applications rely on maintained API and model versions with regular patching
10. Encrypt on-device models with integrity checks; use vendor attestation APIs

**Notable attacks:** PoisonGPT (direct model parameter modification on HuggingFace), CVE-2023-4969 (LeftOvers: GPU memory leakage), WizardLM impersonation, HiddenLayer model merge service compromise.

---

## LLM04:2025: Data and Model Poisoning

**What it is:** Manipulation of training data during pre-training, fine-tuning, or embedding stages to introduce vulnerabilities, backdoors, or biases that compromise model integrity and downstream system security.

**Mitigations:**
1. Track data origins and transformations using OWASP CycloneDX or ML-BOM
2. Rigorously vet data vendors; validate outputs against trusted sources
3. Implement sandboxing and anomaly detection for unverified data sources
4. Enforce infrastructure controls preventing unintended data source access
5. Deploy data version control (DVC) to detect manipulation
6. Store user-supplied information in vector databases enabling adjustment without full retraining
7. Conduct red team testing and adversarial robustness validation
8. Monitor training loss and model behavior for poisoning indicators
9. Integrate RAG and grounding techniques during inference

**Techniques to know:** Split-View Data Poisoning, Frontrunning Poisoning, backdoor insertion enabling authentication bypass or data exfiltration.

---

## LLM05:2025: Improper Output Handling

**What it is:** Insufficient validation, sanitization, and handling of LLM outputs before they are passed downstream to browsers, databases, or execution environments, enabling XSS, CSRF, SSRF, SQL injection, and remote code execution.

**Different from LLM09 (Misinformation):** This is about security vulnerabilities from unhandled output, not accuracy.

**Mitigations:**
1. Treat model outputs as untrusted: apply zero-trust to all model responses
2. Follow OWASP ASVS guidelines for validation and sanitization
3. Encode output for the target context (JavaScript, HTML, Markdown, SQL)
4. Use parameterized queries for any database operations
5. Implement Content Security Policy (CSP) to mitigate XSS from generated content
6. Log and monitor output patterns to detect exploitation attempts

**Attack vectors:** LLM output executed in `exec()`/`eval()`, unfiltered JavaScript in browsers, unparameterized LLM-generated SQL, unsanitized file path construction.

---

## LLM06:2025: Excessive Agency

**What it is:** LLM agents granted more functionality, permissions, or autonomy than required for their task, enabling damaging actions in response to unexpected, ambiguous, or manipulated outputs.

**Three root causes:**
- **Excessive functionality**: extensions include unneeded functions (delete when only read is needed)
- **Excessive permissions**: extensions granted unnecessary downstream system permissions
- **Excessive autonomy**: high-impact actions execute without human approval

**Mitigations:**
1. Minimize extensions to only what is necessary
2. Limit extension functions to minimum required capabilities
3. Avoid open-ended extensions; use granular, specific functionality
4. Restrict extension permissions to minimum necessary scope
5. Execute extensions within individual user contexts (not generic high-privilege identities)
6. Require human approval for high-impact actions
7. Implement complete mediation in downstream systems
8. Sanitize LLM inputs and outputs; use SAST/DAST testing
9. Monitor LLM extension activity and implement rate-limiting

**Canonical attack:** Indirect prompt injection via malicious email causes personal assistant agent to forward inbox contents to attacker, enabled by excessive send permissions and insufficient OAuth scoping.

---

## LLM07:2025: System Prompt Leakage

**What it is:** System prompts inadvertently reveal sensitive information (credentials, architecture details, filtering criteria, permission structures) that enables attackers to plan more targeted attacks.

**Critical insight:** "The system prompt should not be considered a secret, nor should it be used as a security control." Credentials and sensitive data must never be embedded in system prompts.

**Mitigations:**
1. Externalize API keys, credentials, and permission structures; store in systems the LLM cannot access directly
2. Do not rely on system prompts for strict behavior control; implement behavioral controls externally
3. Deploy independent guardrails inspecting outputs for compliance (outside the LLM)
4. Enforce privilege separation and authorization deterministically outside the LLM
5. Use auditable, non-LLM mechanisms for critical security functions

**What the real risk is:** Not the disclosure itself; it's the underlying failures: inadequate session management, compromised authorization, improper privilege separation, sensitive data in wrong locations.

---

## LLM08:2025: Vector and Embedding Weaknesses

**What it is:** Security vulnerabilities in how vectors and embeddings are generated, stored, or retrieved in RAG systems, enabling harmful content injection, output manipulation, and sensitive information access.

**Key risk categories:**
- Unauthorized access and data leakage through inadequate access controls
- Cross-context information leaks in multi-tenant vector databases
- Embedding inversion attacks (reverse-engineering embeddings to recover source data)
- Data poisoning via insider threats, manipulated prompts, or unverified sources
- Behavior alteration: RAG augmentation inadvertently degrades foundational model qualities

**Mitigations:**
1. Deploy fine-grained permissions: partition datasets logically to separate user classes
2. Establish robust validation pipelines accepting only verified sources
3. Review and classify combined datasets by access level
4. Maintain immutable retrieval logs for detecting suspicious behavior
5. Audit knowledge bases regularly

**Attack example:** White text on white background in documents (invisible to humans) embeds hidden instructions that RAG systems process, causing LLMs to bypass safeguards.

---

## LLM09:2025: Misinformation

**What it is:** LLMs produce false or misleading information that appears credible, caused by hallucination (statistical pattern-filling without true comprehension), overreliance, and training data gaps or biases.

**Mitigations:**
1. Use Retrieval-Augmented Generation (RAG) to ground outputs in verified external sources
2. Apply model fine-tuning and chain-of-thought prompting to improve reliability
3. Implement cross-verification and human oversight for critical outputs
4. Deploy automatic validation tools for high-stakes contexts
5. Clearly communicate LLM limitations and uncertainty to users
6. Establish secure coding practices to prevent vulnerable code integration
7. Label AI-generated content and specify field-of-use limitations in UI/UX
8. Train users on LLM limitations and the importance of verification

**Notable incidents:** Air Canada chatbot lawsuit (incorrect travel policy), ChatGPT fabricated legal cases cited in court, medical chatbots misrepresenting uncertainty.

---

## LLM10:2025: Unbounded Consumption

**What it is:** LLM systems allowing excessive, uncontrolled inferences that deplete computational resources, create financial exposure (Denial of Wallet), degrade service availability, or enable model extraction through API abuse.

**Mitigations:**
1. Enforce strict input size limits (input validation)
2. Restrict detailed probability information (limit logits/logprobs exposure)
3. Rate-limit requests per entity within timeframes
4. Monitor and cap resource consumption per user dynamically
5. Set timeouts and throttling for resource-intensive operations
6. Sandbox LLM access: restrict network and internal service access
7. Maintain comprehensive logging to detect unusual consumption patterns
8. Implement watermarking to detect unauthorized output usage
9. Ensure graceful degradation under heavy load
10. Apply RBAC and least-privilege access controls
11. Maintain a centralized ML model inventory
12. Use automated MLOps deployment with governance controls

**Attack types:** Variable-length input flood, Denial of Wallet (DoW), context window overflow, model extraction via API, side-channel attacks exploiting input filtering.

---

## Summary Table

| ID | Risk | Primary Control |
|----|------|----------------|
| LLM01 | Prompt Injection | Input filtering, privilege constraints, human approval gates |
| LLM02 | Sensitive Information Disclosure | Data sanitization, least privilege, differential privacy |
| LLM03 | Supply Chain | Supplier vetting, SBOM, model signing and provenance |
| LLM04 | Data and Model Poisoning | Data lineage tracking, anomaly detection, red teaming |
| LLM05 | Improper Output Handling | Output encoding, parameterized queries, zero-trust output |
| LLM06 | Excessive Agency | Minimal permissions, human-in-the-loop, granular extensions |
| LLM07 | System Prompt Leakage | Externalize credentials, independent guardrails, no security-by-obscurity |
| LLM08 | Vector and Embedding Weaknesses | Access partitioning, validation pipelines, retrieval logging |
| LLM09 | Misinformation | RAG grounding, human oversight, clear uncertainty communication |
| LLM10 | Unbounded Consumption | Rate limiting, input size caps, resource monitoring |
