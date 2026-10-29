# QuickContractors Claims Tracker System

## Product Requirements Document (PRD) — Version 1.1

**Document status:** Approved for implementation  
**Version:** 1.1  
**Supersedes:** PRD Version 1.0  
**Date:** October 29, 2026  
**Authoritative reconciliation source:** `Claims_Tracker_System_Document_Reconciliation_Decision_Record_V1.1.md`  
**Implementation status:** Not started. No production database or application exists yet.

---

## 1. Document Purpose

This PRD defines the approved product requirements for **QuickContractors Claims Tracker System V1.0**.

Where this document conflicts with an earlier project document, this PRD must be interpreted together with the approved **Document Reconciliation Decision Record V1.1**. The Decision Record is the controlling reconciliation reference.

No PHP, SQL, or UI implementation may proceed against requirements that conflict with this PRD or the V1.1 Decision Record.

---

## 2. System Overview

### 2.1 System Name

QuickContractors Claims Tracker System

### 2.2 System Description

The Claims Tracker System is a secure, browser-based internal application that centralizes the tracking and management of claims involving:

- Third-party contractors.
- QuickContractors E-tech technicians.

The system helps authorized users:

- Record and maintain claims.
- Identify overdue, due-today, and upcoming follow-ups.
- Maintain clear claim ownership.
- Schedule, update, complete, cancel, and supersede follow-ups.
- Record claim actions and preserve history.
- Reassign or take over claims.
- Provide management visibility into workload, overdue work, and claim progress.
- Maintain secure role-based access and reliable data integrity.

### 2.3 Business Context

The initial operating context is approximately 400 open claims handled by five team members, including Claim Adjusters, the Supervisor, and the Director.

Approximately 100 claims are expected to be entered initially through manual data entry.

The system is designed around the claims team’s daily workflow, especially reviewing overdue follow-ups and determining the next required action for each claim.

### 2.4 Intended Users

| Role           | Stored role value | Primary responsibilities                                                                                                                                                         |
| -------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Claim Adjuster | `Claim_Adjuster`  | Manage only claims assigned to them; create and update claims; schedule, update, and complete follow-ups; record actions; close or reopen assigned claims; view guidance.        |
| Supervisor     | `Supervisor`      | View and manage all claims; reassign and take over claims; manage Claim Adjuster accounts; create and edit guidance; monitor workload and overdue work.                          |
| Director       | `Director`        | View and manage all claims; reassign and take over claims; manage all user accounts; create, edit, approve, and deactivate guidance; review overall claim activity and workload. |

Third-party contractors and E-tech technicians are claim parties, not system login users.

### 2.5 Technology Stack

| Category                 | Approved technology                    | Implementation rule                                                      |
| ------------------------ | -------------------------------------- | ------------------------------------------------------------------------ |
| Backend                  | Procedural PHP                         | No Laravel or other backend framework.                                   |
| Database access          | MySQLi                                 | Use prepared statements for all user-supplied values. PDO is excluded.   |
| Database                 | MySQL-compatible database using InnoDB | Use transactions and foreign keys where supported.                       |
| Frontend                 | HTML5, CSS, Vanilla JavaScript         | No frontend framework is required.                                       |
| UI framework             | Bootstrap 5.3                          | Use approved responsive layout and components.                           |
| Icons                    | Font Awesome 6                         | Use accessible labels for icon-only controls.                            |
| Alerts and confirmations | SweetAlert2                            | Use for required confirmations and user feedback.                        |
| Hosting                  | InfinityFree                           | Verify supported PHP, MySQL, HTTPS, backup, and deployment capabilities. |
| Local PHP environment    | Herd                                   | Development only.                                                        |
| Local database service   | DBngin                                 | Development only.                                                        |
| Database management      | TablePlus                              | Development and database inspection only.                                |
| File transfer            | FileZilla                              | Deployment utility only where required by the host.                      |
| UI/UX design             | Figma                                  | Optional design tool only.                                               |

### 2.6 V1.0 Scope

#### Included

- Authentication, logout, and role-based authorization.
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

#### Excluded from V1.0

The following are explicitly excluded and must not be implemented, displayed as available, or represented by placeholder functionality:

