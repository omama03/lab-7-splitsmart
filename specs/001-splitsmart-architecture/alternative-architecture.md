# SplitSmart — Alternative Architecture

**Produced with**: Claude Code (task 4 of Lab 7) — alternative to the Spec Kit baseline in [plan.md](plan.md)
**Date**: 2026-11-05

The customer's request says the primary delivery is **web-based** (F9). The Spec Kit baseline keeps the Release 1 desktop Swing monolith only. This document takes the customer requirement seriously and describes a competing architecture built around a web back end, so the two can be reviewed side by side.

## 1. Architecture Style

Three-tier web application with a REST API in the middle:

```text
┌──────────────┐   HTTPS/JSON    ┌──────────────────┐   JDBC    ┌──────────────┐
│  React SPA   │ ◄─────────────► │  Spring Boot     │ ◄───────► │  PostgreSQL  │
│  (browser)   │                 │  REST API        │           │  16          │
└──────────────┘                 │  + domain layer  │           └──────────────┘
                                 └──────────────────┘
```

- **Presentation tier**: React 18 single-page app. Screens: login/registration, groups, expense entry, approval inbox, balances, payments, reports.
- **Application tier**: Spring Boot 3 (Java 21) exposing a REST API. All business rules (split arithmetic, approval transitions, balance computation, payment caps) live in `@Service` beans — the same rules, one implementation, any future client.
- **Data tier**: PostgreSQL 16. Persisted schema mapped with Spring Data JPA. Money stored as `NUMERIC(12,2)` — matches the `BigDecimal` principle.

## 2. Technical Context

| Aspect | Choice |
|--------|--------|
| Language | Java 21 (API), TypeScript (frontend) |
| Backend framework | Spring Boot 3.3 (Web, Data JPA, Validation, Security) |
| Frontend | React 18 + Vite, fetch-based API client |
| Storage | PostgreSQL 16 (JPA/Hibernate) |
| Auth | Email + password, JWT session token (BCrypt-hashed passwords) |
| Testing | JUnit 5 + MockMvc for the API; Vitest for frontend components |
| Build | Maven (`server/`) and npm (`web/`) — two builds |
| Deployment | One JAR + static frontend bundle; `docker-compose` for local Postgres |

## 3. Module Breakdown (server side only — mirrors the baseline's layers)

```text
server/src/main/java/splitsmart/
├── api/          # @RestController per resource: AuthController, GroupController,
│                 # ExpenseController, ApprovalController, PaymentController, ReportController
├── service/      # same services as baseline: ExpenseService (splits), BalanceService,
│                 # ApprovalService (state machine), PaymentService, ReportService
├── domain/       # JPA entities: User, Group, Expense, SplitLine, Payment, Notification
├── repo/         # Spring Data JPA repositories
└── dto/          # request/response records; validation annotations carry FR-009 messages
```

REST resource map (all JSON, JWT-protected except auth):

| Resource | Endpoints | Serves |
|----------|-----------|--------|
| `/auth` | register, login, password reset | FR-001, FR-002 |
| `/groups` | CRUD, membership, owners | FR-003, FR-004 |
| `/expenses` | create (equal % split validation), list, detail | FR-005–FR-009 |
| `/approvals` | pending queue, approve/reject, multi-level chain | FR-011, FR-004 |
| `/balances` | pairwise owed matrix | FR-012 |
| `/payments` | record settlement, partial payments | FR-013 |
| `/reports` | summaries, custom date-range query | FR-014, FR-015 |
| `/notifications` | in-app inbox, GET/READ | FR-010 |

Notification delivery stays in-app (inbox endpoint) in Release 1, exactly as clarified in Lab 6 — email/push deferred.

## 4. Key Behavior Mapping (unchanged from Lab 6 decisions)

- Equal split remainder: half-up to cents, leftover cents to first participants in list order — implemented once in `ExpenseService`, unit-tested.
- Percentage split must total exactly 100% — validated by the DTO before the service runs; rejected with a specific message (FR-007/FR-009).
- Approvals: expense state machine `PENDING → APPROVED | REJECTED`; balances adjust only on APPROVED transition, in the same transaction.
- Pairwise balances derived from approved expenses minus recorded payments; payment capped at the owed balance (FR-013).
- Audit trail: every lifecycle event writes actor + timestamp + outcome (NFR-005) — natural in a server (request log + events table).

## 5. Trade-offs vs. the Base Architecture

**For this architecture**

- F9 "primary web-based" is actually met: no JVM install for end users, works from any browser.
- Multi-user by construction — two roommates share one live database; the desktop monolith would need file sharing or replication to do this at all.
- The service layer survives into Release 2/3 unchanged; Android/iOS (Release 3) would reuse the same REST API — no rewrite of business rules.
- Central persistence means real concurrent access, backups, and easier reporting queries (SQL) instead of loading the whole JSON store.

**Against it**

- Violates Constitution Principle III (MVP must be Swing) — adopting it requires a constitution amendment; this is unavoidable if we want to make F9 real.
- Two builds, two languages (Java + TypeScript), a database to install — heavier than the baseline for a student project and for the MVP timeline.
- Requires hosting/server to be useful at all; the baseline runs on one laptop with zero infrastructure.
- JWT auth, CORS, API error handling are extra scope the baseline never pays.

## 6. Deployment View

- Development: `docker-compose up` (Postgres) + `mvn spring-boot:run` + `npm run dev`.
- Production: single Spring Boot JAR serving the built React bundle from `static/`; Postgres either local Docker or a small managed instance. TLS terminated by a reverse proxy.

## 7. Compatibility Note

This architecture keeps the constitution's money (BigDecimal/NUMERIC), testing (JUnit 5), traceability, and layering principles intact. The single hard conflict is Principle III (Swing-only UI) — resolving it is exactly what the ChatGPT review step (task 5) and the consolidated instructions (task 6) are for.
