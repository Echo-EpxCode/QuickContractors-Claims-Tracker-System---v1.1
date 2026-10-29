# Claims Tracker System

## Implementation Plan — Version 1.1

**Document status:** Approved for implementation planning  
**Version:** 1.1  
**Supersedes:** Implementation Plan Version 1.0  
**Date:** October 29, 2026  
**System:** QuickContractors Claims Tracker System  
**Hosting:** InfinityFree  
**Backend:** Procedural PHP with MySQLi  
**Database:** MySQL-compatible database using InnoDB  
**Current implementation status:** Not started. No database or application exists yet.

---

## 1. Document Purpose

This Implementation Plan defines the approved sequence for building Claims Tracker System V1.0.

Implementation must remain aligned with the approved:

1. Product Requirements Document (PRD) V1.1
2. Technical Requirements Document (TRD) V1.1
3. App Flow
4. Design Brief Document V1.1
5. Background Schema / Database Schema Design Document V1.1
6. Document Reconciliation Decision Record V1.1

The V1.1 Decision Record is the authoritative reconciliation reference. If any document conflicts with it, pause the affected work and resolve the conflict before coding.

---

## 2. Approved V1.0 Scope

### 2.1 Included Scope

- User authentication, logout, and role-based authorization.
- Claim creation, viewing, editing, searching, filtering, and pagination.
- Claim assignment, reassignment, and take-over.
- Claim status and priority management.
- Store management.
- Service-call management.
- Claim-party management.
- Follow-up scheduling and follow-up lifecycle management.
- One current scheduled follow-up per claim.
- Overdue, Due Today, and Upcoming follow-up queues.
- Claim action history.
- Claim activity logs.
- Claim lifecycle history.
- Assignment history.
- Claim guidance and guidance revision history.
- Role-specific dashboards and management oversight.
- Server-side validation, authorization, error handling, transactions, and database integrity.
- Backup and restoration procedures appropriate to InfinityFree.
- Production deployment and acceptance testing.

### 2.2 Excluded Scope

Do not implement, display, or create placeholder functionality for:

- Claim attachments.
- File uploads.
- Attachment management.
- Excel import.
- Bulk import.
- Email reminders.
- Automated notifications.
- A notification subsystem.
- External integrations.
- Multi-tenant SaaS functionality.
- Laravel, PDO, or any other backend framework.
- Separate `roles` and `user_roles` tables.
- The former `actions` table.

---

## 3. Approved Technical Environment

| Component              | Approved technology or decision                 |
| ---------------------- | ----------------------------------------------- |
| Backend                | Procedural PHP                                  |
| Database access        | MySQLi                                          |
| Database engine        | MySQL-compatible database using InnoDB          |
| Frontend               | HTML, CSS, Vanilla JavaScript, Bootstrap 5.3    |
| Icons                  | Font Awesome 6                                  |
| User-facing alerts     | SweetAlert2                                     |
| Local PHP environment  | Herd                                            |
| Local database service | DBngin                                          |
| Database inspection    | TablePlus                                       |
| Hosting                | InfinityFree                                    |
| File transfer          | FileZilla, where required by the hosting method |

Use prepared statements for all queries involving supplied values. Keep credentials out of publicly accessible source files and repositories.

---

## 4. Approved Business Rules

### 4.1 Claim Rules

Every new claim must have:

- A unique `job_id`.
- One required store.
- A claim type from `Leak`, `Damage`, or `Outdoor Damage`.
- One required claim party.
- An assigned handler.
- One current scheduled follow-up with a due date.
- A priority.

Default values:

- Claim status: `Open`
- Claim priority: `Normal`

### 4.2 Claim Status Rules

Approved statuses:

1. `Open`
2. `In Progress`
3. `Waiting`
4. `Resolved`
5. `Closed`

Approved transitions:

| From status   | Allowed transitions                            |
| ------------- | ---------------------------------------------- |
| `Open`        | `In Progress`, `Waiting`, `Resolved`, `Closed` |
| `In Progress` | `Waiting`, `Resolved`, `Closed`                |
| `Waiting`     | `In Progress`, `Resolved`, `Closed`            |
| `Resolved`    | `Closed`, `Open`                               |
| `Closed`      | `Open`                                         |

### 4.3 Follow-Up Rules

