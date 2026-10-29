# Claims Tracker System  
## Technical Requirements Document (TRD) — Version 1.1

**Document status:** Approved for technical implementation  
**Version:** 1.1  
**Supersedes:** TRD Version 1.0  
**Date:** October 29, 2026  
**System release:** V1.0  
**Hosting platform:** InfinityFree  
**Backend:** Procedural PHP with MySQLi  
**Database:** MySQL-compatible relational database using InnoDB  
**Implementation status:** Not started. No database or application exists yet.

---

## 1. Purpose

This TRD defines the approved technical requirements, architecture, database model, security controls, workflow rules, and implementation constraints for Claims Tracker System V1.0.

The approved **Document Reconciliation Decision Record V1.1** is the controlling reconciliation reference. Where an earlier document conflicts with this TRD or the V1.1 Decision Record, the V1.1 decisions govern.

---

## 2. Approved System Scope

### 2.1 System Purpose

The system is a centralized, browser-based claims tracking application for QuickContractors.

It must help authorized users:

- Track claims involving third-party contractors and QuickContractors E-tech technicians.
- Identify overdue, due-today, and upcoming follow-ups.
- Maintain clear claim ownership.
- Record claim actions and preserve history.
- Support claim assignment, reassignment, and take-over.
- Provide management visibility for Supervisors and Directors.
- Maintain secure role-based access and reliable data integrity.

### 2.2 Included Scope

- Authentication and role-based authorization.
- Claims, stores, service calls, claim parties, follow-ups, guidance, users, and history.
- Claim creation, viewing, editing, searching, filtering, and pagination.
- Claim status, priority, assignment, reassignment, take-over, closure, and reopening.
- Follow-up scheduling, completion, cancellation, and superseding.
- Overdue, Due Today, and Upcoming queues.
- Dashboard metrics and Claims Requiring Attention.
- Claim action history, claim activity logs, claim lifecycle history, assignment history, and guidance revisions.
- Server-side validation, authorization, transactions, error handling, and backup/recovery planning.
- Production deployment and acceptance testing on InfinityFree.

### 2.3 Excluded Scope

The following must not be implemented in V1.0:

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
- The former `actions` table; it is replaced by `follow_ups`.

---

## 3. Technology Requirements

| Layer | Approved technology | Requirement |
|---|---|---|
| Hosting | InfinityFree | Verify PHP version, MySQL/MariaDB version, HTTPS, file deployment, database limits, backup options, and cron availability before deployment. |
| Backend | Procedural PHP 8.x | Use only a PHP version supported by the selected InfinityFree plan. |
| Database access | MySQLi | Use prepared statements for all user-controlled values. Do not use PDO. |
| Database | MySQL-compatible database using InnoDB | Use InnoDB for transactional integrity and foreign keys where supported. |
| Character encoding | `utf8mb4` | Use a consistent Unicode-safe collation. |
| Markup | HTML5 | Use semantic, accessible markup. |
| Styling | CSS3 and Bootstrap 5.3 | Apply the approved Design Brief. |
| Client behavior | Vanilla JavaScript | No frontend framework. |
| Icons | Font Awesome 6 | Provide accessible names for icon-only controls. |
| Alerts | SweetAlert2 | Use for required confirmations and user feedback. |
| Source control | Git | Maintain reviewed, version-controlled changes. |

### 3.1 Coding Requirements

- Use procedural PHP and MySQLi only.
- Use prepared statements for every query containing user-supplied values.
- Use strict server-side allowlists for dynamic SQL identifiers such as sort columns.
- Centralize reusable database connection, authentication, authorization, validation, escaping, CSRF, layout, and error-handling logic.
- Use `password_hash()` and `password_verify()` for passwords.
- Use `utf8mb4` and a consistent collation.
- Use transactions for multi-table state changes.
- Keep production error display disabled.
- Do not commit credentials, session secrets, or production configuration to source control.
- Do not rely on cron jobs or scheduled tasks until InfinityFree availability is confirmed.

