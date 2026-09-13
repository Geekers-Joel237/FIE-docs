# FIE Learning Framework

This document defines how every topic in the Financial Infrastructure Engineering curriculum is researched, selected, learned, practiced, assessed, and evidenced.

## 1. The fundamental questions

Every module must answer:

### Why?
Why does this concept exist? What problem does it solve?

### Why here?
Why is it introduced at this point rather than earlier or later?

### Why this and not that?
What competing concepts, techniques, or abstractions exist, and why is this one selected?

### What breaks without it?
Which failure modes or limitations become visible when the concept is missing?

### What is the mental model?
Can the concept be explained without relying on framework-specific vocabulary?

### Where does it apply?
Where does it appear in production systems and, specifically, financial infrastructure?

### What are the trade-offs?
What does the technique improve, and what does it cost?

### What proves mastery?
What implementation, experiment, benchmark, explanation, or external signal demonstrates competence?

## 2. Standard module template

Every substantive topic should eventually use this structure:

```text
# Topic

## 1. Why this topic exists
## 2. Why it matters for Financial Infrastructure Engineering
## 3. Prerequisites
## 4. Why learn it now?
## 5. Learning objectives
## 6. Fundamental questions
## 7. Concept dependency graph
## 8. Concepts in learning order
## 9. Primary resource
## 10. Supporting resources
## 11. Practice and labs
## 12. Failure modes and misconceptions
## 13. Financial infrastructure applications
## 14. Staff-level expectations
## 15. Proof of mastery
## 16. Evidence to publish
## 17. Further research
```

## 3. Resource evaluation framework

Resources are evaluated against:

| Dimension | Question |
|---|---|
| Structure | Does it provide a coherent sequence? |
| Depth | Does it explain mechanisms rather than recipes? |
| Practice | Are there exercises, labs, or projects? |
| Authority | Is the source academically or professionally credible? |
| Currency | Is the material maintained and technically current? |
| Accessibility | Is it free, affordable, or otherwise accessible? |
| Relevance | Does it serve the FIE target profile? |
| Transfer | Can knowledge be applied outside the exact examples? |
| Evidence | Does completion produce credible external proof? |

No resource is automatically preferred because it is popular.

## 4. Primary resource principle

Each topic should have one recommended **primary path** whenever practical.

Supporting resources exist for specific reasons:

- alternative explanation
- deeper theory
- implementation reference
- visual explanation
- exercises
- current documentation
- research perspective
- interview preparation

A list of equivalent resources without a decision is considered incomplete.

## 5. Mastery criteria

Mastery is progressive:

### L0 — Awareness
Can identify and define the concept.

### L1 — Foundation
Can explain the mechanism and solve basic exercises.

### L2 — Practitioner
Can implement it correctly in a contained system.

### L3 — Advanced
Can diagnose edge cases, performance issues, and failure modes.

### L4 — Senior
Can select among alternatives under realistic constraints.

### L5 — Staff
Can establish system boundaries, reason about second-order effects, review designs, and guide other engineers.

### L6 — Expert / Architect
Can handle novel constraints, contribute new designs or technology, and teach the subject deeply.

## 6. Evidence standards

Evidence should be proportional to the claim.

### Conceptual claim
A clear written explanation and successful assessment may be enough.

### Implementation claim
Provide working code, tests, and design notes.

### Performance claim
Provide reproducible benchmarks, environment details, methodology, and interpretation.

### Architecture claim
Provide requirements, alternatives, decision rationale, diagrams, failure analysis, and operational considerations.

### Staff-level claim
Provide integrated project evidence plus written technical reasoning and, where possible, external review or adoption.

## 7. Learning economics

The curriculum optimizes for **signal per unit of time**, not maximum content consumption.

When two resources teach the same material, prefer the one that provides stronger structure, deeper understanding, better practice, stronger evidence, or lower opportunity cost.

## 8. Research standard

For important recommendations, research should prioritize:

1. official course pages and syllabi
2. university material
3. official technical documentation
4. original papers
5. maintained open-source repositories
6. authoritative books
7. reputable practitioner material
8. community discussion when useful for practical context

Claims that may change over time should be re-verified before being treated as current.

## 9. FIE integration test

A topic is especially valuable when it changes how we would design one of these systems:

- ledger
- payment processor
- payment orchestration layer
- core banking service
- reconciliation engine
- settlement system
- risk/fraud engine
- financial data platform
- low-latency transaction processor
- AI-powered financial service

Each major track should contain explicit integration exercises.