- Use `follow_ups`, not `actions`.
- A claim may have only one current scheduled follow-up.
- Historical follow-ups must be preserved.
- The current follow-up follows the claim’s current handler.
- Historical follow-ups retain their original handler.
- Follow-up due dates are date-only.
- Approved follow-up statuses are `Scheduled`, `Overdue`, `Completed`, `Cancelled`, and `Superseded`.
- A new follow-up supersedes the existing current scheduled follow-up.
- Closing a claim cancels its current scheduled follow-up.

### 4.4 Date Rules

- Store dates and timestamps in UTC.
- Display dates and calculate the current date using `Asia/Manila`.
- Display dates as `MM/DD/YYYY`.
- Display timestamps as `MM/DD/YYYY hh:mm AM/PM` in `Asia/Manila` time.
- A `Scheduled` follow-up is `Overdue` when due before the current Philippine date.
- A follow-up due today is `Due Today`, not `Overdue`.
- A future-dated follow-up within the next seven days is `Upcoming`.
- Do not use `Due soon`; use `Upcoming`.

### 4.5 Role Rules

| Stored role value | Display label  |
| ----------------- | -------------- |
| `Claim_Adjuster`  | Claim Adjuster |
| `Supervisor`      | Supervisor     |
| `Director`        | Director       |

- Each user has exactly one role.
- The role is stored directly on the user record.
- Claim Adjusters see only claims assigned to them.
- Supervisors and Directors see all claims.
- Only Supervisor and Director can reassign or take over claims.
- Only Supervisor and Director can manage users.
- Only Director can create or manage Supervisor and Director accounts.
- Supervisor can manage Claim Adjuster accounts.
- Users must be deactivated, not deleted.
- A user cannot delete their own account.

### 4.6 Take-Over Rules

- Supervisor and Director may take over an unassigned claim or a claim assigned to another user.
- Take-over is allowed for claims in any status, including `Resolved` and `Closed`.
- Taking over a `Closed` claim automatically reopens it.
- A closed claim that is taken over must require a new follow-up.
- Before saving take-over, show a SweetAlert2 confirmation identifying the current handler, if any, and stating that the handler will be replaced.
- Every reassignment and take-over must be recorded in assignment history.

### 4.7 Guidance Rules

- Claim Adjuster can view guidance only.
- Supervisor can create and edit guidance.
- Director can create, edit, approve, and deactivate guidance.
- Every guidance edit and every guidance status change (approval or deactivation) must create a new revision.
- Previous revisions remain viewable and must not be deleted through the application.
- Guidance statuses are `Draft`, `Approved`, and `Inactive`.

---

## 5. Implementation Phases

### Phase 1 — Requirements Reconciliation and Baseline

**Objective:** Establish a verified implementation baseline before writing PHP or SQL.

#### Activities

1. Review PRD V1.1, TRD V1.1, Design Brief V1.1, Background Schema V1.1, App Flow, and Decision Record V1.1.
2. Confirm that the approved database model is the 13-table model (the original eleven tables plus `claim_guidance` and `claim_guidance_revisions`).
3. Confirm that `follow_ups` replaces `actions`.
4. Confirm that `claim_attachments` and attachment functionality are excluded.
5. Extract all approved claim types, statuses, priorities, party types, follow-up statuses, guidance statuses, and role values.
6. Extract the approved status-transition matrix.
7. Extract role permissions, claim visibility, user-management rules, guidance permissions, and menu visibility.
8. Extract dashboard metrics, claims-list columns, filters, pagination, and confirmation-dialog requirements.
9. Confirm the column definitions for `claim_guidance` and `claim_guidance_revisions`.
10. Create a requirement-to-feature traceability checklist.
11. Create initial test cases for successful and rejected operations.
12. Record unresolved questions and obtain decisions before implementation.

#### Completion Criteria

- All approved documents are reviewed.
- V1.1 decisions are incorporated into the implementation checklist.
- No conflicting requirement remains unresolved.
- The requirements and test checklists are ready.

**Deliverables:** Requirements traceability matrix, implementation checklist, initial test plan, and open-decision log.

---

### Phase 2 — Local Environment and Database Setup

**Objective:** Create and verify the approved database structure.

#### Activities

