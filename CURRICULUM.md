# FIE Curriculum

## Purpose

This document defines the learning philosophy and progression of the Financial Infrastructure Engineering curriculum.

The curriculum is designed for an experienced backend engineer who wants to progress toward **Staff Engineer / Architect capability in financial infrastructure**, while building strong AI engineering depth.

## The central model

The curriculum is deliberately layered:

```text
Foundations
    ↓
Algorithms & Problem Solving
    ↓
Computer Systems
    ↓
Advanced Java / JVM + Concurrency
    ↓
Distributed Systems
    ↓
Scalability / Cloud / Reliability
    ↓
Financial Infrastructure
    ↓
Performance / Low Latency
    ↓
AI Engineering
    ↓
Financial AI
    ↓
Architecture / Staff Engineering
```

The order is not absolute. Some tracks can progress in parallel once their prerequisites are satisfied. The dependency graph in `ROADMAP.md` is the source of truth for sequencing.

## Why this order?

### 1. Foundations before abstraction

Before using sophisticated frameworks, the engineer should understand the programming, data, mathematical, and networking concepts that the frameworks abstract away.

### 2. Algorithms before large-scale reasoning

Algorithmic thinking develops complexity analysis, data representation, decomposition, and optimization skills used throughout systems engineering.

### 3. Computer systems before distributed systems

Distributed systems are built from machines, networks, storage, processes, threads, and failure. Understanding local system behavior makes distributed failure modes much easier to reason about.

### 4. JVM and concurrency before performance

Low-latency Java requires a mental model of memory, allocation, JIT compilation, garbage collection, synchronization, contention, and CPU behavior.

### 5. Distributed systems before advanced microservices

Microservices, queues, cloud primitives, and orchestration platforms are implementation choices. The underlying distributed-systems concepts must come first so that tools do not replace understanding.

### 6. Financial infrastructure after systems foundations

Core banking, payments, ledgers, settlement, and reconciliation combine transactional correctness with distributed failure, concurrency, security, and operational constraints.

### 7. AI after strong systems foundations

Production AI is itself a systems problem: data pipelines, model serving, latency, reliability, evaluation, observability, and cost. Strong systems knowledge makes AI engineering more rigorous.

## Mastery loop

For every meaningful concept:

```text
Question
  ↓
Conceptual model
  ↓
Structured resource
  ↓
Implementation
  ↓
Experiment / lab
  ↓
Failure analysis
  ↓
Production or financial use case
  ↓
Written explanation
  ↓
Public evidence
```

Watching a course is exposure. Implementing a system is practice. Measuring it is engineering. Explaining the trade-offs is evidence of deeper mastery.

## Resource hierarchy

When selecting resources, prefer:

1. rigorous university courses with assignments
2. authoritative books
3. primary documentation and research papers
4. high-quality implementation repositories
5. structured practitioner courses
6. targeted videos
7. interview-preparation material

A resource can be included because it is exceptionally accessible, practical, visual, or useful for interviews even when it is not the deepest source.

## Evidence hierarchy

Evidence should become progressively stronger:

```text
Explanation
    ↓
Exercises
    ↓
Implementation
    ↓
Tests
    ↓
Benchmark / experiment
    ↓
Production-like project
    ↓
Public technical write-up
    ↓
Open-source contribution / external validation
```

## Staff-level progression

A subject is not considered Staff-level merely because its API is known.

Staff-level capability requires the ability to:

- identify the real problem
- define constraints and invariants
- reason about failure modes
- evaluate alternatives
- quantify important trade-offs
- understand operational consequences
- communicate decisions clearly
- design for evolution rather than only the happy path
- recognize when an abstraction is inappropriate

## Financial Infrastructure lens

Throughout the curriculum, concepts should repeatedly be connected to:

- money movement
- double-entry accounting
- ledger correctness
- payment lifecycle
- idempotency
- reconciliation
- settlement
- risk and fraud
- auditability
- compliance
- availability and recoverability
- high throughput
- predictable latency
- multi-region operation

The goal is not to memorize financial terminology. It is to understand how engineering constraints interact with financial truth.

## AI lens

AI is treated as an engineering discipline rather than a collection of tools.

The curriculum therefore includes:

- machine learning foundations
- data and evaluation
- LLM internals
- retrieval
- agents and tool use
- inference
- observability
- reliability
- security
- cost and latency
- financial AI applications

## What completion means

There is no single final certificate for this curriculum.

Completion means having accumulated a coherent body of evidence demonstrating the ability to **design, implement, benchmark, operate, explain, and evolve complex financial systems**.