- Claim attachments.
- File uploads.
- Attachment management.
- Excel import.
- Bulk import.
- Email reminders.
- Automated notifications.
- A notification subsystem.
- External contractor or store-system integrations.
- Multi-company or multi-tenant SaaS functionality.
- Laravel, PDO, or any unapproved backend framework.
- Separate `roles` and `user_roles` tables.

---

## 3. Product Goal

### 3.1 Primary Goal

Provide QuickContractors with a centralized claims tracking system that enables authorized users to manage claims, monitor follow-ups, maintain ownership, preserve history, and give management reliable visibility into outstanding work.

### 3.2 Specific Goals

| Goal                               | Requirement                                                                                                           | Success criterion                                                              |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Improve daily follow-up management | Display Overdue, Due Today, and Upcoming follow-up queues.                                                            | Users can identify required daily work without manually reviewing every claim. |
| Establish claim ownership          | Require an assigned handler for active claims and preserve assignment history.                                        | The current handler is always clear for active claims.                         |
| Improve traceability               | Preserve follow-up, action, activity, lifecycle, and assignment history.                                              | Users can understand what happened, when it happened, and who performed it.    |
| Improve prioritization             | Support six approved priority levels and distinguish urgent or escalated claims.                                      | High-priority claims are identifiable without confusing priority with status.  |
| Provide management visibility      | Give Supervisor and Director appropriate access to claims, workload, overdue work, and user management.               | Management can monitor operations without bypassing authorization.             |
| Protect claim records              | Enforce authentication, server-side authorization, validation, prepared statements, CSRF protection, and safe errors. | Unauthorized users cannot access or modify restricted records.                 |

---

## 4. Functional Requirements

### 4.1 Authentication and Access Control

#### Requirements

- Users must authenticate before accessing protected pages.
- The system must support exactly three roles: `Claim_Adjuster`, `Supervisor`, and `Director`.
- Each user must have exactly one role.
- The role must be stored directly on the user record.
- Separate `roles` and `user_roles` tables must not be created.
- Authorization must be enforced on the server for every protected page, API-like endpoint, form submission, and database operation.
- Hiding a menu item or button in the browser is not authorization.
- Users must not access records by changing URLs, form values, or request parameters.
- Passwords must be stored only as secure hashes using PHP password-hashing functions.
- Sessions must be secured, expired appropriately, and regenerated after successful login.
- State-changing requests must be protected against CSRF.
- Inactive user accounts must not authenticate or continue using an active session.
- Unauthorized requests must be rejected with a safe, non-technical error response.

#### Acceptance Criteria

1. Unauthenticated users cannot access protected pages.
2. A Claim Adjuster cannot access another adjuster’s claims.
3. A Claim Adjuster cannot manage users, stores beyond view-only access, or service calls.
4. A Supervisor cannot create or manage Supervisor or Director accounts.
5. Changing a claim ID, user ID, or other record identifier in a request does not bypass authorization.
6. Password hashes are never exposed in normal application responses.

---

### 4.2 Claims Dashboard

#### Required Metrics

The dashboard must display the following five metrics:

1. Open Claims.
2. Overdue Follow-ups.
3. Due Today.
4. Upcoming Follow-ups.
5. Claims Requiring Attention.

#### Claims Requiring Attention

A claim belongs in **Claims Requiring Attention** when either condition is true:

- The claim has an overdue follow-up.
- The claim priority is `Urgent` or `Escalated`.

#### Queue Definitions

| Queue     | Approved rule                                                                 |
| --------- | ----------------------------------------------------------------------------- |
| Overdue   | A `Scheduled` follow-up whose due date is before the current Philippine date. |
| Due Today | A `Scheduled` follow-up whose due date equals the current Philippine date.    |
| Upcoming  | A `Scheduled` follow-up with a future due date within the next 7 days.        |

#### Requirements

- Dashboard counts must be calculated from authoritative database records.
- Counts and lists must respect the user’s role and claim-visibility scope.
- A Claim Adjuster sees only claims assigned to them.
- A Supervisor and Director see all claims.
- Completed, cancelled, and superseded follow-ups must not appear in pending queues.
- Users must be able to open a claim from a dashboard entry.
- The dashboard must display clear empty states when no records match.

#### Acceptance Criteria

