# QuickContractors Claims Tracker System

## Document Reconciliation Decision Record — Version 1.1

**Status:** Approved implementation baseline  
**Version:** 1.1  
**Date:** November 2, 2026  
**Applies to:** PRD, TRD, App Flow, Design Brief, Background Schema, Implementation Plan, and Project Instructions V1.1

**Approval evidence:** Project-owner approvals recorded in the documentation reconciliation review on November 2, 2026.

---

## 1. Authority and scope

This record is the controlling reconciliation authority for the QuickContractors Claims Tracker System V1.0 implementation baseline. Where an affected V1.1 document conflicts with this record, this record governs. It resolves documentation gaps only; it does not add product scope, tables, attachment functionality, file uploads, frameworks, or unapproved dependencies.

The approved database baseline is the existing 13-table model. No additional table is authorized:
  
1. `users`
2. `stores`
3. `claims`
4. `claim_types`
5. `claim_statuses`
6. `claim_parties`
7. `service_calls`
8. `follow_ups`
9. `action_history`
10. `claim_activity_logs`
11. `claim_history`
12. `claim_guidance`
13. `claim_guidance_revisions`

## 2. Approved schema decisions

- `follow_ups.due_date` is a required calendar `DATE`. It is not a timestamp and must not be converted across time zones.
- Event timestamps, including `created_at`, `updated_at`, `completed_at`, and status-change timestamps, are stored in UTC and displayed in `Asia/Manila`.
- `users.email`, `stores.store_code`, and `claims.subject` are optional nullable fields. Their absence must not prevent a valid record from being created.
- New claims default to status `Open` and priority `Normal`.
- Preserve records through deactivation and restricted deletion rules. Foreign keys for operational and historical relationships use `RESTRICT` (or an equivalent safe behavior) on delete; no history or revision data may cascade-delete.
- Assignment history is stored in `claim_activity_logs`: `activity_type` is `claim_reassigned` or `claim_taken_over`; `field_name` is `assigned_handler_id`; `old_value` and `new_value` contain the previous and new handler identifiers; `actor_user_id` identifies the acting user; and `created_at` records the UTC event time.
- A status transition that reaches `Resolved` or `Closed` must be recorded in `claim_history` with the UTC transition timestamp. That timestamp is the resolved or closed date displayed by the system.

## 3. Claims, follow-ups, and dashboard decisions

- A claim is active only when its status is `Open`, `In Progress`, or `Waiting`. **Open Claims** counts those active claims, subject to the signed-in user’s authorized claim scope.
- `Overdue` is a derived display and queue state, not a persisted follow-up status. A current follow-up remains stored as `Scheduled` when its date is before the current `Asia/Manila` date.
- Pending queues consider only current `Scheduled` follow-ups: before today is Overdue; today is Due Today; tomorrow through seven days ahead inclusive is Upcoming.
- **Last Follow-up Date** is the `completed_at` date, displayed in `Asia/Manila`, of the most recently completed customer contact by the Claim Adjuster. A completion must include a customer-contact outcome; follow-ups completed without customer contact do not update this display value.
- Completing or cancelling a current follow-up for an active claim requires a replacement `Scheduled` follow-up with a dated `due_date` in the same transaction.
- Resolving or closing a claim does not create a replacement follow-up. Closing cancels the current scheduled follow-up; resolving leaves no pending follow-up once the status transition is committed.

## 4. Assignment and account decisions

- A Claim Adjuster creating a claim is assigned to that claim automatically and cannot choose another handler.
- A Supervisor or Director creating or reassigning a claim may select any active user as handler.
- Before user deactivation, every active claim assigned to the target user must be reassigned to an active user.
- A user cannot deactivate their own account.
- The last active Director cannot be deactivated.

## 5. Guidance lifecycle decisions

- Claim Adjusters, Supervisors, and Directors may view only `Approved` guidance in normal guidance views. Draft and Inactive guidance are not visible to Claim Adjusters.
- Supervisors may create and edit Draft guidance. Directors may create, edit, approve, and deactivate guidance.
- Editing Approved guidance creates a revision and returns the current guidance record to `Draft`.
- The only status path is `Draft` to `Approved` to `Inactive`. Inactive guidance is terminal and cannot be reactivated in V1.0.
- Every content edit and status transition creates an append-only revision in the same transaction.

## 6. Baseline verification

Before PHP or SQL implementation begins, confirm that every affected document incorporates these decisions and that the implementation checklist includes tests for queues, active-claim follow-up replacement, status-date recording, authorization, guidance lifecycle, assignment history, and account deactivation safeguards.

**End of Document Reconciliation Decision Record — Version 1.1**