---

## 4. Approved Database Model

### 4.1 Table Count

The system must use the approved **11-table model**.

### 4.2 Required Tables

| Table | Approved responsibility |
|---|---|
| `users` | User accounts, authentication data, one role per user, and account status. |
| `stores` | Store records linked to claims. |
| `claims` | Main claim record, unique `job_id`, status, priority, assigned handler, and claim references. |
| `claim_types` | Approved claim-type lookup values. |
| `claim_statuses` | Approved claim lifecycle status lookup values. |
| `claim_parties` | Required one-to-one claim-party information. |
| `service_calls` | Optional zero-or-one service-call record per claim. |
| `follow_ups` | Current and historical follow-up records. |
| `action_history` | Workflow-action history. |
| `claim_activity_logs` | Audit events for material claim changes. |
| `claim_history` | Claim lifecycle and major status-transition history. |

### 4.3 Claim Guidance Tables

The V1.1 Decision Record requires claim guidance and guidance revisions. The final 11-table model must reconcile this requirement without adding an unapproved twelfth operational table unless the Database Schema Design Document explicitly defines how guidance and revisions are represented within the approved 11-table structure.

The implementation must not silently add tables or fields. Any schema adjustment required to support guidance must be documented and approved before SQL is written.

### 4.4 Required Database Capabilities

The final schema must support:

- Users and one role per user.
- Claims with a unique `job_id`.
- One required store per claim.
- Zero or one service call per claim.
- One required claim party per claim.
- Claim status and priority.
- Current and historical follow-ups.
- Action history and assignment history.
- Claim guidance and guidance revisions.
- Claim activity and claim history.
- Secure role-based authorization.

### 4.5 Claim Identifier

- `claims.job_id` is the unique business claim identifier.
- `job_id` must have a database-level unique constraint.
- Duplicate `job_id` values must be rejected safely.

### 4.6 Claim Types

Approved values:

1. `Leak`
2. `Damage`
3. `Outdoor Damage`

Claim types must be stored and validated as controlled values, not free-form text.

### 4.7 Claim Statuses

Approved values:

1. `Open`
2. `In Progress`
3. `Waiting`
4. `Resolved`
5. `Closed`

### 4.8 Approved Status Transitions

| From status | Allowed transitions |
|---|---|
| `Open` | `In Progress`, `Waiting`, `Resolved`, `Closed` |
| `In Progress` | `Waiting`, `Resolved`, `Closed` |
| `Waiting` | `In Progress`, `Resolved`, `Closed` |
| `Resolved` | `Closed`, `Open` |
| `Closed` | `Open` |

The database may enforce valid status values, but PHP must enforce permitted transitions.

### 4.9 Claim Priority

Approved values:

1. `Non-Urgent`
2. `Low`
3. `Normal`
4. `High`
5. `Urgent`
6. `Escalated`

Priority must remain separate from claim status.

### 4.10 Stores

- Every claim must be linked to exactly one store.
- A store may have many claims.
- Stores must be deactivated rather than deleted while claims reference them.
- Foreign-key behavior must prevent deletion that would damage claim integrity.

### 4.11 Service Calls

- Each claim may have zero or one service-call record.
- `sc_job_id` is nullable.
- When populated, `sc_job_id` must be unique.
- The claim-to-service-call relationship must prevent more than one service-call record per claim.

### 4.12 Claim Parties

- Each claim must have exactly one `claim_parties` record.
- `claim_parties` is required for every claim.
- `party_type` must be either `Third_Party_Contractor` or `E_Tech`.
- Required fields:
  - `party_type`
  - `contractor_email_address`
  - `legal_company_name`
  - `company_name`
- The claim-to-party relationship must be enforced as one-to-one.
- The claim party record may be edited or changed after claim creation.
- Every claim-party change must be recorded in the appropriate claim history or activity record.

### 4.13 Follow-Ups