1. Confirm the project root and preserve the approved directory structure.
2. Start Herd for local PHP.
3. Start or configure MySQL through DBngin.
4. Create or select the approved local development database.
5. Connect through TablePlus and verify connectivity.
6. Create the approved 13 tables in dependency order.
7. Define primary keys, foreign keys, required columns, data types, defaults, indexes, and unique constraints.
8. Enforce unique `claims.job_id`.
9. Enforce one required store per claim.
10. Enforce one required claim party per claim.
11. Enforce zero-or-one service call per claim.
12. Enforce unique populated `sc_job_id`.
13. Store one role directly on each user record.
14. Do not create `roles`, `user_roles`, `actions`, or `claim_attachments`.
15. Seed approved claim types, claim statuses, and required reference data.
16. Verify foreign-key behavior, invalid references, and deactivation-over-deletion rules.
17. Verify the transaction-safe one-current-follow-up design.
18. Configure database credentials outside public source files where the environment permits.
19. Verify safe handling of database-connection failures.

#### Completion Criteria

- The local environment connects to MySQL.
- The approved schema and constraints are present.
- No excluded table or feature exists.
- Database setup can be recreated from documented SQL and instructions.

**Deliverables:** Verified schema, SQL installation/migration scripts, seed data, environment instructions, and database verification checklist.

---

### Phase 3 — Authentication, Authorization, and Application Foundation

**Objective:** Build the security and shared application foundation.

#### Activities

1. Implement the login page using the approved Design Brief.
2. Verify credentials on the server.
3. Store passwords using PHP password-hashing functions.
4. Implement logout and session invalidation.
5. Regenerate the session ID after successful login.
6. Reject inactive accounts.
7. Load role and identity from trusted server-side data.
8. Create reusable authorization helpers for pages and operations.
9. Enforce role and record-level authorization on every protected request.
10. Prevent authorization bypass through direct URLs or modified request parameters.
11. Add CSRF protection to state-changing forms.
12. Create shared MySQLi connection and prepared-statement patterns.
13. Create validation, output-escaping, date-formatting, and safe-error-handling helpers.
14. Test valid login, invalid login, logout, inactive accounts, expired sessions, and unauthorized access for all roles.

#### Completion Criteria

- Authentication and logout work securely.
- Server-side authorization blocks unauthorized access.
- Direct URL manipulation does not bypass permissions.
- Database and application errors do not expose sensitive details.

**Deliverables:** Authentication module, authorization helpers, shared database-access patterns, security helpers, and test results.

---

### Phase 4 — Core Claim Management

**Objective:** Implement claim records, lifecycle, assignment, and search.

#### Activities

1. Implement the claims list using the approved 11 columns.
2. Implement claim details for authorized users.
3. Implement claim creation with required store, claim type, claim party, handler, follow-up due date, and priority.
4. Enforce unique `job_id`.
5. Set new-claim status to `Open` and priority to `Normal`.
6. Implement claim editing according to role and claim-visibility rules.
7. Implement approved claim statuses and transition rules.
8. Implement the six approved priority values.
9. Implement assignment and reassignment according to role permissions.
10. Implement Supervisor and Director take-over, including Closed-claim reopening and new-follow-up requirement.
11. Show the required SweetAlert2 confirmation before take-over.
12. Record assignment history for every reassignment and take-over.
13. Implement claim-party creation and editing.
14. Record claim-party changes in claim history or activity logs.
15. Implement Job ID search and all approved filters.
16. Implement 25-records-per-page pagination.
17. Use prepared statements and strict allowlists for dynamic query values.
18. Test authorized and unauthorized operations for each role.

#### Completion Criteria

- Authorized users can create, view, edit, search, filter, assign, reassign, and take over claims.
- Required fields, unique identifiers, statuses, priorities, and transitions are enforced.
- Assignment and take-over operations are traceable and transaction-safe.
- Claims lists and filters return only authorized results.

**Deliverables:** Claim list, claim detail, create/edit workflow, assignment/reassignment/take-over workflow, claim-party workflow, search/filter/pagination, and test results.

---

### Phase 5 — Follow-Ups, History, and Date Queues

**Objective:** Implement the daily follow-up workflow and history model.

#### Activities

1. Implement follow-up scheduling with required date-only due date.
2. Enforce one current scheduled follow-up per claim.
3. Mark an existing current scheduled follow-up as `Superseded` when a new follow-up is scheduled.
4. Preserve the prior follow-up and its original handler.
5. Record follow-up superseding in `action_history`.
6. Implement follow-up completion, cancellation, and updating.
7. Record follow-up scheduled, completed, cancelled, and superseded events in `action_history`.
8. Implement the Overdue queue using current Philippine date logic.
9. Implement the Due Today queue.
10. Implement the Upcoming queue for future due dates within seven days.
11. Ensure completed, cancelled, and superseded follow-ups do not appear in pending queues.
12. Implement claim closure with cancellation of the current scheduled follow-up.
13. Implement claim reopening with handler and dated-follow-up requirements.
14. Record claim closure and reopening in `action_history` and `claim_history`.
15. Implement claim activity logging for status, priority, assignment, field, and claim-party changes.
16. Use transactions for related claim, follow-up, history, and audit writes.
17. Test date boundaries, duplicate submissions, concurrent follow-up creation, and invalid operations.

