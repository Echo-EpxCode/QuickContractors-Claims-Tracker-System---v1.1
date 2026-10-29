# Claims Tracker System  
## Design Brief Document — Version 1.1

**Document status:** Approved for implementation  
**Version:** 1.1  
**Supersedes:** Design Brief Version 1.0  
**Date:** October 29, 2026  
**Authoritative reconciliation source:** `Claims_Tracker_System_Document_Reconciliation_Decision_Record_V1.1.md`

---

## 1. Overview

### 1.1 Purpose

This Design Brief defines the visual design standards for the Claims Tracker System user interface, including brand identity, color palette, typography, layout, reusable components, dashboard metrics, claims-list structure, filters, follow-up states, and confirmation dialogs.

It governs presentation and interaction only. It does not alter approved permissions, claim lifecycle rules, database structure, or business rules.

### 1.2 Design Objectives

The interface must:

- Make claim information easy to find, read, and understand.
- Highlight overdue follow-ups and claims requiring attention.
- Maintain consistent visual patterns across pages.
- Provide clear feedback for create, update, cancel, close, reopen, reassign, and take-over operations.
- Support Claim Adjuster, Supervisor, and Director roles.
- Adapt to desktop, tablet, and mobile screens.
- Maintain readable content, clear navigation, and accessible controls.
- Avoid implying excluded functionality, especially file uploads and attachments.

### 1.3 Target Users

| Role | Primary interface requirements |
|---|---|
| Claim Adjuster | View and manage only assigned claims; track follow-ups; review action history; view guidance. |
| Supervisor | Monitor all claims; manage Claim Adjuster accounts; reassign or take over claims; create and edit guidance. |
| Director | Review all claims and operational activity; manage all users; approve and deactivate guidance. |

Visual presentation must never be treated as an authorization mechanism.

### 1.4 Core Design Principles

- **Clarity:** Prioritize Job ID, store, claim type, party type, handler, status, priority, and follow-up dates.
- **Consistency:** Reuse approved colors, typography, spacing, and component patterns.
- **Efficiency:** Minimize unnecessary navigation and decoration.
- **Accountability:** Make action outcomes, assignments, and history understandable.
- **Accessibility:** Support readable text, keyboard navigation, visible focus, and sufficient contrast.
- **Responsiveness:** Preserve essential information across supported screen sizes.
- **Scope discipline:** Do not display attachment, upload, email, notification, or import controls.

---

## 2. Brand Identity

### 2.1 Brand Name

Claims Tracker System

### 2.2 Brand Personality

The interface must communicate:

- Professionalism
- Reliability
- Organization
- Efficiency
- Accountability

Use a clean business interface rather than decorative or marketing-oriented styling.

### 2.3 Logo Direction

Use a simple wordmark accompanied by a briefcase, claims-folder, or document-check icon.

Requirements:

- Use a horizontal logo arrangement in the expanded sidebar.
- Use the icon alone when the sidebar is collapsed.
- Maintain sufficient contrast against the charcoal sidebar.
- Avoid elaborate illustrations, gradients, and unnecessary effects.
- Keep branding secondary to operational content.

The icon direction is a recommendation, not a finalized logo asset.

### 2.4 Brand Usage Rules

- Use consistent branding across all pages.
- Reserve primary blue for interactive elements and selected states.
- Use semantic colors according to their defined meanings.
- Apply consistent spacing, borders, typography, and component styling.
- Never rely on color alone to communicate status or urgency.

---

## 3. Color Palette

### 3.1 Core Colors

| Color | Hex | Application |
|---|---|---|
| Sidebar charcoal | `#343A40` | Primary navigation background |
| Content background | `#F4F6F9` | Main application canvas |
| Primary blue | `#0D6EFD` | Primary actions, links, and selected states |
| Surface white | `#FFFFFF` | Cards, tables, forms, and panels |
| Primary text | `#212529` | Main content text |
| Secondary text | `#6C757D` | Supporting text and metadata |
| Border | `#DEE2E6` | Card, input, table, and divider borders |

### 3.2 Semantic Colors

| Purpose | Hex | Application |
|---|---|---|
| Success | `#198754` | Successful operations and completed actions |
| Warning | `#FFC107` | Caution and approaching deadlines |
| Danger | `#DC3545` | Overdue actions and destructive operations |
| Information | `#0DCAF0` | Informational messages |
| Primary | `#0D6EFD` | Primary actions and links |

### 3.3 Claim Status Styling

| Claim status | Visual treatment | Meaning |
|---|---|---|
| Open | Blue | Active claim requiring work |
| In Progress | Blue | Work is underway |
| Waiting | Amber | Waiting on a response, information, or event |
| Resolved | Green | Claim issue has been resolved |
| Closed | Neutral gray | Claim is closed |

### 3.4 Follow-Up Status Styling

