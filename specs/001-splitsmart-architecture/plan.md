# Implementation Plan: SplitSmart MVP — Base Architecture

**Branch**: `001-splitsmart-architecture` | **Date**: 2026-11-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-splitsmart-architecture/spec.md`, carried over from the Lab 6 requirements work (`requirements.md`, `UserRequirement.md`).

## Summary

SplitSmart Release 1 is a desktop application for sharing expenses between individuals and groups (roommates, trips, coworkers), covering accounts, groups, expenses with equal/percentage splits, an approval workflow, pairwise balances, payments, and reporting. This plan was produced with the Spec Kit workflow as the **baseline architecture**: a single Java Swing monolith split into model / service / persistence / UI packages, Maven as the only build, JUnit 5 as the quality gate, and file-based JSON persistence. It keeps Release 1 fully offline and defers the web/mobile delivery of F9 to later releases, exactly as decided in Lab 6.

## Technical Context

**Language/Version**: Java SE 21 (`--release 21`), records/sealed types/switch expressions used where they help readability

**Primary Dependencies**: Java Swing (bundled with JDK) for UI, JUnit 5 (Jupiter) + Maven Surefire for tests, Gson (com.google.code.gson) for JSON persistence

**Storage**: File-based JSON store under `~/.splitsmart/` (one `users.json`, `groups.json`, `expenses.json`, `payments.json`, `notifications.json`); loaded into in-memory repositories at start-up and written back on each mutation. No database in Release 1.

**Testing**: JUnit 5 through `mvn test`; every business rule (split remainder, approval transitions, pairwise balances, payment caps) is a unit test

**Target Platform**: Desktop (Windows/macOS/Linux) with a JVM installed

**Project Type**: Single desktop application (monolith)

**Performance Goals**: Approved expense visible in balances within 1 second (NFR-003); first expense recorded in under 3 minutes (NFR-004)

**Constraints**: Works fully offline; monetary values only as `BigDecimal` scale 2 (NFR-001); no Swing code below the `ui` package (Principle VII)

**Scale/Scope**: Single-machine, single-user-at-a-time usage; roughly 10 stateful classes in `model`, 8 service classes, 12 Swing panels/dialogs

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-checked after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Java 21 First | PASS | `--release 21` in `pom.xml` |
| II. Maven as the Single Build System | PASS | only `pom.xml`, no IDE settings required |
| III. Java Swing User Interface | PASS | all screens are Swing windows/panels |
| IV. JUnit 5 Testing | PASS | Surefire wired, business logic covered |
| V. PlantUML Requirement Modeling | PASS | class + activity diagrams as `.puml` in `design/` |
| VI. Object-Oriented Design | PASS | domain modeled as encapsulated classes (SOLID) |
| VII. Separation of GUI and Business Logic | PASS | `ui` vs `model`/`service` only; services testable without Swing |
| VIII. BigDecimal for All Money | PASS | scale 2 everywhere; remainder rule documented |
| IX. Requirement Traceability | PASS | FR/NFR IDs map to `UserRequirement.md` F1–F9 |

## Project Structure

### Documentation (this feature)

```text
specs/001-splitsmart-architecture/
├── plan.md                     # This file (Spec Kit /speckit-plan output = BASE architecture)
├── spec.md                     # feature spec carried from the Lab 6 requirements work
├── alternative-architecture.md # Claude Code output — competing Web-API architecture
└── consolidated-instructions.md# ChatGPT output — merge instructions for the final docs
```

### Source Code (repository root)

```text
src/
├── main/java/splitsmart/
│   ├── model/          # User, Group, Expense, SplitLine, Payment, Notification, ApprovalState
│   ├── service/        # AuthService, GroupService, ExpenseService, BalanceService,
│   │                   # PaymentService, ApprovalService, ReportService, NotificationService
│   ├── persistence/    # Repository interfaces + JsonFileRepository implementations
│   ├── report/         # report builders for summaries and custom date-range reports
│   └── ui/             # Swing frames/panels: LoginFrame, GroupLayout, ExpenseDialog,
│                       # ApprovalPanel, BalancePanel, PaymentDialog, ReportPanel
└── test/java/splitsmart/   # mirrors model/service/persistence/report (JUnit 5, no Swing tests)

design/
├── class-diagram.puml          # domain class diagram
├── activity-expense.puml       # activity diagram: create → validate → approve → balance
└── activity-payment.puml       # activity diagram: settle a pairwise balance
```

**Structure Decision**: Single Maven project. The package split (`model` / `service` / `persistence` / `report` / `ui`) is the smallest structure that satisfies Principles VI and VII, keeps every service testable headless, and leaves room to swap `persistence` for a server-backed repository in Release 2 without touching the domain.

## Complexity Tracking

No constitution violations — nothing to justify. The monolithic single-project structure is the simplest option that meets all nine principles.