1. An overdue scheduled follow-up appears only in Overdue.
2. A scheduled follow-up due today appears only in Due Today.
3. A scheduled follow-up due within the next seven days appears in Upcoming.
4. A completed, cancelled, or superseded follow-up does not appear as a pending follow-up.
5. Dashboard counts match the claims and follow-ups visible to the signed-in user.

---

### 4.3 Claim Creation

Every new claim must have:

- A unique `job_id`.
- Exactly one required store.
- One approved claim type.
- Exactly one required claim party.
- An assigned handler.
- One current scheduled follow-up with a due date.
- A priority.

#### Default Values

| Field          | Default value |
| -------------- | ------------- |
| Claim status   | `Open`        |
| Claim priority | `Normal`      |

#### Approved Claim Types

1. `Leak`
2. `Damage`
3. `Outdoor Damage`

#### Requirements

- `job_id` must be unique.
- Duplicate `job_id` submissions must be rejected safely.
- The selected store must exist and be active.
- The claim type must be one of the approved values.
- The claim party must be created or associated according to the approved one-to-one relationship.
- The assigned handler must be an active user authorized to receive the assignment.
- The initial follow-up due date is required.
- The initial follow-up status must be `Scheduled`.
- Server-side validation is mandatory.

#### Acceptance Criteria

1. A valid claim can be created with all required data.
2. A duplicate `job_id` is rejected.
3. A claim cannot be created without a store, claim party, handler, follow-up due date, or priority.
4. Invalid claim types, statuses, priorities, users, and stores are rejected.
5. Claim creation records the required history and audit information.

---

### 4.4 Claim Party Management

Each claim must have exactly one `claim_parties` record.

#### Approved Party Types

| Stored value             | Meaning                                                   |
| ------------------------ | --------------------------------------------------------- |
| `Third_Party_Contractor` | Third-party contractor involved in the claim.             |
| `E_Tech`                 | QuickContractors E-tech technician involved in the claim. |

#### Required Fields

- `party_type`
- `contractor_email_address`
- `legal_company_name`
- `company_name`

#### Requirements

- The claim-to-party relationship must be one-to-one.
- A claim party record is required for every claim.
- The party record may be edited or changed after claim creation.
- Every claim-party change must be recorded in the appropriate claim history or activity record.
- Invalid party types and missing required fields must be rejected.
- Email addresses must be validated on the server.

#### Acceptance Criteria

1. A claim cannot exist without exactly one claim party.
2. A second active claim-party record cannot be created for the same claim.
3. Party changes are preserved in history or activity records.
4. Invalid party data cannot be saved.

---

### 4.5 Claim Status Management

#### Approved Statuses

| Status        | Definition                                                     |
| ------------- | -------------------------------------------------------------- |
| `Open`        | The claim is active and requires work.                         |
| `In Progress` | Work on the claim is underway.                                 |
| `Waiting`     | Progress depends on a response, information, or another event. |
| `Resolved`    | The claim issue has been resolved.                             |
| `Closed`      | The claim is closed.                                           |

#### Approved Status Transitions

| From status   | Allowed transitions                            |
| ------------- | ---------------------------------------------- |
| `Open`        | `In Progress`, `Waiting`, `Resolved`, `Closed` |
| `In Progress` | `Waiting`, `Resolved`, `Closed`                |
| `Waiting`     | `In Progress`, `Resolved`, `Closed`            |
| `Resolved`    | `Closed`, `Open`                               |
| `Closed`      | `Open`                                         |

#### Requirements

- Status values must be validated on the server.
- Prohibited transitions must be rejected.
- Status changes must be recorded in claim history and/or claim activity logs according to the approved history responsibilities.
- Closing a claim must cancel any current scheduled follow-up while preserving its history.
- Reopening a claim requires a current handler and dated follow-up if either is missing.
- Status must remain separate from priority.

#### Acceptance Criteria

1. Only approved transitions can be saved.
2. Invalid status values are rejected.
3. Closing a claim cancels its current scheduled follow-up without deleting history.
4. Reopening a claim enforces handler and follow-up requirements.
5. Status changes are traceable.

---

### 4.6 Claim Priority Management

#### Approved Priority Values