| Follow-up status | Visual treatment | Meaning |
|---|---|---|
| Scheduled | Blue | Current follow-up awaiting its due date |
| Overdue | Red | Scheduled follow-up due before the current Philippine date |
| Completed | Green | Follow-up has been completed |
| Cancelled | Neutral gray | Follow-up was cancelled |
| Superseded | Neutral gray | Follow-up was replaced by a newer current follow-up |

The label `Due soon` must not be used. Use `Upcoming`.

### 3.5 Priority Styling

| Priority | Color | Badge treatment |
|---|---|---|
| Non-Urgent | Green | Light green background, dark green text |
| Low | Blue | Light blue background, dark blue text |
| Normal | Yellow | Light yellow background, dark yellow/brown text |
| High | Amber | Light amber background, dark amber text |
| Urgent | Orange | Light orange background, dark orange text |
| Escalated | Red | Light red background, dark red text |

Always display the priority label with the color.

### 3.6 Accessibility

- Pair every status and priority color with a readable text label.
- Use visible borders and focus indicators.
- Verify text contrast against WCAG 2.2 AA requirements.
- Use a darker blue for ordinary text links if default blue does not meet required contrast.
- Do not use color as the sole means of communicating urgency or state.

---

## 4. Typography

### 4.1 Approved Font

Roboto is the sole primary font for V1.0.

Do not mix Open Sans, Lato, or other primary fonts into the interface.

### 4.2 Typography Scale

| Element | Font size | Weight |
|---|---:|---:|
| Page title | 24–28 px | 600–700 |
| Section heading | 18–20 px | 600 |
| Card heading | 16 px | 600 |
| Body text | 14–16 px | 400 |
| Table content | 14 px | 400 |
| Form labels | 14 px | 500–600 |
| Helper text | 12–13 px | 400 |
| Status badges | 12–13 px | 500–600 |

### 4.3 Typography Rules

- Use sentence case for headings and navigation.
- Keep headings concise and descriptive.
- Use consistent date formatting.
- Maintain line height of 1.4–1.6 for body text.
- Avoid excessively small text for Job ID, deadlines, and critical actions.
- Increase text size where needed for mobile readability.

### 4.4 Font Loading

Define the font family centrally:

```css
body {
    font-family: "Roboto", Arial, sans-serif;
}
```

If Roboto is loaded externally, consider privacy, availability, and performance. A locally hosted font is an alternative when reducing third-party dependencies is important.

---

## 5. Application Layout

### 5.1 Layout Areas

| Area | Specification |
|---|---|
| Sidebar | Dark charcoal background, light navigation text, visible active state. |
| Top bar | Page context and permitted user controls. |
| Main content | Light gray canvas containing white cards, tables, and forms. |
| Dashboard | High-priority metrics and claims requiring attention. |
| Responsive layout | Collapsible sidebar and suitable table behavior on narrow screens. |

### 5.2 Sidebar Menu

The sidebar must include:

1. Dashboard
2. Claims
3. Follow-ups
4. Action History
5. Claim Guidance
6. Users
7. Stores
8. Service Calls

### 5.3 Role-Based Menu Visibility

| Menu | Claim Adjuster | Supervisor | Director |
|---|---:|---:|---:|
| Dashboard | Yes | Yes | Yes |
| Claims | Yes | Yes | Yes |
| Follow-ups | Yes | Yes | Yes |
| Action History | Yes | Yes | Yes |
| Claim Guidance | Yes | Yes | Yes |
| Users | No | Yes | Yes |
| Stores | Yes — view only | Yes | Yes |
| Service Calls | No | Yes | Yes |

The current page must have a distinct active state. Navigation must adapt to smaller screens and reflect user permissions.

### 5.4 Top Bar

The top bar may contain:

- Page title or breadcrumb.
- User identity.
- Account menu.
- Logout action.
- Other approved utilities.

Do not introduce notifications or other functionality unless the system actually supports them.

---

## 6. Dashboard Design

### 6.1 Required Metrics

The dashboard must display:

1. Open Claims
2. Overdue Follow-ups
3. Due Today
4. Upcoming Follow-ups
5. Claims Requiring Attention

### 6.2 Claims Requiring Attention

A claim belongs in **Claims Requiring Attention** when either condition is true:

- The claim has an overdue follow-up.
- The claim priority is `Urgent` or `Escalated`.

### 6.3 Metric Card Requirements

Each metric card must include:

- A descriptive label.
- An accurate count.
- Optional supporting icon.
- A link or navigation path where useful.

Counts must be calculated from authoritative database records and must respect the signed-in user’s authorized claim scope.

---

## 7. Claims List Design

### 7.1 Required Columns

The claims list must display:

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

### 7.2 Required Filters

The claims list must support:

- Job ID search.
- Claim status filter.
- Priority filter.
- Assigned-handler filter, limited to Claim Adjusters.
- Follow-up state filter: `Overdue`, `Due Today`, and `Upcoming`.
- Store filter.
- Party type filter.
- Date-range filter based on follow-up due date.

### 7.3 Pagination

- Claims lists use 25 records per page.
- Provide clear pagination controls.
- Preserve active filters during pagination where practical.
- Provide a clear method to clear filters.