#### Completion Criteria

- One-current-follow-up rule is enforced.
- Historical follow-ups remain preserved.
- Queue classification is correct.
- History and audit records are complete and protected.
- Related updates commit or roll back together.

**Deliverables:** Follow-up workflow, action history, activity logs, claim history, queue pages, and test results.

---

### Phase 6 — Stores, Service Calls, Users, and Guidance

**Objective:** Implement supporting modules and their role-based permissions.

#### Activities

1. Implement Stores with Claim Adjuster view-only access.
2. Implement Supervisor and Director store management.
3. Require store deactivation confirmation and preserve claim references.
4. Implement Service Calls for Supervisor and Director only.
5. Enforce zero-or-one service call per claim.
6. Enforce unique populated `sc_job_id`.
7. Implement Users for Supervisor and Director.
8. Restrict Supervisor user management to Claim Adjuster accounts.
9. Restrict Supervisor and Director account management to Director.
10. Implement user deactivation rather than deletion.
11. Prevent a user from deleting their own account.
12. Implement Claim Guidance.
13. Enforce Claim Adjuster view-only guidance access.
14. Enable Supervisor guidance creation and editing.
15. Enable Director guidance creation, editing, approval, and deactivation.
16. Create a new revision for every guidance edit and every guidance status change (approval or deactivation).
17. Preserve prior guidance revisions.
18. Implement required SweetAlert2 confirmations for deactivation operations.

#### Completion Criteria

- Each module enforces its approved role permissions.
- Store, service-call, user, and guidance rules work correctly.
- Guidance revisions are preserved.
- No excluded feature is exposed.

**Deliverables:** Store module, service-call module, user-management module, guidance module, and test results.

---

### Phase 7 — Dashboards and Management Oversight

**Objective:** Implement role-specific dashboards and management views.

#### Activities

1. Implement the approved sidebar and role-based menu visibility.
2. Apply approved visual tokens: sidebar `#343A40`, content background `#F4F6F9`, primary accent `#0D6EFD`, and Roboto.
3. Implement dashboard metrics:
    - Open Claims
    - Overdue Follow-ups
    - Due Today
    - Upcoming Follow-ups
    - Claims Requiring Attention
4. Implement Claims Requiring Attention for claims with overdue follow-ups or `Urgent`/`Escalated` priority.
5. Calculate all counts from database records and approved rules.
6. Apply role and record-visibility restrictions to every dashboard query.
7. Implement Claim Adjuster dashboard views for assigned claims.
8. Implement Supervisor and Director management views for all claims.
9. Implement clear loading, empty, validation-error, and success states.
10. Integrate Bootstrap 5.3, Font Awesome 6, and SweetAlert2 as approved.
11. Test desktop, tablet, and mobile layouts.
12. Confirm excluded features do not appear as available actions.

#### Completion Criteria

- Each role sees the correct navigation and authorized information.
- Dashboard counts match the underlying authorized data.
- The interface follows the Design Brief.
- Responsive layouts preserve core workflows.

**Deliverables:** Role-specific dashboards, navigation, management pages, shared UI components, and visual-review results.

---

### Phase 8 — Integration, Security, and Acceptance Testing

**Objective:** Verify the complete integrated application before deployment.

#### Activities

1. Create a requirements-to-test checklist covering every approved V1.0 feature.
2. Test login through claim creation, assignment, follow-up, status updates, closure, reopening, and final claim handling.
3. Test each role independently.
4. Attempt unauthorized access through direct URLs and modified requests.
5. Test required fields, invalid types, excessive lengths, invalid enumerations, and malformed input.
6. Test duplicate `job_id` values and invalid foreign-key references.
7. Test all approved and prohibited status transitions.
8. Test follow-up date calculations for Overdue, Due Today, and Upcoming.
9. Test one-current-follow-up enforcement under duplicate and concurrent requests.
10. Test action history, activity logs, claim history, assignment history, and guidance revisions.
11. Verify search, filters, pagination, and dashboard totals.
12. Review SQL queries for prepared-statement use and safe dynamic identifiers.
13. Test database failures, invalid sessions, and failed multi-step operations.
14. Verify transaction rollback behavior.
15. Review output escaping and CSRF protection.
16. Test backup and restoration procedures.
17. Test responsive layouts.
18. Record defects with severity, reproduction steps, expected behavior, actual behavior, and resolution.
19. Fix release-blocking defects and rerun affected tests.
20. Run regression tests.
21. Produce a test summary and obtain release approval.