| Priority     | Display color | Meaning                                                       |
| ------------ | ------------- | ------------------------------------------------------------- |
| `Non-Urgent` | Green         | Routine work with no immediate action required.               |
| `Low`        | Blue          | Low-impact work requiring attention.                          |
| `Normal`     | Yellow        | Standard work requiring normal attention.                     |
| `High`       | Amber         | Work requiring increased attention.                           |
| `Urgent`     | Orange        | Work requiring prompt action.                                 |
| `Escalated`  | Red           | Work requiring escalation and immediate management attention. |

#### Requirements

- The system must support exactly these six priority values.
- Priority must be displayed with a readable label, not color alone.
- Priority must remain independent from claim status.
- Priority changes must be recorded in claim activity logs.
- Invalid priority values must be rejected.

#### Acceptance Criteria

1. Users can select only approved priority values.
2. Priority is visible on claim records, lists, and dashboard queues.
3. Priority changes are traceable.
4. An `Urgent` or `Escalated` claim appears in Claims Requiring Attention.

---

### 4.7 Follow-Up and Next-Action Management

The standard table name is `follow_ups`.

#### Approved Follow-Up Statuses

1. `Scheduled`
2. `Overdue`
3. `Completed`
4. `Cancelled`
5. `Superseded`

#### Core Rules

- A claim may have only one current scheduled follow-up at a time.
- Historical follow-ups must be preserved.
- The current follow-up follows the claim’s current handler.
- Historical follow-ups retain their original handler.
- The follow-up due date is a date-only value; users do not enter a follow-up time.
- A new follow-up supersedes the existing current scheduled follow-up.
- The superseded follow-up must be marked `Superseded`.
- The superseded follow-up’s details and history must remain preserved.
- The new follow-up becomes the only current scheduled follow-up.
- The superseding operation must be recorded in `action_history`.
- Closing a claim cancels the current scheduled follow-up.
- Completed, cancelled, and superseded follow-ups must not appear in pending queues.

#### Requirements

- Authorized users can schedule, update, and complete follow-ups.
- The due date must be validated on the server.
- The system must prevent two current scheduled follow-ups for one claim, including under duplicate or concurrent requests.
- Follow-up status and claim status are separate concepts.
- Follow-up queues must use Philippine date logic.

#### Acceptance Criteria

1. A claim cannot have two current scheduled follow-ups.
2. Scheduling a new follow-up marks the prior current follow-up as `Superseded`.
3. Historical follow-ups retain their original handler and details.
4. A follow-up due before the current Philippine date is Overdue.
5. A follow-up due today is Due Today, not Overdue.
6. A follow-up due within the next seven days is Upcoming.
7. Closing a claim cancels its current follow-up.

---

### 4.8 Claim Action History

`action_history` records workflow actions.

#### Required Recorded Events

At minimum, `action_history` must record:

- Follow-up scheduled.
- Follow-up completed.
- Follow-up cancelled.
- Follow-up superseded.
- Claim closed.
- Claim reopened.

#### Requirements

- Each action must identify the affected claim.
- Each action must identify the acting user where applicable.
- Each action must retain a timestamp.
- Historical records must not be edited or deleted through normal application screens.
- History insertion must occur in the same transaction as the related operational change where applicable.
- Previous actions must remain available after new actions are recorded.

#### Acceptance Criteria

1. Recording a new action does not silently replace prior actions.
2. Authorized users can review historical claim activity.
3. Unauthorized users cannot modify or delete history.
4. Failed operations do not create incomplete or orphaned history records.

---

### 4.9 Claim Activity Logs

`claim_activity_logs` records audit events for material claim changes.

#### Required Audit Events

At minimum, the system must record activity for changes to:

- Claim status.
- Claim priority.
- Claim assignment.
- Material claim field changes.
- Claim-party changes.

#### Requirements

- Activity records must identify the claim, acting user, event, timestamp, and relevant change details.
- Before-and-after values must be recorded where appropriate and where doing so does not expose secrets.
- Activity records must not be editable or deletable through normal application screens.
- Activity logging must be included in the transaction for the related operational change where applicable.

#### Acceptance Criteria

1. Material claim changes are traceable.
2. Assignment changes are traceable.
3. Claim-party changes are traceable.
4. Users cannot alter audit records through the application.

---

### 4.10 Claim Lifecycle History

`claim_history` records claim lifecycle and major status-transition history.

#### Requirements

