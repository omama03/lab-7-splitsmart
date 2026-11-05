# Feature Specification: SplitSmart — Shared Expense Management

**Feature Branch**: `001-splitsmart-mvp`

**Created**: 2026-10-29

**Status**: Draft

**Input**: User description: "SplitSmart — an application for sharing expenses between individuals or groups of people (friends, roommates, family members, coworkers), as described in UserRequirement.md — the original customer request."

## Customer Request Source

This specification is derived exclusively from `UserRequirement.md` in the workspace root, which records the original customer request. The numbered features F1–F9 below map 1:1 to the customer's request. Customer requirements are distinguished from implementation decisions: the **Java Swing MVP subset** is an implementation decision made in planning, labeled separately in this spec (see *MVP Scope (Java Swing Subset)*). `UserRequirement.md` is read-only and must not be modified by any workflow in this project.

## Clarifications

### Session 2026-10-29

- Q: How are the payer and the participating users related for each expense? → A: Payer and participants are set independently; the payer may also include themselves as a participant; no user ever owes themselves.
- Q: What are the group membership rules for expenses and removed members? → A: Expenses may involve any registered user (group membership is not mandatory); a removed member keeps their past balances but is excluded from future group expense participant selection.
- Q: When an equal split does not divide into exact cents, how is the remainder assigned? → A: Each share is rounded half-up to cents; leftover cents are assigned to the first participants in participant-list order until exhausted.
- Q: How should invalid expense amounts be handled? → A: Zero, negative, non-numeric, and blank amounts are rejected at entry with a specific message; amounts require exactly two decimal places.
- Q: Must percentages total exactly 100%? → A: Yes — percentages must sum to exactly 100% at submission; otherwise the expense is rejected with a clear message.
- Q: Which authentication method do accounts use? → A: Email + password; session persists until logout; password reset is available.
- Q: How are new-expense notifications delivered in the first release? → A: In-app only (login inbox); email/push channels are deferred to a later release.
- Q: How are payments recorded? → A: Partial payments are allowed; a payment cannot exceed the currently owed pairwise balance; each payment reduces that specific pairwise debt.

## Actors

| Actor | Description |
|-------|-------------|
| **User** (Member) | Any registered person who logs in to log expenses, view balances, record payments, and run reports. |
| **Group Member** | A User who belongs to one or more Groups and shares expenses within them. |
| **Expense Payer** | The User who paid an expense on behalf of others. |
| **Expense Participant** | A User whose balance is affected by an expense because they share the cost. |
| **Approver** | A User designated to review and approve (or reject) expenses before they affect balances. |
| **Group Owner** | The User who created a Group and manages its membership. |
| **Guest / Public** | Anyone without an account; can view nothing but the login/registration screen. |

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and split an expense among participants (Priority: P1)

A user logs in and records an expense they paid for a group: they enter the amount, date, description, a shared manner (equally split, or split by percentage), an optional receipt image, and the participants involved. The system computes each participant's share and adjusts everyone's balance.

**Why this priority**: This is the core value of the product — without creating and splitting expenses, no balance or payment exists.

**Independent Test**: Can be fully tested by creating one equal-split expense for 3 participants and checking each resulting stakeholder balance; delivers the primary "who owes whom" value.

**Acceptance Scenarios**:

1. **Given** I am logged in and a member of a group, **When** I create an expense of 90.60 shared equally among 3 participants with a description and date, **Then** the system shows each participant's share and updates the balances of payer and participants accordingly.
2. **Given** I am creating an expense, **When** I choose split-by-percentage (50/30/20) and enter valid percentages, **Then** each participant's balance is adjusted by their exact percentage share.
3. **Given** I am creating an expense, **When** I attach a receipt image, **Then** participants can view the receipt image with the expense.
4. **Given** I am creating an expense, **When** I enter an amount that is not a valid positive monetary value, **Then** the system prevents submission and tells me what is wrong.

---

### User Story 2 - Track balances between every pair of users (Priority: P1)

After expenses (and payments) exist, every user can see, for every other relevant user, the exact amount owed to or by them, considering all entered expenses.

**Why this priority**: Balances are the direct output of expense sharing; pairwise debts are what users act on.

**Independent Test**: Can be tested with two users and one expense plus one payment to verify the pairwise balance equals paid minus share minus payments.

**Acceptance Scenarios**:

