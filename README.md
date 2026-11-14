# Compliance Evidence & Audit Copilot
**Control management, evidence traceability, and cited Q&A — with an evaluation gate.**

> Real product: web UI + API + workers + database (not a CLI, not a chat wrapper).  
> Built for the kind of work behind RegTech/audit platforms: **EY Tunisie** (64 explicitly paid PFE subjects in the dataset: AI assistants for ISMS, SWIFT CSP, PCI DSS, ISO 27001/42001, EU AI Act), Wevioo (*ConformIA*, audit missions), Olindias (risk + document analysis).

## Why this project
Compliance breaks in two ways: evidence is scattered and hard to defend, or “AI assistants” answer confidently without proof. This project treats audit work as **evidence + traceability**: every control maps to evidence, every answer cites a source, and quality is measured by an eval suite that blocks regressions.

## What you can do
- **Control room:** define frameworks and controls (e.g. NIST CSF / ISO 27001 / PCI-style), assign owners, track status, due dates, and findings.
- **Evidence vault:** attach evidence, link it to controls, record provenance and validity windows, and flag stale/missing evidence.
- **Automated checks (where feasible):** pluggable checks that produce pass/fail with evidence alongside manual attestations.
- **Audit-ready reports:** generate scoped reports with control status, linked evidence, gaps, and full history.
- **Evidence-grounded copilot:** ask questions over policies/controls/evidence; responses include **exact citations** and **refuse** when support is insufficient.

## Architecture
```text
[Web UI (React/Next)] -> [FastAPI: controls, evidence, checks, reporting, Q&A]
                                  |
                     [Postgres (+ pgvector): controls, evidence metadata,
                                findings, audit log, chunks/embeddings]
                                  |
                     [Workers: ingestion/chunking, embedding, check runners,
                               report & eval jobs]  <->  [Object storage: files]
```
- Hybrid retrieval (lexical + vector) with reranking; generation constrained to retrieved evidence.
- A separate **eval pipeline** runs a gold set and publishes metrics per build.

## Hard engineering problems
- **Grounding & abstention:** cite the source span; refuse on unsupported queries (no fluent hallucinations).
- **Evidence model & provenance:** versioned controls/evidence, validity, and traceability from report claim → evidence → source.
- **Evaluation as a gate:** gold set (answerable + unanswerable) measuring faithfulness, citation precision/recall, abstention — **CI fails on regression**.
- **Cost/latency & access control:** tracked cost/p95 per query; RBAC over sensitive evidence; append-only audit log.

## Proof (what “done” means)
| Claim | Evidence |
|---|---|
| Grounded answers | Gold set (~150 Qs): faithfulness, citation precision/recall, correctness |
| Knows its limits | Abstention accuracy on unanswerable/undersupported queries |
| No regressions | `make eval` / CI thresholds with diffable metrics history |
| Operable | p95 latency & cost/query; failure taxonomy (retrieval vs generation) |
| Traceable | Report findings deep-link to stored evidence & citations |

## Stack
Python 3.12+, FastAPI, Pydantic v2 · PostgreSQL + pgvector, SQLAlchemy/Alembic · hybrid retrieval + reranker, provider-agnostic LLM interface · workers (Celery/arq + Redis) · React/Next.js · pytest, Testcontainers, eval harness, ruff/mypy, GitHub Actions · Docker Compose.

## Status
**Design / spec-first.** Milestones: `model` → `ingestion & retrieval` → `cited Q&A` → `eval harness & CI gate` → `checks, reporting & RBAC`.