- Claim creation, closure, reopening, and major status transitions must be traceable.
- History records must be preserved even after later changes.
- History records must not be deleted through normal application actions.
- The distinction between `claim_history`, `claim_activity_logs`, and `action_history` must follow the approved history-table responsibilities.

#### Acceptance Criteria

1. Claim lifecycle events are preserved.
2. Major status transitions are traceable.
3. History is not lost when a claim is edited, reassigned, closed, or reopened.

---

### 4.11 Claim Assignment, Reassignment, and Take-Over

#### Assignment Rules

- Every new claim must have an assigned handler.
- Active claims must have a current handler and dated follow-up.
- `Resolved` and `Closed` claims are exempt from the active-claim handler/action requirement.

#### Reassignment Rules

- Only `Supervisor` and `Director` can reassign claims.
- Only `Supervisor` and `Director` can take over claims.
- A Supervisor or Director may take over an unassigned claim or a claim already assigned to another user.
- Take-over is allowed for claims in any status, including `Resolved` and `Closed`.
- Taking over a `Closed` claim must automatically reopen the claim.
- A closed claim that is taken over must require a new follow-up.
- Before take-over is saved, the system must show a confirmation warning identifying the current handler, if any, and stating that the handler will be replaced.
- Every reassignment and take-over must be recorded in assignment history.

#### Requirements

- The current handler must be updated only after authorization, validation, confirmation, and transaction success.
- Failed reassignment or take-over operations must not leave inconsistent claim or assignment-history records.
- Assignment history must identify the previous handler, new handler, acting user, timestamp, and operation type.

#### Acceptance Criteria

1. A Claim Adjuster cannot reassign or take over a claim.
2. A Supervisor or Director can reassign or take over an authorized claim.
3. Taking over a Closed claim reopens it and requires a new follow-up.
4. Assignment history is preserved for every reassignment and take-over.
5. Failed operations do not create inconsistent records.

---

### 4.12 Claim Closure and Reopening

#### Closure Rules

- `Claim_Adjuster`, `Supervisor`, and `Director` can close a claim within their authorized scope.
- A Claim Adjuster may close only a claim assigned to them.
- Closing a claim cancels any current scheduled follow-up while preserving history.
- Closure must be confirmed using SweetAlert2.

#### Reopening Rules

- `Claim_Adjuster`, `Supervisor`, and `Director` can reopen a `Resolved` or `Closed` claim within their authorized scope.
- A Claim Adjuster may reopen only a claim assigned to them.
- Reopening a claim requires a current handler and dated follow-up if either is missing.
- Reopening must be confirmed using SweetAlert2.

#### Acceptance Criteria

1. Unauthorized closure or reopening is rejected.
2. Closing a claim cancels its current scheduled follow-up.
3. Reopening enforces handler and follow-up requirements.
4. Closure and reopening are recorded in the appropriate history tables.

---

### 4.13 Claim Search, Filters, and Pagination

#### Required Claims-List Columns

1. Job ID
2. Store
3. Claim Type
4. Party Type
5. Assigned Handler
6. Claim Status
7. Priority
8. Next Follow-up Date
9. Last Follow-up Date
10. Follow-up Status
11. Actions

#### Required Search and Filters

- Job ID search.
- Claim status filter.
- Priority filter.
- Assigned-handler filter, limited to Claim Adjusters.
- Follow-up state filter: `Overdue`, `Due Today`, and `Upcoming`.
- Store filter.
- Party type filter.
- Date-range filter based on follow-up due date.

#### Requirements

- Claims lists must use 25 records per page.
- Search and filter parameters must be validated on the server.
- Dynamic SQL identifiers, including sort fields, must come from strict server-side allowlists.
- Results must respect role-based and record-level authorization.
- Users must be able to clear filters.
- Empty results must display a clear message.

#### Acceptance Criteria

1. Searching an existing Job ID returns the authorized matching claim.
2. Combining filters returns only claims matching all criteria.
3. Unauthorized claims never appear in results.
4. Pagination returns 25 records per page.
5. Invalid or malicious filter values are safely rejected.

---

### 4.14 Store Management

#### Rules

- Every claim must be linked to exactly one store.
- A store may have many claims.
- Stores must be deactivated rather than deleted while claims reference them.
- Store deactivation requires a SweetAlert2 confirmation dialog.

#### Menu Visibility