1. **Given** two groups share three expenses and one payment, **When** either user opens the balance view, **Then** the owed/owing amount per counterpart user reflects all of it.
2. **Given** an expense is rejected in the approval workflow, **When** I view balances, **Then** the rejected expense never changed any balance.

---

### User Story 3 - Log in with a personal account (Priority: P1)

Each person registers their own user account and logs in to access only their own expense and balance information.

**Why this priority**: Personal accounts gate all other features and personalize balances/notifications.

**Independent Test**: Can be tested by registering an account, logging in, verifying another user's private data is not visible, and logging out.

**Acceptance Scenarios**:

1. **Given** a new user, **When** they register with required identity fields and then log in, **Then** they land on their personal view of expenses, groups, and balances.
2. **Given** an existing account, **When** a user logs in with correct credentials, **Then** they see their own expense, balance, and payment information.
3. **Given** an existing account, **When** a user enters a wrong credential, **Then** login fails with a clear message and they stay logged out.

---

### User Story 4 - Create and manage groups (Priority: P2)

A user creates a group (e.g., "Roommates", "Trip to Chicago"), adds/removes members, and shares within the group become organized so shared expenses and balances can be tracked per group.

**Acceptance Scenarios**:

1. **Given** I am logged in, **When** I create a group and add two users, **Then** the group appears in my groups with those members.
2. **Given** a group I own, **When** I add a member to future expenses, **Then** shared expenses and balances within that group reflect the membership.

### User Story 5 - Record payments to settle balances (Priority: P2)

A user records that a payment was made toward an owed balance so the balances accurately reflect the settlement state.

**Acceptance Scenarios**:

1. **Given** A owes B 30.00, **When** A records a payment of 30.00 to B, **Then** the balance between A and B is zero.
2. **Given** A owes B 30.00, **When** A records a partial payment of 10.00, **Then** the balance between A and B shows 20.00 owed.

### User Story 6 - Approval workflow for expenses (Priority: P3)

Before an expense is used to adjust balances, designated reviewers must approve it. A group may apply a single review step or multiple approval levels, depending on the group's needs.

**Acceptance Scenarios**:

1. **Given** a group with single approval enabled, **When** a member creates an expense, **Then** it stays pending until an approver approves it, and only then do balances adjust.
2. **Given** a group with two approval levels, **When** the first approver approves, **Then** the expense waits at the second level and balances still do not change until fully approved.
3. **Given** a pending expense, **When** an approver rejects it, **Then** no balance is affected and the creator is informed.

### User Story 7 - Get summaries and custom reports (Priority: P3)

Users receive reports and summaries of their expenses, balances, and payments, and can generate custom reports over a specified date range and other parameters.

**Acceptance Scenarios**:

1. **Given** months of activity, **When** I request a summary report of my expenses, balances, and payments, **Then** the report matches the underlying records.
2. **Given** a date range 2026-01-01 to 2026-01-31 and optional filters, **When** I generate a custom report, **Then** only in-range activity with the requested parameters appears.

### User Story 8 - Notify group members of new expenses (Priority: P4)

When a group member creates a new expense, other members of the group are notified.

**Acceptance Scenarios**:

1. **Given** I am in a group with two other members, **When** one of them creates a new expense, **Then** I receive a notification describing the expense (amount, description, creator).
2. **Given** I created an expense myself, **When** it is submitted, **Then** I am not notified of my own expense.

### User Story 9 - Platform reach: web-first with mobile companions (Priority: P4)

The customer expects the product primarily web-based, and also as apps runnable on Android and iOS. The MVP is a desktop application subset; the web and mobile deliveries are deferred releases of the same behaviors.

**Acceptance Scenarios**:

1. **Given** the deferred platform release, **When** a user accesses SplitSmart from the web or a phone, **Then** the same accounts, groups, expenses, balances, payments, reports, notification, and approval behaviors of this specification apply.

---

### Edge Cases

- What happens when an expense amount cannot be divided exactly into cents (e.g., 100.00 equal-split among 3)? → Shares are rounded half-up (33.34, 33.33, 33.33); leftover cents go to the first participants in list order (FR-006).
- What happens when an amount is zero, negative, blank, or non-numeric? → Rejected at entry with a specific message; no expense created (FR-009).
- What happens when percentages do not sum to exactly 100%? → Expense rejected at submission with a clear message (FR-007).
- What happens when a participant is removed from a group while they still owe or are owed? → Their past balances remain in force; they are excluded from future participant selection in that group (FR-003).
- What happens when the payer is also a participant? → The payer's share is computed like any participant's; the payer never appears to owe themselves (clarification 1).
- What happens when duplicate expenses with identical values are submitted? → Treated as separate expenses; balances count each one by design, and each can be approved or rejected independently.
- What happens when notifications are generated while a user is offline? → Queued in the in-app inbox and shown at next login (FR-010).
- What happens when a payment larger than the owed balance is submitted? → Rejected; payments cannot exceed the current pairwise debt (FR-013).