### 7.4 Date Display

- Dates are stored in UTC.
- Dates display as `MM/DD/YYYY`.
- Timestamps display as `MM/DD/YYYY hh:mm AM/PM` in `Asia/Manila` time.

### 7.5 Responsive Table Behavior

On narrow screens:

- Preserve critical information: Job ID, handler, status, priority, and follow-up date.
- Use a responsive table, horizontal scroll, or compact record view.
- Do not silently hide critical workflow information.

---

## 8. UI Components

### 8.1 Component Standards

Reusable components must define, where applicable:

- Default state
- Hover state
- Focus state
- Active or selected state
- Disabled state
- Loading state
- Success and error feedback

Disabled appearance must correspond to actual behavior. Server-side authorization and validation remain mandatory.

### 8.2 Buttons

| Variant | Purpose | Treatment |
|---|---|---|
| Primary | Main action | Blue background with readable white text |
| Secondary | Alternative or cancel action | Neutral gray or outlined treatment |
| Danger | Destructive action | Red treatment and appropriate confirmation |
| Outline | Secondary navigation or viewing | Clear border and readable label |
| Icon | Compact utility action | Accessible name or tooltip |
| Disabled | Temporarily unavailable action | Clearly disabled appearance and behavior |

Buttons must prevent accidental duplicate submissions, display loading feedback where appropriate, and report operation results.

### 8.3 Forms and Inputs

Supported input types:

- Text inputs
- Textareas for notes
- Select dropdowns
- Date inputs
- Checkboxes and radio controls where appropriate

Form requirements:

- Provide visible labels.
- Clearly identify required fields.
- Display validation errors near the relevant field.
- Preserve entered values when validation fails where practical.
- Validate all requests on the server.
- Use consistent date formats and clear instructions.
- Do not include file-input controls because file uploads are excluded from V1.0.

### 8.4 Status Badges

| State | Styling |
|---|---|
| Overdue | Light red background, dark red text |
| Scheduled | Light blue background, dark blue text |
| Upcoming | Light blue background, dark blue text |
| Completed | Light green background, dark green text |
| Cancelled | Light gray background, dark gray text |
| Superseded | Light gray background, dark gray text |

Badges must use consistent meanings throughout the application. Status values must come from approved application logic, not arbitrary frontend conditions.

### 8.5 Alerts and Feedback

| Type | Purpose |
|---|---|
| Success | Confirms an operation completed |
| Warning | Highlights a condition requiring attention |
| Error | Explains an unsuccessful operation |
| Information | Provides context or instructions |

Example messages:

- “Claim updated successfully.”
- “The follow-up date is required.”
- “This record could not be saved. Please try again.”
- “You do not have permission to perform this action.”

Messages must not expose SQL errors, stack traces, server paths, or sensitive technical details.

### 8.6 Required Confirmation Dialogs

SweetAlert2 confirmation dialogs are required for:

- Take over claim.
- Reassign claim.
- Close claim.
- Reopen claim.
- Cancel follow-up.
- Deactivate user.
- Deactivate store.
- Deactivate guidance.

#### Take-Over Dialog Requirements

Before take-over is saved, the dialog must:

- Identify the current handler, if any.
- State that the current handler will be replaced.
- Explain that taking over a Closed claim will reopen it.
- Explain that a new follow-up will be required for a reopened closed claim.
- Provide clear confirm and cancel actions.

### 8.7 Empty States

Use clear empty states such as:

- “No claims found.”
- “No follow-ups are currently overdue.”
- “No action history is available for this claim.”
- “No guidance revisions are available.”

Database failures must display error states, not misleading empty states.

### 8.8 Loading States

- Use a spinner for short operations.
- Use skeleton layouts for larger content areas.
- Prevent duplicate submissions during processing.
- Show an error if data cannot be loaded.
- Avoid leaving the user with a blank page or panel.

### 8.9 Shared Spacing and Dimensions

| Property | Recommended value |
|---|---:|
| Base spacing unit | 4 px |
| Small gap | 8 px |
| Standard component gap | 16 px |
| Major section gap | 24 px |
| Card padding | 16–24 px |
| Standard input height | 38–44 px |
| Button height | 38–44 px |
| Border radius | 4–8 px |
| Border width | 1 px |

These values may be adjusted to preserve readability, responsiveness, and Bootstrap compatibility.

---

## 9. Implementation Alignment

| Layer | Responsibility |
|---|---|
| HTML and PHP templates | Render page structure and reusable components |
| CSS and Bootstrap | Apply colors, typography, spacing, and responsive layout |
| JavaScript | Provide permitted client-side interactions and feedback |
| PHP server logic | Validate requests, enforce authorization, and execute business operations |
| MySQL database | Store authoritative claim, follow-up, and history records |

All security-sensitive operations must be enforced on the server. UI components do not replace authentication, authorization, validation, transaction handling, or audit controls.

---

**End of Design Brief Document — Version 1.1**
