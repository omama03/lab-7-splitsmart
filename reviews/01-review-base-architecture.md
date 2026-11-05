# ChatGPT Review — Report 1: Spec Kit Base Architecture

**Reviewed by**: ChatGPT (GitHub connector, read access to `omama03/lab-7-splitsmart`)
**Document under review**: `specs/001-splitsmart-architecture/plan.md` (with `spec.md` context)
**Date**: 2026-11-05

---

## What I reviewed

I read the Spec Kit plan produced by the `/speckit-plan` workflow — the baseline architecture for SplitSmart Release 1: a Java 21 Swing desktop monolith, Maven build, JUnit 5 tests, JSON file persistence, packages split into `model / service / persistence / report / ui`.

## Strengths

1. **Constitution fit is complete.** All nine principles in `.specify/memory/constitution.md` are marked PASS and I could verify each claim from the plan text itself: `--release 21`, Maven-only, Swing screens, JUnit 5, PlantUML, layered OO design, GUI/business separation, BigDecimal scale 2, and the FR/NFR traceability back to `UserRequirement.md` F1–F9. Nothing is hand-waved.
2. **Right-sized for Release 1.** For a student-scale MVP delivering F1–F8 in-app, a one-JAR desktop app with file storage is honest engineering: zero deployment, zero servers, everything testable with `mvn test`. The "Complexity Tracking" section correctly reports no violations.
3. **Persistence seam exists.** `persistence/` is interfaces plus JSON implementations. That is a deliberate decoupling point — a Release 2 server could be added by swapping the repository implementations without touching `model` or `service`.
4. **Traceability is preserved**, not restarted: the plan links the Lab 6 `spec.md` and keeps the clarification decisions (remainder rule, exactly-100% percentages, payment caps).

## Weaknesses / Risks

1. **F9 is unmet by design.** The customer asked for a primarily web-based product. The baseline defers web delivery to Release 2 — legitimate, but it means the Release 1 product cannot actually be used by two users on different machines, and the "shared" part of shared expenses is per-file.
2. **JSON whole-store read/write.** Loading every entity at start-up and rewriting files on mutation is fine at hundreds of records, but corrupts risk if the process dies mid-write; there is no locking story for simultaneous access.
3. **Notifications are in-app only** (already clarified, but worth repeating in review): any reviewer should not mistake this for a full notification system.
4. **No concurrency model.** Single user at a time is implied but never stated as a constraint line. It should be.

## Dimension Scores (1–5)

| Dimension | Score | Comment |
|-----------|-------|---------|
| Constitution compliance | 5 | complete, each gate verifiable |
| Requirements coverage (F1–F8, Release-1 subset) | 5 | full |
| Requirements coverage (F9) | 2 | deferred entirely |
| Simplicity / delivery risk | 5 | lowest possible for this scope |
| Extensibility toward web/mobile | 3 | persistence seam exists, UI is throwaway |
| Testability | 5 | headless services, JUnit 5 |

**Overall: 4.2 / 5** — safe, constitution-perfect baseline whose main cost is scope on F9.

## Recommendation

Keep this as the Release 1 vehicle. If the team chooses the web alternative in Report 2, do not discard this plan: its service/persistence separation is exactly the part worth carrying into the consolidated architecture.