## Requirements *(mandatory)*

### Functional Requirements

**Accounts & Access (customer feature F1)**

- **FR-001**: The system MUST let each person register their own user account and log in to access their own expense and balance information.
  - *Acceptance*: A user can register, log in, log out; two different users cannot see each other's private data without being in the same context (group/expense).
- **FR-002**: The system MUST authenticate each user on login with an email + password credential and reject invalid credentials with a clear message; the session persists until logout, and a password reset flow is available.
- **FR-002a**: The system MUST keep each logged-in user's private views (own expenses, balances, payments) inaccessible to other users outside shared contexts.

**Groups (customer feature F2)**

- **FR-003**: The system MUST allow users to create groups (e.g., roommates, trip or party participants), and manage them (rename, add or remove members, end a group).
  - *Acceptance*: A created group lists its members; membership changes are reflected in future expense participant selection. An expense MAY involve any registered user even outside the group; a removed member keeps their past balances and is excluded from future participant selection for that group.
- **FR-004**: A group owner MUST be able to choose the approval structure for the group: a single review step or multiple approval levels.

**Expenses (customer feature F3)**

- **FR-005**: The system MUST allow users to create new expenses containing: amount, date, description, shared manner (equally split or split by %), and an optional receipt image.
  - *Acceptance*: Every created expense carries all five attributes; receipt image is optional (absence does not block creation).
- **FR-006**: The system MUST divide an equal-split expense among the participant users exactly: each share is rounded half-up to cents and leftover cents are assigned to the first participants in participant-list order until exhausted.
  - *Acceptance*: For 100.00 among 3 participants the shares (33.34, 33.33, 33.33 in list order) sum exactly to 100.00.
- **FR-007**: The system MUST accept a percentage split only when the entered percentages sum to exactly 100% at submission, and MUST compute each participant's share by exact percentage arithmetic.
  - *Acceptance*: Over/under-100% submissions are rejected with a clear message; accepted splits sum exactly to the expense amount.
- **FR-008**: The system MUST let the expense creator specify which other users were involved in the expense so that the appropriate balance adjustments are made.
  - *Acceptance*: Only named participants' balances change; the payer and participant sets are independent (Clarification 1), and no user ever owes themselves.
- **FR-009**: The system MUST reject invalid expense input — non-numeric, blank, zero, or negative amounts; amounts with more than two decimal places; percentages not summing to exactly 100%; incomplete required fields — with a specific validation message.
  - *Acceptance*: Each invalid input type produces a distinct, actionable message and no balance change.

**Notifications (customer feature F4)**

- **FR-010**: The system MUST notify group members (through the in-app notification inbox) when another group member creates a new expense (amount, description, creator); external channels (email/push) are deferred.
  - *Acceptance*: Every other member of the group receives the event; the creator does not self-notify; notifications are queued while a user is offline and shown at next login.

**Approval (customer feature F5)**

- **FR-011**: The system MUST review and approve expenses before they are added to the system and used to adjust balances; a group MAY configure one or multiple approval levels.
  - *Acceptance*: Pending expenses produce no balance adjustment; approval at the final level immediately adjusts balances; rejection never adjusts balances.

**Balances (customer feature F6)**

- **FR-012**: The system MUST track the balance owed by each user to every other user, considering all entered (approved) expenses.
  - *Acceptance*: Balance per user pair equals the documented computed result of the scenario table; adding an approved expense updates both directions correctly.

**Payments (customer feature F7)**

- **FR-013**: The system MUST allow users to record payments that settle owed balances so the balances accurately reflect settlement state.
  - *Acceptance*: Full payment zeroes the pairwise balance; partial payment reduces it by the exact paid amount; a payment cannot exceed the currently owed balance between the two users.

**Reporting (customer feature F8)**

- **FR-014**: The system MUST provide users with reports and summaries of their expenses, balances, and payments.
- **FR-015**: The system MUST let users generate custom reports with specified date ranges and other parameters (e.g., group, counterpart user, expense vs. payment type).
  - *Acceptance*: A custom report over a range contains exactly the matching activity, totals correct to the smallest monetary unit.