- Use `follow_ups` as the standard table name.
- A claim may have only one current scheduled follow-up at a time.
- Historical follow-ups must be preserved.
- The current follow-up follows the claim’s current handler.
- Historical follow-ups retain their original handler.
- The follow-up due date is a date-only value; users do not enter a follow-up time.

Approved follow-up statuses:

1. `Scheduled`
2. `Overdue`
3. `Completed`
4. `Cancelled`
5. `Superseded`

When a new follow-up is scheduled while a current follow-up exists:

- The existing current follow-up must be marked `Superseded`.
- The prior follow-up details and history must remain preserved.
- The new follow-up becomes the only current scheduled follow-up.
- The change must be recorded in `action_history`.

### 4.14 One-Current-Follow-Up Enforcement

The implementation must prevent two current scheduled follow-ups for one claim.

Required approach:

1. Define the current follow-up using an explicit follow-up state, such as `Scheduled`.
2. Use a database transaction when creating or replacing a follow-up.
3. Mark the existing current scheduled follow-up as `Superseded`.
4. Insert the new `Scheduled` follow-up.
5. Write the corresponding `action_history` record.
6. Commit only if every operation succeeds.
7. Test concurrent and duplicate submissions.

If the target MySQL/MariaDB version supports a suitable partial or filtered unique index, use it only after verifying compatibility. Otherwise, use a transaction and locking strategy and test concurrency.

### 4.15 History-Table Responsibilities

| Table | Approved responsibility |
|---|---|
| `follow_ups` | Current and historical follow-up records. |
| `action_history` | Workflow actions: follow-up scheduled, completed, cancelled, superseded, claim closed, and claim reopened. |
| `claim_activity_logs` | Audit events for material claim changes: status, priority, assignment, and field changes. |
| `claim_history` | Claim lifecycle and major status-transition history. |

History and audit records must not be deleted through normal application actions.

---

## 5. Roles, Permissions, and Authorization

### 5.1 Role Values

| Stored role value | Display label |
|---|---|
| `Claim_Adjuster` | Claim Adjuster |
| `Supervisor` | Supervisor |
| `Director` | Director |

### 5.2 Role Model

- Each user has exactly one role.
- The role is stored directly on the user record.
- Separate `roles` and `user_roles` tables are not required.

### 5.3 Claim Visibility

| Role | Claim visibility |
|---|---|
| `Claim_Adjuster` | Only claims assigned to them |
| `Supervisor` | All claims |
| `Director` | All claims |

### 5.4 Claim Work Permissions

All three roles may perform authorized claim work, including:

- Create claims.
- View claims within their authorized scope.
- Update claim details.
- Change claim status and priority.
- Schedule, update, and complete follow-ups.
- Record action history.

### 5.5 Reassignment and Take-Over

- Only `Supervisor` and `Director` can reassign claims.
- Only `Supervisor` and `Director` can take over claims.
- A Supervisor or Director may take over an unassigned claim or a claim already assigned to another user.
- Take-over is allowed for claims in any status, including `Resolved` and `Closed`.
- Taking over a `Closed` claim must automatically reopen the claim.
- A closed claim that is taken over must require a new follow-up.
- Before take-over is saved, the system must show a confirmation warning identifying the current handler, if any, and stating that the handler will be replaced.
- Every reassignment and take-over must be recorded in assignment history.

### 5.6 Claim Closure and Reopening

- `Claim_Adjuster`, `Supervisor`, and `Director` can close a claim within their authorized scope.
- A Claim Adjuster may close only a claim assigned to them.
- `Claim_Adjuster`, `Supervisor`, and `Director` can reopen a `Resolved` or `Closed` claim within their authorized scope.
- A Claim Adjuster may reopen only a claim assigned to them.
- Closing a claim cancels any current scheduled follow-up while preserving history.
- Reopening a claim requires a current handler and dated follow-up if either is missing.

### 5.7 User Management

