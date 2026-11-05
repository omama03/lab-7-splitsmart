# SplitSmart — Final Architecture

**Consolidated per**: ChatGPT consolidated instructions, Instruction 2 (execution with Claude Code, Lab 7 task 7)
**Decision**: [`decision.md`](../../decision.md) — base architecture chosen for Release 1; alternative architecture adopted as the Release 2 target
**Date**: 2026-11-05

## 1. What this document is

The single architecture of record for SplitSmart after the Lab 7 review-and-consolidation flow: the chosen monolith, expressed so its service layer survives unchanged into the adopted Release 2 web architecture. Supersedes neither source document — [`plan.md`](plan.md) (baseline) and [`alternative-architecture.md`](alternative-architecture.md) remain the reviewed candidates this consolidates.

## 2. Release 1 — chosen architecture (from the Spec Kit baseline)

Single Maven project, Java 21, Swing desktop UI, JSON file persistence, JUnit 5.

```text
┌─────────────────────────── splitsmart.ui ───────────────────────────┐
│ login · groups · expense entry · approval inbox · balances ·        │
│ payments · reports                    (Swing, disposable)           │
└───────────────────────────────┬─────────────────────────────────────┘
                                │  calls only
┌───────────────────────────────▼─────────────────────────────────────┐
│ splitsmart.service                                                   │
│ AuthService · GroupService · ExpenseService · BalanceService ·       │
│ PaymentService · ApprovalService · ReportService ·                   │
│ NotificationService · AuditService                                   │
│ — all business rules live here; runnable headless; JUnit 5 covered   │
└───────────────────────────────┬─────────────────────────────────────┘
                                │  interfaces
┌───────────────────────────────▼─────────────────────────────────────┐
│ splitsmart.persistence  UserRepository/GroupRepository/…             │
│ implementations: JsonFile*Repository (~/.splitsmart/*.json)          │
│ + splitsmart.report (summary / custom date-range report builders)    │
└──────────────────────────────────────────────────────────────────────┘
```

Key Release 1 properties: offline, single-machine, money as `BigDecimal` scale 2 (NFR-001/NFR-007), audit events recorded per NFR-008, all screens Swing (Principle III as amended — Release 1 scope only).

## 3. Release 2 — adopted evolution (from the alternative architecture)

The repository interfaces of Release 1 become server-side JPA repositories; services move into Spring beans; the Swing UI is replaced by a React SPA over a REST API.

| Concern | Release 1 (now) | Release 2 (target) |
|---------|-----------------|--------------------|
| UI | Swing panels (splitsmart.ui) | React 18 SPA over HTTPS/JSON |
| API | direct service calls in-process | Spring Boot 3 REST: `/auth /groups /expenses /approvals /balances /payments /reports /notifications` |
| Persistence | `JsonFile*` implementations of repository interfaces | PostgreSQL 16 + Spring Data JPA (same interfaces) |
| Money | BigDecimal, JSON scale 2 | BigDecimal ↔ `NUMERIC(12,2)` (NFR-007) |
| Auth | local session in app | JWT (email + password, BCrypt) — FR-001/FR-002 unchanged |
| Audit | `AuditService` writing log records (NFR-008) | same records into an events table (seeds it directly) |
| Deployment | run the JAR on one machine | single Boot JAR serving the React bundle; docker-compose Postgres |

The REST resource map and its FR mapping are the ones from [`alternative-architecture.md`](alternative-architecture.md) §3, adopted verbatim.

## 4. Rejected elements (not adopted for Release 1)

Each rejected element was weighed in the two ChatGPT review reports:

| Rejected element | Source | Justification |
|------------------|--------|---------------|
| React frontend now | Alternative §1 | two languages + second build double delivery risk for the MVP (report 2, weakness 2); Swing ships Release 1 with constitution intact |
| PostgreSQL now | Alternative §1 | requires an installation/ops story for a single-machine MVP (report 2, weakness 3); JSON store meets Release-1 scale; the repository seam absorbs the swap |
| JWT/CORS/auth plumbing now | Alternative §2 | real scope the baseline never pays (report 2, weakness 3); FR-002 semantics stay identical when it arrives |
| Dropping the monolith entirely | — | report 1, strength 2: lowest-risk vehicle for F1–F8; its persistence seam was exactly the part worth carrying forward |

## 5. Traceability

- Customer features F1–F9 → FR-001…FR-016 → release plan: unchanged in [`requirements.md`](../../requirements.md) (F9 now scheduled for Release 2 per ADR-001).
- NFR-007/NFR-008: added by ADR-001, enforced in both the Release 1 store and the Release 2 schema (§3 above).
- UML models of the designed behavior: [`design/design.md`](../../design/design.md) with `class-diagram.puml`, `activity-expense.puml`, `activity-payment.puml`.
- Constitution Principle III amendment (web API sanctioned for Release 2): noted in [`.specify/memory/constitution.md`](../../.specify/memory/constitution.md).
