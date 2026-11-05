# ChatGPT Review — Report 2: Claude Code Alternative Architecture

**Reviewed by**: ChatGPT (GitHub connector, read access to `omama03/lab-7-splitsmart`)
**Document under review**: `specs/001-splitsmart-architecture/alternative-architecture.md` (compared against `plan.md`)
**Date**: 2026-11-05

---

## What I reviewed

I read the alternative architecture produced with Claude Code: a three-tier web system — React 18 SPA, Spring Boot 3 REST API in Java 21, PostgreSQL 16 — with an endpoint map covering every functional requirement, JWT auth, and a deployment view (single JAR + Docker Compose local Postgres).

## Strengths

1. **It actually delivers F9.** The customer's "primary web-based" requirement is met for real, not deferred. Any two group members can share live data through the API — the baseline cannot do this at all.
2. **Business rules stay single-sourced.** Split remainder, 100% percentage validation, approval state machine, payment caps — all stated to live in the same `service` layer as the baseline, behind DTO validation. This is the key architectural insight: the *domain* is portable; only the presentation/delivery changes.
3. **Endpoint map is traceable.** The resource table maps each route to FR IDs (FR-001…FR-015), so the plan's Principle IX survives the rewrite. Money is `NUMERIC(12,2)` — consistent with the BigDecimal principle.
4. **Honest conflict declaration.** The document itself flags the Principle III (Swing-only) violation instead of hiding it, and routes the resolution to the review/consolidation step. That is the correct process behavior.
5. **Deployment reality**: docker-compose for Postgres plus a single Boot JAR is genuinely how such an app ships; it is not hand-waving.

## Weaknesses / Risks

1. **Constitution conflict is one amendment away from chaos.** Principle III exists to keep the MVP shippable on the agreed timeline. Adopting the alternative wholesale means amending the constitution *and* expanding the delivery scope (two languages, DB operations, auth/CORS) mid-course.
2. **Doubles the build surface.** Maven + npm, Java + TypeScript, JUnit + Vitest — for a class MVP, roughly double the moving parts, and every test now has an HTTP seam (`MockMvc`) rather than plain method calls.
3. **Operational prerequisites appear**: Postgres must run somewhere, password hashing and token handling must be correct, and a process-crash story replaces the simple file store. Some of this (BCrypt, token expiry) is unstated in the document — it should be explicit.
4. **No offline story at all** — a real regression versus the baseline for a laptop-only demo.

## Dimension Scores (1–5)

| Dimension | Score | Comment |
|-----------|-------|---------|
| Constitution compliance | 2 | VI/VIII/IX hold; III broken outright |
| Requirements coverage (F1–F8, Release-1 subset) | 5 | full, plus real multi-user |
| Requirements coverage (F9) | 5 | fully delivered |
| Simplicity / delivery risk | 2 | two builds, DB, auth |
| Extensibility toward web/mobile | 5 | API is already the Release 3 contract |
| Testability | 4 | strong for services; UI tests cheaper to skip |

**Overall: 3.8 / 5** — higher ceiling, higher floor-of-risk; the customer-facing gap it closes is real.

## Recommendation

Do not choose either document as-is. The strongest consolidated outcome is: **keep the baseline's Constitution discipline and service tests for Release 1, and formalize this alternative as the Release 2 target architecture** — the team should document the amendment path (Principle III becomes "Release 1 Swing; Release 2 web API") rather than quietly switching platforms. My consolidated instructions after your decision (task 6) will follow whichever direction you choose.