- Supervisor and Director can manage user accounts.
- Claim Adjuster cannot manage user accounts.
- Only Director can create or manage Supervisor and Director accounts.
- Supervisor can manage Claim Adjuster accounts.
- User accounts must be deactivated, not deleted.
- A user cannot delete their own account.

### 5.8 Guidance Permissions

- Claim Adjuster can view guidance only.
- Supervisor can create and edit guidance.
- Director can create, edit, approve, and deactivate guidance.
- Every guidance edit must create a new revision record.
- Previous guidance revisions remain viewable and must not be deleted through the application.

### 5.9 Guidance Lifecycle Statuses

1. `Draft`
2. `Approved`
3. `Inactive`

---

## 6. Application Architecture

### 6.1 Architecture Principles

Use a small procedural PHP application with clear separation of responsibilities.

Required shared modules or includes:

- Configuration and database connection.
- Authentication and session checks.
- Role and record-level authorization.
- Input validation and normalization.
- Output escaping helpers.
- CSRF token generation and verification.
- Shared layout, sidebar, navigation, and page templates.
- Claim and follow-up business rules.
- History and audit-writing helpers.
- Centralized safe error handling.

The final directory structure must preserve the approved project architecture and must not introduce an unapproved framework.

### 6.2 Request Handling

- Use POST for operations that create, update, complete, cancel, close, reopen, reassign, take over, deactivate, or otherwise change state.
- Do not change database state through GET requests.
- Before processing a state-changing request, verify:
  1. Authenticated session.
  2. Active account.
  3. Role permission.
  4. Record-level authorization.
  5. CSRF token.
  6. HTTP method.
  7. Required input.
  8. Business rules.
- Use transactions for related multi-table updates.
- Return clear success or failure feedback without exposing SQL, credentials, stack traces, or internal paths.
- Make critical state-changing operations safe against duplicate submission.

### 6.3 Date and Time Handling

- Store dates and timestamps in UTC.
- Display dates and calculate the current date using `Asia/Manila`.
- Follow-up due dates are date-only values.
- Display dates as `MM/DD/YYYY`.
- Display timestamps as `MM/DD/YYYY hh:mm AM/PM` in `Asia/Manila` time.
- A `Scheduled` follow-up is `Overdue` when its due date is before the current Philippine date.
- A follow-up due today belongs in `Due Today`, not `Overdue`.
- A follow-up with a future due date within the next seven days belongs in `Upcoming`.
- Do not use the label `Due soon`; use `Upcoming`.

---

## 7. Security Requirements

### 7.1 Authentication

- Store passwords only as secure password hashes.
- Use PHP’s supported password-hashing functions.
- Regenerate the session ID after successful login and appropriate privilege changes.
- Reject inactive accounts.
- Apply session expiration and logout behavior.
- Use secure cookie settings where HTTPS is available, including `HttpOnly` and `Secure`.
- Apply login throttling or equivalent protection appropriate to InfinityFree.

### 7.2 Authorization

- Deny access by default when a role or permission is missing or invalid.
- Enforce authorization in PHP before reading or changing protected records.
- Apply record-level visibility rules to database queries, not only navigation.
- Prevent users from escalating privileges through modified request data.
- Prevent a Claim Adjuster from accessing another adjuster’s claims.
- Prevent unauthorized reassignment, take-over, user management, store modification, and service-call access.

### 7.3 Input and Output Safety

- Validate all user input on the server.
- Use allowlists for statuses, priorities, roles, party types, claim types, guidance statuses, and sort fields.
- Use MySQLi prepared statements for user-controlled values.
- Escape output according to HTML, attribute, JavaScript, and URL context.
- Protect state-changing requests with CSRF tokens.
- Do not expose credentials, SQL errors, stack traces, file paths, or internal configuration.

### 7.4 Audit Protection

- History and audit records must not be editable or deletable through normal application screens.
- Prefer append-only inserts for history.
- Use transactions so operational changes and history records commit together.
- Do not store passwords, session tokens, or other secrets in history or audit records.

