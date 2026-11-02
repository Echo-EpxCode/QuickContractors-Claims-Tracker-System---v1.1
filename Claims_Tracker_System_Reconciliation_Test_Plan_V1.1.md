# QuickContractors Claims Tracker System

## Reconciliation Verification Test Plan — Version 1.1

**Purpose:** Verify the approved V1.1 documentation baseline before PHP or SQL implementation begins.  
**Controlling authority:** `Claims_Tracker_System_Document_Reconciliation_Decision_Record_V1.1.md`

| ID | Scenario | Expected result |
| --- | --- | --- |
| RB-01 | Inspect the approved model | Exactly the 13 approved tables are specified; no attachment, roles, user_roles, or actions table is specified. |
| RB-02 | Inspect physical fields | `users.email`, `stores.store_code`, and `claims.subject` are nullable; new claims default to Open and Normal. |
| RB-03 | Review due-date handling | `follow_ups.due_date` is a calendar DATE and cannot shift because of timezone conversion. |
| RB-04 | Evaluate date queues | A current Scheduled due date before/today/+1 through +7/+8 days is respectively Overdue/Due Today/Upcoming/outside pending queues. |
| RB-05 | Inspect Overdue persistence | No workflow persists Overdue; a qualifying current row remains Scheduled. |
| RB-06 | Calculate Open Claims | Only Open, In Progress, and Waiting claims are counted, after authorized claim scope is applied. |
| RB-07 | Calculate Last Follow-up Date | The displayed value is the most recent Claim Adjuster customer-contact completion in Asia/Manila; non-contact completion does not replace it. |
| RB-08 | Complete/cancel active follow-up | The transaction requires and saves a replacement dated Scheduled follow-up; no active claim is left without one. |
| RB-09 | Resolve/close a claim | The status transition records its UTC timestamp in claim history; no replacement follow-up is created; closing cancels the current Scheduled follow-up. |
| RB-10 | Create a claim as Claim Adjuster | The creator is the handler and cannot submit another handler. |
| RB-11 | Create/reassign as management | A Supervisor or Director may select any active user as handler. |
| RB-12 | Reassign/take over | `claim_activity_logs` records event code, handler field, old/new handler values, actor, and UTC timestamp; prior adjuster access is removed. |
| RB-13 | Edit Approved guidance | A new revision is created and the current guidance becomes Draft; Claim Adjusters cannot view it. |
| RB-14 | Progress guidance | Only Draft → Approved → Inactive is allowed; Inactive cannot be reactivated. |
| RB-15 | Deactivate an account | Self-deactivation, final-Director deactivation, and deactivation while active claims remain assigned are rejected. |

**Completion condition:** Every scenario is represented in the final requirements-to-test checklist and has a documented acceptance result before implementation moves beyond the baseline gate.

**End of Reconciliation Verification Test Plan — Version 1.1**
