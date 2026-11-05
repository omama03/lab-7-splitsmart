# Lab 7 — UML Modeling: architecture generation, review, and consolidation

**Project**: SplitSmart (continues Lab 6) · **Course**: CIS 285 · **Due 2026-11-05**

This repository documents the full Lab 7 flow: two competing architectures were produced with different tools, reviewed by ChatGPT, one was chosen, and ChatGPT's consolidated instructions were executed to produce the final requirements / architecture / design documents with UML class and activity diagrams.

## Steps taken (mapping to the lab tasks)

| # | Lab task | What was done | Where the evidence is |
|---|----------|---------------|----------------------|
| 1 | Install Codex or Claude Code in VS Code | Claude Code is installed and in use in my environment (CLI + VS Code integration) | every artifact below; the alternative architecture is its output |
| 2 | Configure ChatGPT connector to GitHub (read/write) | Connected ChatGPT to my GitHub account; it can read this repository and write the review artifacts | review/consolidation artifacts in [`reviews/`](reviews/) |
| 3 | Spec Kit → base architecture | `specify init . --integration copilot` (Spec Kit 1.2.5), plan workflow filled from the Lab 6 spec and constitution | [`specs/001-splitsmart-architecture/plan.md`](specs/001-splitsmart-architecture/plan.md) — Swing desktop monolith baseline |
| 4 | Claude Code → alternative architecture | Claude Code authored a competing design taking the deferred F9 web requirement seriously | [`specs/001-splitsmart-architecture/alternative-architecture.md`](specs/001-splitsmart-architecture/alternative-architecture.md) — React + Spring Boot REST + PostgreSQL |
| 5 | ChatGPT → review both, reports | ChatGPT reviewed each architecture against requirements coverage, constitution fit, delivery risk | [`reviews/01-review-base-architecture.md`](reviews/01-review-base-architecture.md) (base: 4.2/5), [`reviews/02-review-alternative-architecture.md`](reviews/02-review-alternative-architecture.md) (alternative: 3.8/5) |
| 6 | My decision + ChatGPT consolidated instructions | Decision: keep the base for Release 1, adopt the alternative as the Release 2 target (three carry-overs); ChatGPT then issued consolidated instructions | [`decision.md`](decision.md), [`reviews/03-chatgpt-consolidated-instructions.md`](reviews/03-chatgpt-consolidated-instructions.md) |
| 7 | Run the consolidated instructions | Executed with Claude Code → updated requirements, final architecture, design document, UML diagrams, constitution amendment | See below |

## Consolidated (final) deliverables

- [`requirements.md`](requirements.md) — Lab 7 revision: ADR-001, new NFR-007/NFR-008, F9 moved to a Release 2 schedule; FR/NFR IDs unchanged from Lab 6.
- [`specs/001-splitsmart-architecture/final-architecture.md`](specs/001-splitsmart-architecture/final-architecture.md) — chosen architecture + Release 2 evolution + explicitly rejected elements.
- [`design/design.md`](design/design.md) — design of record (domain model, dynamic views, design rules, traceability).
- [`design/class-diagram.puml`](design/class-diagram.puml) — UML class diagram (domain + services + persistence seam).
- [`design/activity-expense.puml`](design/activity-expense.puml) — UML activity diagram: expense lifecycle (entry → validation → split → approval → balances → notifications → audit).
- [`design/activity-payment.puml`](design/activity-payment.puml) — UML activity diagram: settlement of a pairwise balance.
- [Constitution amendment](.specify/memory/constitution.md) — Principle III now scopes the Swing mandate to Release 1 (web API sanctioned for Release 2/3).

## Traceability summary

All 9 customer features (F1–F9 in `UserRequirement.md`) → 16 functional + 8 non-functional requirements → design elements: verified 2026-11-05, table in `design/design.md` §5.
