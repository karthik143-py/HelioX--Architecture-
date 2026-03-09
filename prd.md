# HelioX RAG 3.0 — Product Requirements Document

**Document Version:** 1.0
**Date:** 2026-03-03
**Status:** Engineering-Ready Draft
**Related:** [architecture_blueprint.md](file:///d:/HelioX/architecture_blueprint.md)

---

## Table of Contents

1. [Product Vision](#1-product-vision)
2. [Target Users](#2-target-users)
3. [Problem Statement](#3-problem-statement)
4. [Functional Requirements](#4-functional-requirements)
5. [Non-Functional Requirements](#5-non-functional-requirements)
6. [Performance Targets](#6-performance-targets)
7. [Scalability Requirements](#7-scalability-requirements)
8. [Security Requirements](#8-security-requirements)
9. [Cost Constraints](#9-cost-constraints)
10. [Evaluation Metrics](#10-evaluation-metrics-offline--online)
11. [Failure Handling](#11-failure-handling)
12. [MVP vs Full Version Breakdown](#12-mvp-vs-full-version-breakdown)

---

## 1. Product Vision

### Mission Statement

HelioX RAG 3.0 is an enterprise-grade Retrieval-Guided Multi-Agent Deliberative System that delivers **provably grounded, citation-backed answers** over private document corpora — with deterministic cost guarantees, measurable confidence, and transparent reasoning.

### Strategic Goals

| Goal | Description |
|------|-------------|
| **Trustworthiness** | Every answer is evidence-bound. No uncited claims reach the user. |
| **Transparency** | Users see exactly which source spans support each statement, plus a calibrated confidence score. |
| **Cost Predictability** | Per-query token budgets, mode switching, and organizational guardrails ensure no cost surprises. |
| **Operational Readiness** | Circuit breakers, degradation ladders, and observability from day one — not bolted on later. |
| **Adaptive Complexity** | Simple queries get fast, cheap answers (Light mode). Complex queries get thorough, multi-agent deliberation (Heavy mode). |

### What HelioX RAG 3.0 Is NOT

- Not a general-purpose chatbot — it only answers from indexed corpora.
- Not a search engine — it synthesizes answers, not ranked links.
- Not a knowledge graph product — it uses graph structure internally but exposes natural-language answers.

---

## 2. Target Users

### Primary Personas

| Persona | Role | Need | Key Workflow |
|---------|------|------|-------------|
| **Compliance Analyst** | Enterprise risk / compliance team | Query regulatory documents, SOPs, and audit trails with citation-backed answers | "Does Policy X apply to EU customers after the 2025 amendment?" |
| **Legal Researcher** | Internal legal counsel | Cross-reference contracts, case law, and policy documents | "Compare indemnity clauses across Vendor A and Vendor B MSAs." |
| **Technical Writer** | Documentation team lead | Verify accuracy of technical docs against source specifications | "What does the API spec say about rate limits for enterprise tier?" |
| **Product Manager** | Cross-functional product lead | Synthesize insights from research reports and customer feedback | "Summarize the top 3 churn drivers from Q4 research." |
| **Platform Engineer** | DevOps / SRE team | Operate, monitor, and tune the system | Dashboard monitoring, configuration tuning, cost management |

### Secondary Personas

| Persona | Need |
|---------|------|
| **Data Engineer** | Manage ingestion pipelines, embedding versioning, index health |
| **Security Auditor** | Review access logs, data residency, PII handling |

### User Assumptions

- Users interact via a web-based UI or programmatic API.
- Users have domain expertise — they can evaluate answer quality and identify hallucination.
- Users expect answers within 5 seconds for simple queries and 30 seconds for complex multi-document synthesis.

---

## 3. Problem Statement

### Current State (Industry Pain Points)

| Problem | Impact |
|---------|--------|
| **Hallucination** | LLMs generate plausible but unsourced claims. Users cannot trust outputs for compliance, legal, or financial decisions. |
| **Flat retrieval** | Single-stage vector search misses metadata-filtered precision. Irrelevant chunks dilute context. |
| **No confidence signal** | Systems return answers with equal conviction regardless of evidence quality, leaving users to self-assess reliability. |
| **Unbounded cost** | Full-context reasoning on every query burns tokens. No mode switching, no budget caps, no early termination. |
| **Opaque reasoning** | Users cannot trace an answer back to specific source spans. Audit trails are absent. |
| **One-size-fits-all** | Simple fact lookups and complex multi-document synthesis share the same expensive pipeline. |

### Desired State

A system where:
1. Every answer sentence maps to a cited source span — or the system explicitly states "Insufficient evidence."
2. Retrieval is staged (metadata → entity → vector) to maximize precision before incurring embedding-search cost.
3. A calibrated confidence score accompanies every answer, with automatic escalation when confidence is low.
4. Cost is bounded per-query, per-user, and per-organization with hard token ceilings.
5. Simple queries are fast and cheap; complex queries activate multi-agent deliberation only when needed.
6. Conflicts between sources are detected, adjudicated, and transparently reported.

---

## 4. Functional Requirements

### FR-1: Document Ingestion

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1.1 | Support ingestion of PDF, DOCX, HTML, Markdown, and CSV file formats | P0 |
| FR-1.2 | Parse documents into hierarchical structure: section → subsection → paragraph → sentence | P0 |
| FR-1.3 | Extract tables into structured JSON representations | P0 |
| FR-1.4 | Remove noise (headers, footers, watermarks, boilerplate) | P0 |
| FR-1.5 | Build parent–child document graph preserving section hierarchy | P0 |
| FR-1.6 | Perform adaptive chunking: target 800–1200 tokens with 15% overlap at sentence/paragraph boundaries | P0 |
| FR-1.7 | Normalize ontology: resolve abbreviations, expand synonyms — without rewriting original text | P1 |
| FR-1.8 | Provide ingestion status API with per-document progress tracking | P0 |
| FR-1.9 | Support incremental re-ingestion (add/update/delete documents without full re-index) | P1 |

### FR-2: Multi-Vector Expert Profiling

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-2.1 | Generate primary semantic embedding per chunk | P0 |
| FR-2.2 | Generate NER-based entity vector per chunk | P0 |
| FR-2.3 | Generate and store HyDE embeddings in a separate collection | P1 |
| FR-2.4 | Extract and store constraint metadata: temporal, geographic, regulatory, applicability scope | P0 |
| FR-2.5 | Compute and store authority score per chunk (based on source rank, publication date, citation density) | P0 |
| FR-2.6 | Tag every embedding with `model_id` + `model_version`; support zero-downtime model upgrades via shadow-collection swap | P1 |

### FR-3: Query Analysis & Decomposition

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-3.1 | Classify query intent: fact / comparison / synthesis / compliance | P0 |
| FR-3.2 | Extract entity targets from query | P0 |
| FR-3.3 | Extract constraint filters: temporal, geographic, regulatory, scope | P0 |
| FR-3.4 | Decompose query into fact components and reasoning components | P0 |
| FR-3.5 | Detect query ambiguity and missing constraints | P0 |
| FR-3.6 | Validate entity grounding against indexed corpus | P0 |
| FR-3.7 | Score decomposition confidence; re-decompose if < 0.65 (max 1 retry) | P0 |
| FR-3.8 | Lock Structured Query Object (SQO) — immutable downstream | P0 |

### FR-4: Hybrid Retrieval

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-4.1 | Stage 1: Filter chunks by metadata constraints in PostgreSQL | P0 |
| FR-4.2 | Stage 2: Filter by entity overlap between SQO entities and chunk entity vectors | P0 |
| FR-4.3 | Stage 3: Perform vector similarity search in Qdrant within the filtered candidate set | P0 |
| FR-4.4 | Stage 4: Fallback broad retrieval if recall < 0.60 (bounded by `MAX_RETRIEVAL_EXPANSIONS = 2`) | P0 |
| FR-4.5 | Dynamic K: start at K=25, apply filters, re-rank, stop when marginal coverage gain < ε (0.02). Final K ∈ [5, 12] | P0 |
| FR-4.6 | Compute relevance-aware weighted score combining recency, source rank, citation density, worker confidence, cosine similarity, and query coverage | P0 |

### FR-5: Light vs Heavy Mode

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-5.1 | Light Mode: single retrieval pass, top-K ∈ [5, 8], no multi-agent adjudication, direct composition | P0 |
| FR-5.2 | Heavy Mode: full pipeline — parallel workers, cross-chunk synthesis, conflict detection + adjudication, confidence gating | P0 |
| FR-5.3 | Automatic mode selection based on intent classification and ambiguity detection | P0 |
| FR-5.4 | Light → Heavy escalation when Light-mode confidence < 0.70 | P0 |
| FR-5.5 | Allow user-specified mode override (`light`, `heavy`, `auto`) via API | P1 |

### FR-6: Parallel Worker Agents

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-6.1 | Each worker receives a locked SQO and exactly one chunk | P0 |
| FR-6.2 | Workers produce: claim, supporting spans (exact indices), reasoning trace, local confidence, coverage score | P0 |
| FR-6.3 | Workers must return `INSUFFICIENT` if evidence is inadequate — no fabrication | P0 |
| FR-6.4 | Workers execute in parallel with async dispatch | P0 |
| FR-6.5 | Support early termination: cancel remaining workers when global confidence ≥ 0.90 | P1 |

### FR-7: Cross-Chunk Synthesis

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-7.1 | Deduplicate semantically equivalent claims (similarity > 0.92 → merge) | P0 |
| FR-7.2 | Merge complementary, non-overlapping claims | P0 |
| FR-7.3 | Detect contradiction candidates (semantically opposing claims) | P0 |
| FR-7.4 | Build evidence graph: claims → supporting spans → source chunks | P0 |
| FR-7.5 | Compute global coverage score as fraction of SQO fact-components covered | P0 |

### FR-8: Conflict Detection & Adjudication

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-8.1 | Perform semantic contradiction detection via pairwise entailment scoring (threshold ≤ 0.30) | P0 |
| FR-8.2 | Apply authority weighting: source rank × recency × cross-worker agreement | P0 |
| FR-8.3 | Execute re-verification loop: max `MAX_CONFLICT_ROUNDS = 2` | P0 |
| FR-8.4 | If unresolved after 2 rounds → accept highest-weighted consensus and reduce global confidence | P0 |

### FR-9: Evidence-Bound Answer Composition

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-9.1 | Every factual sentence in the answer must map to at least one citation ID | P0 |
| FR-9.2 | All SQO constraints must be applied and reflected in the answer | P0 |
| FR-9.3 | If coverage < 0.60 → return "Insufficient evidence" with any partial results | P0 |
| FR-9.4 | Output schema includes: direct answer, supporting evidence array, constraints applied, confidence score, coverage score, conflict notes | P0 |

### FR-10: Confidence Gating

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-10.1 | Compute global confidence as weighted mean of: retrieval coverage, citation density, worker agreement, conflict severity, authority score, cosine similarity | P0 |
| FR-10.2 | Confidence ≥ 0.80 → deliver answer | P0 |
| FR-10.3 | Confidence 0.60–0.79 → deliver with confidence warning | P0 |
| FR-10.4 | Confidence 0.40–0.59 → trigger expanded retrieval (if expansions remaining) | P0 |
| FR-10.5 | Confidence < 0.40 → request user clarification | P0 |

### FR-11: Session Memory

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-11.1 | Store: validated structured summary, confirmed constraints, user preference context | P0 |
| FR-11.2 | Never store: raw worker reasoning, unverified claims, LLM speculation | P0 |
| FR-11.3 | Session TTL: 30 minutes idle (configurable) | P1 |
| FR-11.4 | Support multi-turn conversations with constraint carry-forward | P1 |

---

## 5. Non-Functional Requirements

### NFR-1: Reliability

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-1.1 | System availability | ≥ 99.9% (8.7 h downtime/year) |
| NFR-1.2 | Query success rate (non-5xx) | ≥ 99.5% |
| NFR-1.3 | Data durability (ingested documents) | 99.999% (PostgreSQL + Qdrant replicas) |
| NFR-1.4 | Zero data loss on graceful shutdown | Mandatory |
| NFR-1.5 | Graceful degradation under partial failure | 4-level degradation ladder (see §11) |

### NFR-2: Observability

| ID | Requirement |
|----|-------------|
| NFR-2.1 | Distributed tracing (OpenTelemetry) across all service-to-service calls |
| NFR-2.2 | Structured JSON logging with correlation IDs |
| NFR-2.3 | Real-time dashboards: latency, throughput, token usage, cost, confidence distribution |
| NFR-2.4 | Alerting on: circuit breaker trips, budget breaches, worker failure rate > 5%, P95 latency > 25 s |
| NFR-2.5 | Audit trail: every query, SQO, retrieval set, and answer logged with timestamps |

### NFR-3: Maintainability

| ID | Requirement |
|----|-------------|
| NFR-3.1 | All services independently deployable with no shared mutable state |
| NFR-3.2 | Configuration via environment variables and centralized config store (no hardcoded values) |
| NFR-3.3 | Embedding model swappable without downtime (shadow-collection swap) |
| NFR-3.4 | LLM provider swappable per phase (workers, synthesis, composition) via config |
| NFR-3.5 | Feature flags for: HyDE expansion, Heavy mode, conflict adjudication |

### NFR-4: Testability

| ID | Requirement |
|----|-------------|
| NFR-4.1 | Unit test coverage ≥ 80% for core logic (retrieval scoring, dynamic K, confidence computation) |
| NFR-4.2 | Integration test suite covering full Light and Heavy mode pipelines |
| NFR-4.3 | Confidence calibration test suite with ECE measurement |
| NFR-4.4 | Load test harness for sustained throughput benchmarks |

---

## 6. Performance Targets

### 6.1 Latency

| Metric | Light Mode | Heavy Mode | Measurement |
|--------|-----------|------------|-------------|
| P50 end-to-end | ≤ 2 s | ≤ 8 s | API gateway → response |
| P95 end-to-end | ≤ 4 s | ≤ 20 s | API gateway → response |
| P99 end-to-end | ≤ 6 s | ≤ 28 s | API gateway → response |
| Hard timeout | 10 s | 30 s | Partial result returned on breach |

### 6.2 Per-Phase Latency Budgets

| Phase | Budget | Hard Wall |
|-------|--------|-----------|
| Query analysis + decomposition | ≤ 500 ms | 2,000 ms |
| Metadata + entity filtering | ≤ 200 ms | 1,000 ms |
| Vector search | ≤ 800 ms | 3,000 ms |
| Worker pool (all workers) | ≤ 5 s (Light) / 12 s (Heavy) | 15,000 ms |
| Synthesis + adjudication | ≤ 2 s | 6,000 ms |
| Composition | ≤ 1 s | 2,000 ms |

### 6.3 Throughput

| Metric | Target |
|--------|--------|
| Concurrent queries (sustained) | 50 per gateway instance |
| Peak burst | 200 queries / 10 s window |
| Ingestion throughput | 500 documents/hour (batch) |
| Re-indexing throughput | 1,000 chunks/minute per profiling worker |

### 6.4 Resource Efficiency

| Metric | Target |
|--------|--------|
| Average tokens per Light query | ≤ 8,000 |
| Average tokens per Heavy query | ≤ 24,000 |
| Redis cache hit rate (repeated queries) | ≥ 30% in steady state |
| Qdrant query latency (vector search only) | ≤ 100 ms at 10M vectors |

---

## 7. Scalability Requirements

### 7.1 Data Scale

| Dimension | MVP Target | Full Version Target |
|-----------|-----------|-------------------|
| Total documents | 10,000 | 1,000,000 |
| Total chunks | 500,000 | 50,000,000 |
| Total vectors (across 3 collections) | 1,500,000 | 150,000,000 |
| Metadata rows (PostgreSQL) | 500,000 | 50,000,000 |
| Average document size | 20 pages | 20 pages |

### 7.2 Compute Scale

| Dimension | MVP | Full Version |
|-----------|-----|-------------|
| Gateway instances | 2 | 8+ (auto-scaled) |
| Worker pool size | 12 | 48 (across 4 nodes) |
| Qdrant nodes | 1 (single replica) | 3-node cluster with sharding |
| PostgreSQL | Single primary + 1 read replica | Primary + 2 read replicas + connection pooler (PgBouncer) |
| Redis | Single instance | 3-node sentinel cluster |

### 7.3 Scaling Strategy

| Component | Strategy |
|-----------|----------|
| **API Gateway** | Horizontal: add instances behind load balancer; stateless |
| **Worker Pool** | Horizontal: scale worker count per node; bounded by `MAX_WORKERS_PER_QUERY` per request |
| **Qdrant** | Shard by document collection; replicate per shard for read throughput |
| **PostgreSQL** | Read replicas for retrieval stages; write path is ingestion-only (low contention) |
| **Redis** | Sentinel for HA; cache eviction via LRU with TTL |
| **Ingestion Pipeline** | Vertical (CPU-bound parsing) + horizontal (parallel per-document) |
| **Embedding Generation** | GPU-accelerated; batch processing; queue-buffered |

### 7.4 Multi-Tenancy (Full Version)

| Requirement | Implementation |
|-------------|----------------|
| Tenant isolation | Separate Qdrant collections + PostgreSQL schemas per tenant |
| Per-tenant quotas | Configurable token budgets, query rate limits, storage caps |
| Cross-tenant prohibition | Retrieval scoped to tenant collection; no cross-tenant data leakage |
| Tenant onboarding | API-driven: create collection, provision schema, configure quotas |

---

## 8. Security Requirements

### 8.1 Authentication & Authorization

| ID | Requirement | Priority |
|----|-------------|----------|
| SEC-1 | API authentication via OAuth 2.0 / API key (configurable) | P0 |
| SEC-2 | Role-based access control (RBAC): admin, analyst, viewer | P0 |
| SEC-3 | Per-document access control lists (ACLs) — retrieval respects user permissions | P1 |
| SEC-4 | Session tokens signed (JWT) with configurable expiry | P0 |

### 8.2 Data Protection

| ID | Requirement | Priority |
|----|-------------|----------|
| SEC-5 | TLS 1.3 for all external and internal communication | P0 |
| SEC-6 | Encryption at rest: PostgreSQL (AES-256), Qdrant (disk encryption), Redis (encrypted RDB) | P0 |
| SEC-7 | PII detection during ingestion with configurable redaction policy | P1 |
| SEC-8 | No document content stored in logs — only chunk IDs and metadata | P0 |
| SEC-9 | Embedding vectors are non-invertible; raw text reconstruction from vectors must not be feasible | P0 |

### 8.3 Audit & Compliance

| ID | Requirement | Priority |
|----|-------------|----------|
| SEC-10 | Immutable audit log: query, user, timestamp, answer, chunks accessed, confidence | P0 |
| SEC-11 | Data residency controls: configurable deployment region for all storage layers | P1 |
| SEC-12 | Data retention policy: configurable TTL per document with automated cleanup | P1 |
| SEC-13 | SOC 2 Type II readiness — controls mapped to trust service criteria | P2 |

### 8.4 Network Security

| ID | Requirement | Priority |
|----|-------------|----------|
| SEC-14 | Internal services communicate via private network / service mesh only | P0 |
| SEC-15 | API gateway rate limiting: per-user and per-IP | P0 |
| SEC-16 | Input sanitization: all query inputs validated and length-bounded | P0 |
| SEC-17 | No outbound network calls from workers (LLM API excepted, via egress proxy) | P0 |

---

## 9. Cost Constraints

### 9.1 Per-Query Cost Targets

| Mode | Target Cost (USD) | Max Cost (Hard Cap) |
|------|-------------------|---------------------|
| Light Mode | $0.005 – $0.015 | $0.03 |
| Heavy Mode | $0.02 – $0.05 | $0.10 |

### 9.2 Token Budget Constraints

| Parameter | Value |
|-----------|-------|
| `MAX_TOKEN_BUDGET` per query | 32,000 tokens |
| Light Mode typical usage | 4,000 – 8,000 tokens |
| Heavy Mode typical usage | 16,000 – 28,000 tokens |

**Budget Allocation by Phase (Heavy Mode):**

| Phase | Share | Tokens |
|-------|-------|--------|
| Query analysis | 5% | 1,600 |
| HyDE (if triggered) | 3% | 960 |
| Workers (total) | 60% | 19,200 |
| Synthesis | 12% | 3,840 |
| Adjudication | 10% | 3,200 |
| Composition | 10% | 3,200 |

### 9.3 Organizational Guardrails

| Guardrail | Default | Behavior on Breach |
|-----------|---------|-------------------|
| Per-query ceiling | 32,000 tokens | Hard stop → partial answer |
| Per-user hourly limit | 500 queries | 429 Too Many Requests |
| Per-org daily budget | $100 | Disable Heavy mode; Light only |
| Monthly spend alert | $2,000 | Alert ops; optionally degrade to Level 1 |

### 9.4 Infrastructure Cost Targets (Monthly, Full Version)

| Component | Target Range |
|-----------|-------------|
| Qdrant cluster (3-node) | $400 – $800 |
| PostgreSQL (primary + 2 replicas) | $300 – $600 |
| Redis (3-node sentinel) | $100 – $200 |
| Compute (gateway + workers + services) | $600 – $1,200 |
| LLM API costs | Variable (dominated by query volume) |
| **Total infrastructure (excl. LLM)** | **$1,400 – $2,800/month** |

---

## 10. Evaluation Metrics (Offline + Online)

### 10.1 Offline Evaluation

Run against a curated evaluation dataset of human-judged question-answer-evidence triples.

| Metric | Definition | Target |
|--------|-----------|--------|
| **Answer Accuracy** | % of answers judged correct by human evaluators | ≥ 85% |
| **Citation Precision** | % of cited spans that actually support the claim | ≥ 90% |
| **Citation Recall** | % of relevant spans cited (vs. available in corpus) | ≥ 75% |
| **Faithfulness** | % of answer claims traceable to retrieved chunks (no hallucination) | ≥ 95% |
| **Coverage** | % of query fact-components answered | ≥ 80% |
| **Retrieval Precision@K** | % of top-K chunks relevant to query | ≥ 70% at K=10 |
| **Retrieval Recall@K** | % of all relevant chunks appearing in top-K | ≥ 85% at K=25 |
| **Conflict Detection Recall** | % of true contradictions detected by adjudication engine | ≥ 80% |
| **Confidence Calibration (ECE)** | Expected calibration error across confidence bins | ≤ 0.05 |
| **Insufficient-Evidence Precision** | When system returns "Insufficient evidence," % of time this is correct | ≥ 90% |

### 10.2 Online Evaluation

Measured in production on live traffic.

| Metric | Definition | Target |
|--------|-----------|--------|
| **User Acceptance Rate** | % of answers accepted without follow-up clarification | ≥ 75% |
| **Escalation Rate** | % of queries escalating Light → Heavy | ≤ 25% |
| **Clarification Request Rate** | % of queries where system asks for user clarification | ≤ 10% |
| **P50 Latency** | Median end-to-end response time | ≤ 3 s (Light) / ≤ 10 s (Heavy) |
| **P95 Latency** | 95th percentile response time | ≤ 5 s (Light) / ≤ 22 s (Heavy) |
| **Cost Per Query** | Average LLM + infra cost per query | ≤ $0.03 (blended) |
| **Cache Hit Rate** | % of queries served from Redis cache | ≥ 30% |
| **Worker Failure Rate** | % of worker invocations returning error (not INSUFFICIENT) | ≤ 2% |
| **Conflict Rate** | % of Heavy queries with detected contradictions | Monitored (no target — used for quality dashboarding) |
| **Confidence Distribution** | Histogram of confidence scores | Reviewed weekly for drift |

### 10.3 Calibration Protocol

- **Evaluation set**: 200 human-judged QA pairs, refreshed quarterly.
- **Cadence**: Weekly calibration job computes ECE across 10 confidence bins.
- **Action**: If ECE > 0.05, adjust confidence weight vector. If ECE > 0.10, trigger manual review.
- **A/B framework**: New weight vectors tested on 10% of traffic before full rollout.

---

## 11. Failure Handling

### 11.1 Failure Taxonomy

| Category | Examples | Severity | User Impact |
|----------|----------|----------|-------------|
| **Transient** | DB timeout, network error, LLM rate limit | Low | Retried automatically; no user impact if retry succeeds |
| **Degraded** | Worker failure, low recall, partial synthesis | Medium | Answer returned with warnings or reduced confidence |
| **Logic** | Invalid SQO, ungrounded entities, circular conflicts | Medium | User-facing warning or clarification request |
| **Catastrophic** | OOM, corrupted index, auth system down | High | 503 with retry-after header; incident alert |

### 11.2 Retry Policies

| Dependency | Max Retries | Backoff Strategy | Circuit Breaker Threshold |
|-----------|-------------|-------------------|--------------------------|
| PostgreSQL | 3 | Exponential: 100 ms × 2^n + jitter | 5 failures / 30 s → 60 s recovery |
| Qdrant | 2 | Exponential: 200 ms × 2^n | 5 failures / 30 s → 60 s recovery |
| LLM API | 2 | Exponential: 500 ms × 2^n | 3 failures / 15 s → 30 s recovery |
| Redis | 0 | None (cache is non-critical) | 10 failures / 60 s → 30 s recovery |
| Worker dispatch | 1 | Immediate re-dispatch | N/A (individual worker failures absorbed) |

### 11.3 Graceful Degradation Ladder

```
Level 0 — NOMINAL
  │  All systems healthy. Full pipeline available.
  │  Heavy mode enabled.
  ▼
Level 1 — DEGRADED
  │  Trigger: LLM circuit breaker tripped OR Qdrant latency > 3×baseline
  │  Action: Force Light mode. Reduce top-K to 5. Skip adjudication.
  ▼
Level 2 — MINIMAL
  │  Trigger: Qdrant unavailable OR LLM fully down
  │  Action: Metadata-only retrieval. Direct answer from cached results
  │          or minimal LLM call with caveats. No workers.
  ▼
Level 3 — OFFLINE
     Trigger: PostgreSQL unavailable OR auth system down
     Action: Return cached answers if available.
             Otherwise, 503 Service Unavailable with Retry-After header.
```

### 11.4 Timeout Cascade

| Phase | Soft Timeout | Hard Timeout | Action on Hard Breach |
|-------|-------------|-------------|----------------------|
| Query analysis | 500 ms | 2,000 ms | Use raw query as fallback SQO |
| Retrieval (all stages) | 3,000 ms | 5,000 ms | Use whatever candidates found so far |
| Worker pool | 10,000 ms | 15,000 ms | Collect completed workers; discard pending |
| Synthesis | 2,000 ms | 3,000 ms | Pass raw worker outputs to composer |
| Adjudication (per round) | 2,000 ms | 3,000 ms | Accept current state; skip remaining rounds |
| Composition | 1,500 ms | 2,000 ms | Return structured evidence without prose |

### 11.5 Error Response Contract

All error responses follow a consistent schema:

```json
{
  "error": {
    "code": "RETRIEVAL_TIMEOUT | LLM_UNAVAILABLE | INSUFFICIENT_EVIDENCE | ...",
    "message": "Human-readable description",
    "query_id": "uuid",
    "partial_result": { "...if available..." },
    "retry_after_ms": 5000,
    "degradation_level": 1
  }
}
```

---

## 12. MVP vs Full Version Breakdown

### 12.1 Feature Matrix

| Feature | MVP (v3.0-alpha) | Full (v3.0) |
|---------|:-:|:-:|
| **Ingestion: PDF, DOCX, MD** | ✅ | ✅ |
| **Ingestion: HTML, CSV** | ❌ | ✅ |
| **Structural parsing** | ✅ (section-level) | ✅ (sentence-level) |
| **Table extraction** | ❌ | ✅ |
| **Noise removal** | Basic | Advanced (ML-based) |
| **Adaptive chunking** | ✅ | ✅ |
| **Ontology normalization** | Abbreviation resolution only | Full: abbreviation + synonym + standardization |
| **Semantic embeddings** | ✅ | ✅ |
| **Entity vectors** | ✅ | ✅ |
| **HyDE embeddings** | ❌ | ✅ |
| **Constraint metadata** | Temporal + regulatory | All 4 dimensions |
| **Embedding versioning** | Manual re-index | Zero-downtime shadow swap |
| **Query intent classification** | fact / comparison / synthesis | + compliance |
| **Query decomposition** | ✅ | ✅ |
| **Decomposition validation** | Ambiguity detection only | Full: ambiguity + grounding + confidence scoring |
| **Hybrid retrieval (3-stage)** | ✅ (metadata → entity → vector) | ✅ |
| **Fallback retrieval (Stage 4)** | ❌ | ✅ |
| **Dynamic K** | Fixed K=10 | Full dynamic K ∈ [5, 12] with ε-stop |
| **Light Mode** | ✅ | ✅ |
| **Heavy Mode** | ✅ | ✅ |
| **Light → Heavy escalation** | Manual (user override) | Automatic (confidence-triggered) |
| **Parallel workers** | ✅ (max 8) | ✅ (max 12) |
| **Worker early termination** | ❌ | ✅ |
| **Cross-chunk synthesis** | ✅ | ✅ |
| **Conflict detection** | ✅ | ✅ |
| **Adjudication (re-verification)** | 1 round | 2 rounds |
| **Evidence-bound composition** | ✅ | ✅ |
| **Confidence scoring** | ✅ | ✅ |
| **Confidence calibration** | Manual review | Automated weekly ECE calibration |
| **Session memory** | ✅ (single-turn context carry) | ✅ (full multi-turn) |
| **Redis cache** | ✅ | ✅ |
| **Kafka queue** | ❌ (in-memory queue) | ✅ |
| **Multi-tenancy** | Single tenant | Full multi-tenant with isolation |
| **RBAC** | API key auth | OAuth 2.0 + RBAC + document ACLs |
| **PII detection** | ❌ | ✅ |
| **Audit logging** | Basic (query + answer) | Full immutable audit trail |
| **Observability** | Prometheus + basic dashboards | OpenTelemetry + Grafana + alerting |
| **Circuit breakers** | ✅ | ✅ |
| **Degradation ladder** | Level 0 + Level 3 only | All 4 levels |
| **Cost tracking** | Per-query token logging | Full per-query + per-user + per-org budgets |
| **A/B testing framework** | ❌ | ✅ |

### 12.2 MVP Scope Summary

> [!IMPORTANT]
> The MVP delivers a **functional, evidence-bound RAG system** with hybrid retrieval, parallel workers, conflict detection, confidence gating, and Light/Heavy mode — but with simplified configuration, single-tenant operation, and reduced operational tooling.

**MVP delivers:**
- Core 13-layer pipeline (simplified at edges)
- 3-stage hybrid retrieval (no fallback expansion)
- Parallel workers (up to 8) with citation enforcement
- Cross-chunk synthesis + 1-round adjudication
- Confidence-gated answers
- Light and Heavy mode with manual override
- Basic auth, observability, and cost tracking

**MVP defers:**
- HyDE embeddings, table extraction, HTML/CSV ingestion
- Zero-downtime embedding model upgrades
- Automatic Light→Heavy escalation
- Multi-tenancy, OAuth + RBAC, document ACLs, PII detection
- Full degradation ladder (levels 1 & 2)
- Kafka-based async queue
- Confidence calibration automation + A/B framework

### 12.3 Milestone Plan

| Milestone | Scope | Target |
|-----------|-------|--------|
| **M0 — Foundation** | Infrastructure setup, DB schemas, ingestion pipeline (PDF + DOCX + MD) | Week 1–3 |
| **M1 — Retrieval** | Expert profiling, 3-stage hybrid retrieval, dynamic K (fixed K=10), query analysis | Week 4–6 |
| **M2 — Workers** | Worker agent pool (max 8), evidence-bound composition, Light/Heavy mode | Week 7–9 |
| **M3 — Deliberation** | Cross-chunk synthesis, conflict detection, 1-round adjudication, confidence engine | Week 10–12 |
| **M4 — MVP Release** | API gateway, basic auth, observability, cost tracking, integration tests | Week 13–14 |
| **M5 — Full: Retrieval+** | HyDE, Stage 4 fallback, dynamic K with ε-stop, auto-escalation | Week 15–17 |
| **M6 — Full: Ops+** | Multi-tenancy, OAuth+RBAC, full degradation ladder, Kafka, PII detection | Week 18–21 |
| **M7 — Full: Quality+** | Confidence calibration automation, A/B framework, advanced ingestion (HTML, CSV, tables) | Week 22–24 |
| **M8 — v3.0 GA** | Load testing, security audit, documentation, SOC 2 readiness | Week 25–26 |

---

*HelioX RAG 3.0 — Product Requirements Document v1.0*
*Generated: 2026-03-03*