#### Completion Criteria

- Required functional, security, validation, integrity, regression, and recovery tests pass.
- Release-blocking defects are resolved.
- Backup and restoration procedures are tested or hosting limitations are documented and accepted.
- The release candidate is approved for deployment.

**Deliverables:** Test cases, defect log, regression results, security checklist, recovery-test results, and release-candidate approval.

---

### Phase 9 — Production Deployment and Release

**Objective:** Deploy the verified application to InfinityFree.

#### Activities

1. Confirm Phase 8 completion and deployment approval.
2. Review current InfinityFree PHP, MySQL, resource, HTTPS, backup, and deployment capabilities.
3. Prepare the production database using the approved schema.
4. Create production credentials without committing secrets to source control.
5. Separate production configuration from local development configuration.
6. Back up existing production data or files before replacing an existing deployment, if applicable.
7. Upload approved files using FileZilla or the approved deployment method.
8. Exclude local-only files, development credentials, temporary files, and test data.
9. Apply the approved schema and initialize authorized accounts securely.
10. Verify database connectivity.
11. Verify login, logout, role permissions, claim creation, editing, assignment, follow-up scheduling, history, search, and dashboards.
12. Verify safe production error handling.
13. Run production smoke tests.
14. Confirm the available backup process and document hosting limitations.
15. Record deployed version, deployment date, configuration steps, and known limitations.
16. Obtain final acceptance.

#### Completion Criteria

- The application is accessible at the approved production location.
- Database connectivity and core workflows pass production smoke tests.
- Role restrictions remain enforced.
- No development credentials or test-only data are exposed.
- Recovery and operational procedures are documented.
- Known limitations are recorded and accepted.

**Deliverables:** Deployed application, production smoke-test results, deployment record, and operational notes.

---

## 6. Milestones

| Milestone                            | Completion criteria                                                                                               | Required evidence                                   |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| M1 — Requirements Baseline Approved  | V1.1-aligned requirements, permissions, workflows, exclusions, and schema decisions are traceable.                | Requirements checklist and implementation task list |
| M2 — Development Environment Ready   | Local PHP and MySQL run; application connects to the development database.                                        | Environment verification checklist                  |
| M3 — Database Verified               | Approved 13-table model, relationships, constraints, indexes, seed values, and unique `job_id` rule are verified. | Schema verification results                         |
| M4 — Security Foundation Complete    | Login, logout, sessions, and server-side role authorization pass tests.                                           | Authentication and authorization test results       |
| M5 — Claim Management Complete       | Claim CRUD, statuses, priorities, assignment, take-over, party, search, filters, and pagination work.             | Claim-module test results                           |
| M6 — Follow-Up Workflow Complete     | Follow-ups, history, queues, closure, reopening, and transaction safety work correctly.                           | Follow-up and history test results                  |
| M7 — Supporting Modules Complete     | Stores, service calls, users, and guidance follow approved permissions and rules.                                 | Supporting-module test results                      |
| M8 — Dashboards Complete             | Role-specific dashboards, metrics, menus, and management views follow approved rules.                             | Role-based UI review and dashboard checks           |
| M9 — Testing Approved                | Functional, security, validation, integrity, regression, and recovery checks are complete.                        | Test summary, defect log, release approval          |
| M10 — Production Deployment Complete | InfinityFree deployment is available and production smoke tests pass.                                             | Deployment record and smoke-test results            |
| M11 — Final Acceptance               | Required workflows pass; release-blocking defects are resolved; operational instructions are complete.            | Final acceptance checklist                          |

### 6.1 Dependency Sequence

1. M1 — Requirements Baseline Approved
2. M2 — Development Environment Ready
3. M3 — Database Verified
4. M4 — Security Foundation Complete
5. M5 — Claim Management Complete
6. M6 — Follow-Up Workflow Complete
7. M7 — Supporting Modules Complete
8. M8 — Dashboards Complete
9. M9 — Testing Approved
10. M10 — Production Deployment Complete
11. M11 — Final Acceptance

### 6.2 Milestone Review Rules

At each milestone:

