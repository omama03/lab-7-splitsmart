# SplitSmart — Design Document (Release 1)

**Design of record** after the Lab 7 architecture consolidation (ADR-001 in [`requirements.md`](../requirements.md)): base Swing monolith now, service layer written so it lifts into the Release 2 REST API unchanged.
**UML models**: this directory, plain PlantUML, renderable per constitution Principle V.
**Date**: 2026-11-05

## 1. Scope

Release 1 design for F1–F8 as their Java Swing release subset. F9 (web) is a Release 2 delivery — this design supports it only at the boundaries (repository seam, service layer, audit log), not in the UI.

## 2. Domain model

![Class diagram](class-diagram.puml)

Source: [`class-diagram.puml`](class-diagram.puml). Decisions visible in the model:

- `Expense *- SplitLine` captures *what each participant eventually owes for this expense*, separate from the expense total; equal/percentage services only produce different `SplitLine` distributions.
- `ApprovalState` on `Expense` makes the FR-011 rule structural: balances look exclusively at approved expenses.
- `Group.approvalLevels` makes FR-004 (single or multiple review levels) a data property, not branching code.
- `AuditEvent` (NFR-008 via ADR-001) is a first-class domain object — the Release 2 events table seeds from it — rather than console logging.
- All money attributes are `BigDecimal` scale 2 (NFR-001, NFR-007). No Swing type appears anywhere in the model or services (Principle VII).

## 3. Dynamic views

### 3.1 Expense lifecycle (FR-005–FR-011, FR-010, FR-012, NFR-008)

Source: [`activity-expense.puml`](activity-expense.puml). Flow: entry → input validation (FR-009) → split computation (FR-006 remainder rule / FR-007 exactly-100% rule) → PENDING → multi-level approval (FR-004/FR-011) → balance recompute (FR-012, NFR-003) → in-app notifications (FR-010) → audit events at every transition.

### 3.2 Settlement (FR-013)

Source: [`activity-payment.puml`](activity-payment.puml). Flow: select pairwise debt → validate cap (payment ≤ owed) → record full or partial payment → exact balance reduction → audit event.

## 4. Design rules

1. **Service layer owns rules.** Split arithmetic, approval transitions, payment caps live in `splitsmart.service`; UI listeners only call services. Every service is testable without Swing.
2. **Persistence is a seam.** Services see `UserRepository`-style interfaces in `splitsmart.persistence`; `JsonFile*` implementations are the Release 1 adapters. The Release 2 JPA repositories implement the same interfaces (ADR-001).
3. **Money discipline.** Anything named `amount`/`share`/`balance` is `BigDecimal` with explicit scale; parsing/rounding rules are the documented Lab 6 clarifications; NFR-007 keeps scale 2 in storage.
4. **Audit on transitions.** Created / approved / rejected / payment recorded each emit one `AuditEvent` (NFR-005, NFR-008) — in a Release 2 service, the identical call writes to the events table.
5. **Single-sourced remainder rule.** `ExpenseService` owns it; both the UI path and any future REST path call the same method.

## 5. Traceability

| Design element (above) | Requirement |
|------------------------|-------------|
| `User`, `AuthService`, register/login | FR-001, FR-002 |
| `Group`, `approvalLevels`, `GroupService` | FR-003, FR-004 |
| `Expense`, `SplitLine`, `createExpense`, validation | FR-005, FR-008, FR-009 |
| `equalSplit` remainder rule | FR-006 |
| `percentageSplit` exactly-100% rule | FR-007 |
| `ApprovalState`, `ApprovalService`, expense lifecycle diagram (approve/reject branches) | FR-011 |
| `BalanceService` pairwise matrix, balance-recompute activity | FR-012, NFR-003 |
| `Payment`, `PaymentService`, settlement diagram | FR-013 |
| `ReportService` summary/custom range | FR-014, FR-015 |
| `Notification`, `NotificationService` in-app | FR-010 |
| REST/React/PostgreSQL path in final architecture §3 | FR-016 (Release 2) |
| `AuditEvent`, audit steps in both diagrams | NFR-005, NFR-008 |
| `BigDecimal` money attributes, storage scale-2 rule | NFR-001, NFR-007 |

No FR-001…FR-016 or NFR-001…NFR-008 id is left without a design element; no design element exists without a requirement — re-run check from Instruction 4.3 passed 2026-11-05.