---

## 8. Performance, Usability, and Maintainability

### 8.1 Performance

- Use indexes that support dashboard queues, claim filters, joins, and follow-up date queries.
- Use server-side pagination.
- Claims lists must use 25 records per page.
- Avoid retrieving all claims when a filtered or paginated result is needed.
- Validate index choices using representative queries and `EXPLAIN` where possible.

### 8.2 Usability

- Display clear claim references, statuses, priorities, handlers, and follow-up dates.
- Use readable labels alongside colors.
- Provide useful empty, loading, validation-error, and success states.
- Keep essential workflows usable on desktop and mobile screens.

### 8.3 Maintainability

- Use consistent table, column, function, route, and status naming.
- Keep schema changes in version-controlled SQL migration or installation scripts.
- Document implementation decisions and deviations.
- Do not add, remove, or rename approved features, tables, or fields without approval.

---

## 9. Operational Requirements

### 9.1 Configuration and Secrets

- Keep database credentials out of public files and version control.
- Use separate local and production configuration.
- Disable debug mode in production.
- Restrict access to configuration and diagnostic files.
- Rotate credentials if exposed.

### 9.2 Backup and Restore

Document:

1. What database data is included.
2. How and how often backups are created.
3. Where backups are stored and who can access them.
4. Retention period.
5. Restoration procedure.
6. Validation steps after restoration.
7. Handling of failed or incomplete backups.

Do not claim backups are successful until a restore test passes.

### 9.3 Logging and Errors

- Show users safe, actionable messages.
- Log technical errors securely where hosting permits.
- Never expose SQL, credentials, stack traces, or internal paths.
- Record important business events in approved history and audit tables.
- Avoid logging sensitive claim or user data unnecessarily.

### 9.4 Deployment

- Maintain a known-good source-controlled release.
- Test schema creation and application setup in a non-production environment first.
- Deploy only reviewed code and SQL.
- Verify environment configuration, database connection, permissions, HTTPS, and critical workflows after deployment.
- Maintain a rollback plan.
- Do not assume SSH, cron, or other server capabilities are available.

---

## 10. Acceptance Criteria

V1.0 is technically acceptable when:

- The application runs on the approved InfinityFree environment using procedural PHP and MySQLi.
- The approved 11-table model is implemented and verified.
- `claim_attachments`, attachment functionality, file uploads, email reminders, notifications, Excel import, and external integrations are absent.
- `claims.job_id` uniqueness is enforced.
- Every claim has exactly one required store and exactly one required claim party.
- Each claim has zero or one service-call record.
- Populated `sc_job_id` values are unique.
- A claim cannot have more than one current scheduled follow-up.
- Historical follow-ups, action history, activity logs, claim history, assignment history, and guidance revisions are preserved.
- Role authorization, record visibility, CSRF protection, input validation, output escaping, and safe errors are enforced.
- Approved status transitions, priorities, claim types, party types, and guidance statuses are enforced.
- Date handling uses UTC storage and `Asia/Manila` display/current-date logic.
- Claims lists use 25 records per page and the approved columns, filters, and dashboard metrics.
- Backup and restore procedures are documented and tested.
- Required functional, security, workflow, and hosting-compatibility tests pass.

---

## 11. Implementation Verification Items

Confirm before or during implementation:

1. Exact InfinityFree-supported PHP and MySQL/MariaDB versions.
2. Final column definitions, primary-key types, defaults, and timestamp conventions.
3. Final representation of guidance and guidance revisions within the approved 11-table model.
4. Exact database-compatible mechanism for enforcing one current scheduled follow-up.
5. Exact foreign-key and deletion/deactivation policies.
6. Complete role/permission matrix and record-visibility rules.
7. Exact fields requiring before/after audit values.
8. InfinityFree backup, HTTPS, PHP extension, database-version, and deployment capabilities.

Any material deviation requires user approval before implementation.

---

**End of TRD — Version 1.1**