1. Compare completed work with its acceptance criteria.
2. Run required tests.
3. Record defects, incomplete work, and decisions.
4. Resolve release-blocking issues before proceeding.
5. Confirm no unapproved feature, schema, or architecture change was introduced.
6. Record approval to close the milestone.

A milestone is not complete merely because code has been written; required behavior must be verified.

---

## 7. Cross-Phase Standards

### 7.1 Requirements and Change Control

- Treat the approved V1.1 documents as the source of truth.
- Do not silently resolve conflicting requirements by guessing.
- Explain the impact of a proposed change and obtain approval before changing scope, architecture, file structure, or schema.
- Keep V1.0 exclusions out of implementation.

### 7.2 PHP and Database Safety

- Use procedural PHP and MySQLi.
- Use prepared statements for supplied values.
- Validate all untrusted input on the server.
- Escape output appropriately.
- Use transactions for related writes.
- Handle database failures without exposing internal diagnostics.
- Enforce uniqueness, relationships, and allowed values at the database level where appropriate.

### 7.3 Authentication and Authorization

- Hash passwords using supported PHP password-hashing functions.
- Protect sessions and regenerate session IDs after authentication.
- Check permissions on every protected server-side operation.
- Apply role and record-visibility restrictions to database queries.
- Use CSRF protection for relevant state-changing requests.

### 7.4 History and Data Integrity

- Follow the approved history-table responsibilities.
- Do not overwrite or delete historical information through normal application actions.
- Associate history with the correct claim and acting user.
- Use transactions where partial completion would create inconsistent data.
- Do not add unapproved history tables or fields.

### 7.5 Testing and Release

- Write test cases from approved requirements and acceptance criteria.
- Test successful and rejected behavior.
- Test each role separately.
- Rerun affected tests after defect fixes.
- Keep production deployment separate from unverified development work.
- Maintain a record of deployed version, known issues, and recovery procedures.

---

## 8. Final Release Checklist

### Requirements and Scope

- [ ] All approved V1.1 source documents have been followed.
- [ ] Every approved V1.0 feature has been implemented or explicitly documented as blocked.
- [ ] Claim attachments, file uploads, Excel import, email reminders, notifications, and external integrations have not been added.
- [ ] No unapproved architecture or schema changes have been introduced.

### Database and Security

- [ ] Approved 13-table structure and relationships are verified.
- [ ] `claims.job_id` uniqueness is enforced.
- [ ] One required store per claim is enforced.
- [ ] One required claim party per claim is enforced.
- [ ] Zero-or-one service call per claim is enforced.
- [ ] Populated `sc_job_id` values are unique.
- [ ] Each user has exactly one role.
- [ ] Database queries use prepared statements where supplied values are involved.
- [ ] Server-side authorization is enforced for every protected operation.
- [ ] CSRF, output escaping, input validation, and safe error handling are verified.
- [ ] Transactions protect related updates.

### Claims and Follow-Ups

- [ ] Claim creation requires store, party, handler, follow-up, type, status, and priority.
- [ ] New claims default to `Open` and `Normal`.
- [ ] Approved statuses and transitions are enforced.
- [ ] Six approved priority values are enforced.
- [ ] One current scheduled follow-up per claim is enforced.
- [ ] Follow-up superseding preserves history.
- [ ] Closing a claim cancels its current follow-up.
- [ ] Reopening enforces handler and follow-up requirements.
- [ ] Overdue, Due Today, and Upcoming queues use Philippine date logic.

### History and Audit

- [ ] `action_history` records required workflow actions.
- [ ] `claim_activity_logs` records material claim changes.
- [ ] `claim_history` records lifecycle and major status transitions.
- [ ] Assignment history records reassignment and take-over.
- [ ] Guidance edits and status changes create revisions.
- [ ] History and audit records are not editable or deletable through normal application actions.

### Interface and Release

- [ ] Sidebar menus and role visibility match V1.1.
- [ ] Dashboard displays all five required metrics.
- [ ] Claims list displays all 11 required columns.
- [ ] Claims list uses 25 records per page.
- [ ] All approved filters are implemented.
- [ ] Required SweetAlert2 confirmations are implemented.
- [ ] `Upcoming` is used; `Due soon` is not used.
- [ ] Dates display as `MM/DD/YYYY`.
- [ ] Timestamps display as `MM/DD/YYYY hh:mm AM/PM` in `Asia/Manila`.
- [ ] Backup and restore procedures are documented and tested.
- [ ] Production smoke tests pass on InfinityFree.

---

**End of Implementation Plan — Version 1.1**