**Platform (customer feature F9)**

- **FR-016**: The product MUST primarily be delivered as a web-based application and MUST also ship as applications runnable on Android and iOS. [Deferred release — see MVP Scope]

### Non-Functional Requirements

- **NFR-001**: Monetary accuracy — all expense, balance, payment, and report amounts MUST be exact to the smallest monetary unit used by the product (cents), with no floating-point drift, on any operation sequence.
- **NFR-002**: Privacy — a logged-in user MUST NOT see any personal data of users they do not share a group/expense/payment context with.
- **NFR-003**: Performance — a newly approved expense MUST be reflected in the affected users' balance views within one (1) second.
- **NFR-004**: Usability — a first-time user MUST be able to create an account and record their first expense in under 3 minutes without external help.
- **NFR-005**: Auditability — creation, submission, approval, rejection, and payment of an expense MUST be traceable in the system with actor, timestamp, and outcome.
- **NFR-006**: Availability of record — receipt images, once attached, MUST remain viewable by the expense participants for the life of the expense.

### Key Entities *(include if feature involves data)*

- **User**: an individual with an account; credentials, personal identifiers, own expense/balance/payment views.
- **Group**: a named set of Users created and managed by a Group Owner (roommates, trip or party participants).
- **Expense**: amount, date, description, shared manner (equal or percentage), optional receipt image, payer, set of participating Users, approval state.
- **Share**: one User's portion of one Expense (amount or percentage), the basis of balance adjustment.
- **Approval**: review step(s) with level, reviewer, decision, timestamps.
- **Payment**: amount, date, direction (who paid whom), applied against owed balances.
- **Notification**: event about a new expense targeted at other group members.
- **Report**: parameterized summary (date range, filters) of expenses, balances, and payments.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A new user can register and record their first expense in under 3 minutes (NFR-004).
- **SC-002**: 100% of pairwise balances displayed are consistent with the underlying expenses and payments when audited on a sample of 10 random scenarios.
- **SC-003**: Equal-split scenarios (including non-divisible amounts like 100.00 across 3 users) reconcile to the exact expense amount in 100% of test cases.
- **SC-004**: 0% of rejected or unapproved expenses appear in any balance or report.
- **SC-005**: A user can produce a date-ranged report in under 10 seconds covering any range of stored activity.
- **SC-006**: 90% of testers successfully complete the primary flow (login → create group → create expense → view balance) on first attempt without help.
- **SC-007**: Every other group member receives the in-app expense notification in 100% of expense-creation cases.

## Assumptions

- The customer's platform request (F9) is a long-term roadmap; the course MVP implements a single-platform subset (see MVP Scope below). This is an implementation decision documented here and in the constitution, not a change to the customer request.
- Receipt images are attached as files chosen through the application's file picker; no automatic (e.g., OCR) interpretation is required.
- One user can belong to many groups; expenses always belong to exactly one group if a group context is used, or to a pair-of-users context.
- Currency: a single currency setting per environment (e.g., USD); multi-currency conversion is out of scope.
- Expense dates may be entered as today or a past date, but not a future date; each expense has exactly one payer.
- Data is stored persistently by the product; no retention-limit policy is requested by the customer.
- Authentication (email + password), exactly-two-decimal amounts, exact-100% percentage rule, in-app notifications, and capped pairwise payments were confirmed with the customer (see Clarifications, 2026-10-29) and are no longer assumptions.

## MVP Scope (Java Swing Subset) — implementation decision

Per the project constitution (Java 21, Maven, Java Swing, JUnit 5, PlantUML), the first implementation release covers:

- **In MVP**: FR-001 through FR-015 (accounts, groups, expenses with equal and percentage splits, in-app notifications, single- and multi-level approval, pairwise balances, payment recording, reports with date ranges) as a Java Swing desktop application; monetary arithmetic exact to the cent per NFR-001.
- **Deferred to later releases**: FR-016 web-based delivery, Android and iOS applications (F9); external notification channels (email/push) for the notification system; multi-currency; receipt interpretation.

This subset preserves all original customer features in the requirements inventory (see `requirements.md`); deferred items are labeled "Deferred", never dropped.

## Open Questions — Resolved

All eight open questions from the draft were put to the customer representative and answered on 2026-10-29 (see **Clarifications** above and the full decision log in [`clarifications.md`](./clarifications.md)). No open questions remain. Questions raised during later phases will be appended here with their resolution date.