| Role           | Store access         |
| -------------- | -------------------- |
| Claim Adjuster | View only            |
| Supervisor     | Full approved access |
| Director       | Full approved access |

#### Acceptance Criteria

1. A claim cannot be created without a valid store.
2. A store referenced by claims cannot be deleted through normal application actions.
3. Store deactivation is confirmed and traceable.
4. Claim Adjusters cannot modify stores.

---

### 4.15 Service-Call Management

#### Rules

- Each claim may have zero or one service-call record.
- `sc_job_id` is nullable.
- When populated, `sc_job_id` must be unique.
- Service Calls are visible only to Supervisor and Director.

#### Requirements

- A claim cannot have more than one service-call record.
- Duplicate populated `sc_job_id` values must be rejected.
- Service-call changes must follow server-side validation and authorization rules.
- Service-call records must be associated with the correct claim.

#### Acceptance Criteria

1. A Claim Adjuster cannot access Service Calls.
2. A Supervisor or Director can manage authorized service-call records.
3. A claim cannot have multiple service-call records.
4. Duplicate populated `sc_job_id` values are rejected.

---

### 4.16 User Management

#### Rules

- Supervisor and Director can manage user accounts.
- Claim Adjuster cannot manage user accounts.
- Only Director can create or manage Supervisor and Director accounts.
- Supervisor can manage Claim Adjuster accounts.
- User accounts must be deactivated, not deleted.
- A user cannot delete their own account.
- User deactivation requires a SweetAlert2 confirmation dialog.

#### Acceptance Criteria

1. A Claim Adjuster cannot access user management.
2. A Supervisor can manage Claim Adjuster accounts only.
3. Only a Director can create or manage Supervisor and Director accounts.
4. Users are deactivated rather than deleted.
5. A user cannot delete their own account.

---

### 4.17 Claim Guidance

#### Guidance Statuses

1. `Draft`
2. `Approved`
3. `Inactive`

#### Permissions

| Role           | Guidance permission                   |
| -------------- | ------------------------------------- |
| Claim Adjuster | View only                             |
| Supervisor     | Create and edit                       |
| Director       | Create, edit, approve, and deactivate |

#### Requirements

- Every guidance edit and every guidance status change (approval or deactivation) must create a new revision record that stores the resulting status and the acting user.
- Previous guidance revisions must remain viewable.
- Previous revisions must not be deleted through the application.
- Guidance deactivation requires a SweetAlert2 confirmation dialog.
- Guidance must not introduce file-upload functionality.

#### Acceptance Criteria

1. Claim Adjusters can view but not edit guidance.
2. Supervisors can create and edit guidance.
3. Directors can create, edit, approve, and deactivate guidance.
4. Every edit, approval, and deactivation creates a revision.
5. Prior revisions remain available.

---

## 5. Non-Functional Requirements

| Area            | Requirement                                                                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Security        | Enforce authentication, server-side authorization, prepared statements, output escaping, CSRF protection, secure sessions, and safe errors. |
| Data integrity  | Enforce unique identifiers, foreign keys, required fields, controlled values, and transaction-safe related updates.                         |
| Auditability    | Preserve follow-up, action, activity, lifecycle, assignment, and guidance-revision history.                                                 |
| Usability       | Make queues, ownership, status, priority, and next actions clear.                                                                           |
| Accessibility   | Use readable labels, sufficient contrast, keyboard-accessible controls, and visible focus states.                                           |
| Responsiveness  | Support desktop, tablet, and mobile screen sizes.                                                                                           |
| Reliability     | Prevent partial updates through transactions and safe error handling.                                                                       |
| Performance     | Use appropriate indexes, server-side pagination, and efficient dashboard queries.                                                           |
| Maintainability | Use procedural PHP, MySQLi, consistent naming, version-controlled SQL, and documented decisions.                                            |
| Recovery        | Document and test database backup and restoration procedures.                                                                               |
| Date handling   | Store dates and timestamps in UTC; display dates and calculate the current date using `Asia/Manila`.                                        |

---

## 6. Date, Time, and Queue Rules

