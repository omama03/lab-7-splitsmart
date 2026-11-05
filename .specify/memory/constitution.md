# SplitSmart Constitution

## Core Principles

### I. Java 21 First
All production code and tests MUST compile and run on Java SE 21 (`--release 21`). Sources MUST avoid deprecated APIs and MUST use modern language features (records, sealed types, pattern matching, switch expressions) where they improve clarity. No code may target an older Java level without an explicit, documented amendment.

### II. Maven as the Single Build System
The project MUST be built exclusively with Maven (`mvn`). `pom.xml` MUST set `maven.compiler.release=21`, declare all dependencies (JUnit 5, Swing only ships with the JDK), and make `mvn test` the reproducible quality gate on any machine. IDE-specific build settings MUST NOT be required to compile, test, or package the project.

### III. Java Swing User Interface
The MVP user interface MUST be implemented with Java Swing. All user-facing screens (login, groups, expenses, balances, payments, reports, approvals) MUST live in Swing windows/panels. No third-party UI frameworks or web frontends may be introduced into the MVP without an amendment to this constitution.

> **Amendment (2026-11-05, per ADR-001 in `requirements.md`):** the Swing mandate applies to Release 1. The web REST API described in `specs/001-splitsmart-architecture/final-architecture.md` (Spring Boot + React, adopted from the alternative architecture) is the sanctioned Release 2/3 user interface. All other principles, in particular VIII (BigDecimal money) and IX (traceability), apply unchanged to the Release 2 web architecture.

### IV. JUnit 5 Testing (NON-NEGOTIABLE)
All business logic MUST be covered by JUnit 5 (Jupiter) tests executed through the Maven Surefire plugin. `mvn test` MUST pass before any feature is considered done. JUnit 4 vintage tests are not permitted. Tests MUST address validation, splitting arithmetic, balance and payment accounting, and approval transitions.

### V. PlantUML Requirement Modeling
All UML diagrams (use case diagram, class diagrams, sequence diagrams) MUST be maintained as PlantUML (`.puml`) sources inside the repository. Generated images MUST be reproducible from these sources at any time. Specifications in this repository MUST be traceable to PlantUML models, and the models MUST remain consistent with `spec.md` and `requirements.md`.

### VI. Object-Oriented Design
The SplitSmart domain (users, groups, expenses, approvals, payments, reports) MUST be modeled with well-encapsulated object-oriented classes following SOLID principles. Mutable global state and procedural code blobs are forbidden. Responsibilities MUST sit in the correct layer: domain logic in model/service classes, interaction logic in controllers/UI handlers.

### VII. Separation of GUI and Business Logic
The codebase MUST be split into a UI layer (`ui`, Swing-only) and a business layer (`model`/`service`, plain Java). Swing components MUST NOT appear in business classes, and business rules (splitting, rounding, balances, approvals) MUST NOT be written inside event listeners. Every business computation MUST be runnable and testable without constructing any Swing component.

### VIII. BigDecimal for All Money
Monetary amounts MUST be represented and calculated with `java.math.BigDecimal` (scale set explicitly, e.g. 2) — never with `double` or `float`. Every rounding/parsing rule (input parsing, equal split with remainder, percentage splits that do not sum to 100%) MUST be explicit, documented in the specification, and tested in JUnit 5.

### IX. Requirement Traceability
Every functional and non-functional requirement MUST carry a stable identifier (FR-001, FR-002, NFR-001, …) in `requirements.md`/`spec.md`, MUST trace to a numbered customer feature in `UserRequirement.md`, and MUST trace forward to test cases and PlantUML models. No customer feature may be dropped or silently reinterpreted: the traceability matrix MUST show each requirement mapped to its source feature in `UserRequirement.md`.

### X. Human Review of AI Output
Any artifact produced with AI assistance (constitution, specifications, clarifications, models, code) MUST be reviewed by a human owner before acceptance. Reviewers MUST verify each artifact against `UserRequirement.md` and this constitution, and MUST record acceptance in the review notes/commits. Unreviewed AI output MUST NOT be merged, committed as final, or trusted for grading decisions.

## Customer Requirements vs. Implementation Decisions

`UserRequirement.md` is the authoritative record of the **customer request** and MUST be treated as read-only: no command, agent, or developer may edit, reformat, or extend it. Everything else — Java Swing as the MVP technology, the exact split/rounding algorithms, approval chain design, data storage — is an **implementation decision** and MUST be documented in `requirements.md`, `spec.md`, and downstream artifacts. Implementation documents MUST clearly state which requirements are satisfied by the Java Swing MVP and which customer expectations (web, Android, iOS, notification system) are deferred to later releases rather than silently omitted.

## Quality Gates and Development Workflow

1. **Baselining**: `/speckit-specify` MUST produce a testable specification from `UserRequirement.md`; ambiguities MUST go through `/speckit-clarify` and be resolved *after* human confirmation — never silently invented.
2. **Test-first**: JUnit 5 tests are written and reviewed against acceptance criteria before implementation.
3. **Compliance check**: each commit MUST pass `mvn test`; each release of the specification MUST be checked manually (or with an AI tool) for accidental changes to `constitution.md`, `requirements.md`, and `spec.md`.
4. **Traceability review**: reviewers MUST verify requirement IDs, the source-feature mapping in `UserRequirement.md`, and the updated PlantUML models.
5. **AI review**: AI-generated changes are only complete after a named human has reviewed and approved them.

## Governance

This constitution supersedes all other project practices and documents for SplitSmart. Amendments MUST be proposed, reviewed by a human owner, documented with a version increment (MAJOR for principle removal/redefinition, MINOR for additions/expansions, PATCH for clarifications), and recorded with a ratification/amendment date. All pull requests and reviews MUST verify compliance with the principles above; non-compliance blocks merge. Runtime development guidance lives in `.specify/memory/` and the specs directory, not in this document.

**Version**: 1.0.0 | **Ratified**: 2026-10-29 | **Last Amended**: 2026-10-29