# Architecture Decision — Lab 7, Task 6

**Date**: 2026-11-05
**Decision by**: Omama
**Decision fed to**: ChatGPT (to produce the consolidated instructions in [reviews/03-chatgpt-consolidated-instructions.md](reviews/03-chatgpt-consolidated-instructions.md))

## The two candidates

| | Base (Spec Kit plan) | Alternative (Claude Code) |
|---|---|---|
| Shape | Java Swing desktop monolith, JSON files | React + Spring Boot REST API + PostgreSQL |
| Constitution | 9/9 PASS | Principle III violated |
| F9 (web-based) | Deferred to Release 2 | Delivered now |
| Delivery risk | Lowest | Highest (two builds, DB, auth) |
| ChatGPT score | 4.2 / 5 | 3.8 / 5 |

## Decision

**Choose the base architecture**, but adopt three concrete elements from the alternative as documented forward commitments:

1. **Release 1 stays the Swing desktop monolith** with JSON persistence — it passes every constitution gate and is the lowest-risk way to deliver F1–F8 this term.
2. **The Release 2 target architecture is the alternative's web design** (Spring Boot REST + React + PostgreSQL). The service layer must be written now so it can be lifted into that API unchanged; the `ui` package remains disposable.
3. **Small amendments to record**: constitution Principle III gains a scoped Release-2 exception ("Swing is the Release 1 UI; the web REST API is the sanctioned Release 2 UI"), and the plan gains two derived non-functional requirements from the alternative (NUMERIC(12,2)-equivalent money storage rule and the audit event table direction for NFR-005).

## Rationale

- Multi-user sharing is the real product gap of the baseline, but shipping a web platform inside the MVP timeline risks F1–F8 delivery. The repository seam in `persistence/` is the honest middle path: monolith now, API-ready domain.
- Both ChatGPT reviews converged on exactly this hybrid ("do not choose either as-is"), so committing to it required no new evidence.