- Store dates and timestamps in UTC.
- Display dates and calculate the current date using `Asia/Manila`.
- Follow-up due dates are date-only values.
- Display dates as `MM/DD/YYYY`.
- Display timestamps as `MM/DD/YYYY hh:mm AM/PM` in `Asia/Manila` time.
- A `Scheduled` follow-up is `Overdue` when its due date is before the current Philippine date.
- A follow-up due today belongs in `Due Today`, not `Overdue`.
- A follow-up with a future due date within the next seven days belongs in `Upcoming`.
- The label `Due soon` must not be used; use `Upcoming`.

---

## 7. User Interface Requirements

### 7.1 Sidebar Menu

The sidebar must include:

1. Dashboard
2. Claims
3. Follow-ups
4. Action History
5. Claim Guidance
6. Users
7. Stores
8. Service Calls

### 7.2 Role-Based Menu Visibility

| Menu           |  Claim Adjuster | Supervisor | Director |
| -------------- | --------------: | ---------: | -------: |
| Dashboard      |             Yes |        Yes |      Yes |
| Claims         |             Yes |        Yes |      Yes |
| Follow-ups     |             Yes |        Yes |      Yes |
| Action History |             Yes |        Yes |      Yes |
| Claim Guidance |             Yes |        Yes |      Yes |
| Users          |              No |        Yes |      Yes |
| Stores         | Yes — view only |        Yes |      Yes |
| Service Calls  |              No |        Yes |      Yes |

### 7.3 Required Confirmation Dialogs

SweetAlert2 confirmation dialogs are required for:

- Take over claim.
- Reassign claim.
- Close claim.
- Reopen claim.
- Cancel follow-up.
- Deactivate user.
- Deactivate store.
- Deactivate guidance.

### 7.4 Visual Standards

The interface must follow the approved Design Brief:

- Sidebar: `#343A40`
- Main content background: `#F4F6F9`
- Primary accent: `#0D6EFD`
- Typography: Roboto
- Content surfaces: white
- Primary text: `#212529`
- Secondary text: `#6C757D`
- Borders: `#DEE2E6`

Color must never be the only indicator of status, priority, or urgency.

---

## 8. Data Integrity Requirements

- Use the approved 13-table model.
- `claims.job_id` is the unique business claim identifier.
- Every claim must have exactly one store.
- Every claim must have exactly one claim party.
- Each claim may have zero or one service call.
- A claim may have only one current scheduled follow-up.
- Historical follow-ups must be preserved.
- `sc_job_id` is nullable but unique when populated.
- `party_type` must be `Third_Party_Contractor` or `E_Tech`.
- Users have exactly one role stored directly on the user record.
- History and audit records must not be deleted through normal application actions.
- Related updates must use transactions where partial completion would create inconsistent data.

---

## 9. Testing and Acceptance Plan

| Test area          | Required verification                                                                                       |
| ------------------ | ----------------------------------------------------------------------------------------------------------- |
| Database           | All 13 approved tables, relationships, constraints, indexes, and seed values match the approved schema.     |
| Authentication     | Login, logout, inactive accounts, session expiration, and denied access work correctly.                     |
| Authorization      | Each role can perform only its approved operations.                                                         |
| Claim creation     | Required store, party, handler, follow-up, `job_id`, type, status, and priority rules are enforced.         |
| Claim status       | Approved transitions pass; prohibited transitions fail.                                                     |
| Claim priority     | All six values work and urgent/escalated claims appear in Claims Requiring Attention.                       |
| Follow-ups         | One-current-follow-up rule, superseding, completion, cancellation, and queue classification work correctly. |
| History            | Action, activity, lifecycle, assignment, and guidance-revision history are preserved.                       |
| Assignment         | Reassignment and take-over follow role, confirmation, reopening, and history rules.                         |
| Search and filters | All approved filters, pagination, authorization, and empty states work correctly.                           |
| UI                 | Sidebar visibility, columns, dashboard metrics, dialogs, and responsive behavior match the approved design. |
| Security           | CSRF, XSS, SQL injection, direct URL access, and malformed-request tests pass.                              |
| Recovery           | Backup and restore procedures are documented and tested.                                                    |
| Deployment         | The application works within InfinityFree’s supported PHP, MySQL, HTTPS, and hosting limitations.           |

---

## 10. Scope Control

Any change to approved scope, database structure, role permissions, workflow rules, or excluded functionality must be reviewed and approved before implementation.

The V1.1 Decision Record and this PRD are the authoritative product requirements for V1.0.

---

**End of PRD — Version 1.1**
