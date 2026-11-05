# ChatGPT Output — Consolidated Instructions (Task 6)

**Produced by**: ChatGPT (GitHub connector, read/write access to `omama03/lab-7-splitsmart`)
**Input given**: [`decision.md`](../decision.md) plus both architecture documents and both review reports
**Date**: 2026-11-05

---

You chose the **base architecture for Release 1 with the alternative adopted as the Release 2 target**. Below are the consolidated instructions. Execute them with Claude Code (or Copilot) as one work package; each instruction states the file(s) it touches. Do not invent new requirements — every change traces to `UserRequirement.md` F1–F9 or to the decisions above.

## Instruction 1 — Update the requirements document (root `requirements.md`)

1.1. Add a header note that this is the Lab 7 revision of the Lab 6 requirements inventory, and keep every FR/NFR row from Lab 6 **unchanged** (IDs are frozen).
1.2. Add a new section **"Architecture Decision Record (ADR-001)"** summarizing: Release 1 = Swing desktop monolith (chosen), Release 2 target = Spring Boot REST + React + PostgreSQL (adopted from the alternative architecture), decision date 2026-11-05, citing `decision.md`.
1.3. Add two new non-functional rows to the table:
    - **NFR-007 (from ADR-001 / alternative §2):** monetary values are stored with fixed scale 2 (file/JSON now, `NUMERIC(12,2)` in the Release 2 schema) so no precision is lost on migration.
    - **NFR-008 (from ADR-001 / alternative §4):** all expense lifecycle events (created, approved, rejected, payment recorded) persist actor + timestamp + outcome in a dedicated audit log structure, seedable into the Release 2 events table.
1.4. In the "Customer vs. Implementation Decisions" section, append: "Architecture split: Release 1 desktop (Swing) is an implementation decision; F9 web delivery remains a customer requirement assigned to Release 2 — see ADR-001."

## Instruction 2 — Final architecture document

2.1. Create `specs/001-splitsmart-architecture/final-architecture.md` containing the **consolidated architecture**: chosen baseline (monolith, packages `model/service/persistence/report/ui`) plus the Release 2 evolution path from the alternative (REST resource map, JWT auth, PostgreSQL/Numeric money, docker-compose deployment view).
2.2. Include a **"Rejected elements"** subsection listing exactly what was *not* adopted from the alternative for Release 1: React frontend, second build (npm), database persistence, JWT/CORS — each with one-line justification from review reports 1 and 2.
2.3. Include the traceability line: every FR-001…FR-016 still maps to its Release assignment as in `requirements.md`.

## Instruction 3 — Design document with UML models

3.1. Create `design/design.md` as the design document of record for Release 1, structured as: scope → domain model (class diagram) → dynamic views (activity diagrams) → design rules → traceability.
3.2. Include the class diagram as PlantUML source in `design/class-diagram.puml`: classes `User`, `Group`, `Expense`, `SplitLine`, `Payment`, `Notification`, `AuditEvent` (for NFR-008), enums `ApprovalState {PENDING, APPROVED, REJECTED}` and `SplitType {EQUAL, PERCENTAGE}`; services `AuthService`, `GroupService`, `ExpenseService`, `BalanceService`, `PaymentService`, `ReportService`, `NotificationService`, `AuditService`; repository interfaces `UserRepository`, `GroupRepository`, `ExpenseRepository`, `PaymentRepository` with `JsonFile*` implementations in `persistence/`. Money fields are typed `BigDecimal` scale 2 (NFR-001/NFR-007). No Swing class may appear in this diagram (Principle VII).
3.3. Include **two activity diagrams** as PlantUML sources in `design/`:
    - `design/activity-expense.puml` — the expense lifecycle of FR-005…FR-011: create expense → validate amount (reject zero/negative/non-numeric, FR-009) → choose split → equal path (remainder rule to first participants) or percentage path (reject unless exactly 100%) → submit → approval loop (PENDING, approve / reject, FR-011) → on approval compute pairwise balances (FR-012) → notify members in-app (FR-010) → write audit event (NFR-008).
    - `design/activity-payment.puml` — settlement of FR-013: select pairwise balance → enter payment → validate ≤ owed → record payment (partial or full) → zero/reduce balance → audit event.
3.4. Every diagram node that affects balances or state transitions must carry the FR/NFR identifier it satisfies, so that Principle IX traceability holds from diagram to code plan.
3.5. Keep all PlantUML sources renderable with plain PlantUML (no C4/stdlib extensions) so images are reproducible (Principle V).

## Instruction 4 — Consistency pass

4.1. Verify `plan.md` now links the final architecture and design; add a line to its Summary naming the decision.
4.2. Add an amendment note to `.specify/memory/constitution.md` Principle III: "Amended 2026-11-05 (ADR-001): the Swing mandate applies to Release 1; the web REST API described in the final architecture is the sanctioned Release 2/3 UI." Do not weaken any other principle.
4.3. Re-run the traceability check: F1–F9 → FR/NFRs → design diagrams all resolve; report any dangling ID instead of fixing silently.

## Definition of Done for this work package

- [ ] `requirements.md` — Lab 7 revision with ADR-001, NFR-007, NFR-008 (Instruction 1)
- [ ] `specs/001-splitsmart-architecture/final-architecture.md` (Instruction 2)
- [ ] `design/design.md`, `design/class-diagram.puml`, `design/activity-expense.puml`, `design/activity-payment.puml` (Instruction 3)
- [ ] Constitution amendment note + `plan.md` decision line (Instruction 4)
- [ ] All commits authored normally in the repo; no tool output pasted verbatim into traceable docs
