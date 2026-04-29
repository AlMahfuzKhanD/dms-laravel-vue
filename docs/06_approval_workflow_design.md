# Document Management System (DMS) — Approval Workflow Design (Updated: Fully Dynamic)

## 1. Purpose

This document defines a **fully dynamic, configurable approval workflow system** for the DMS.

Goal:

- Enable/disable approval per document category
- Configure any number of steps (1…N)
- Configure approver by **user OR role**
- Change flows without code change
- Support company/department/category level overrides

This design is **100% configuration-driven (DB-driven)**.

---

## 2. Core Principle (Very Important)

```text
NO approval logic is hardcoded.
Everything is controlled from database.
```

---

## 3. Approval Control Levels

Approval behavior is controlled at **three levels**:

```text
1. Category Level → whether approval is required
2. Flow Level → which workflow applies
3. Step Level → how many steps + who approves
```

---

## 4. Category-Level Control

From: `document_categories`

```text
requires_approval BOOLEAN
```

### Behavior

```text
FALSE → No approval needed
TRUE  → Must go through approval workflow
```

### Example

```text
HR Circular → no approval
Land Document → approval required
```

---

## 5. Approval Flow Configuration

Table: `approval_flows`

### New Required Fields (IMPORTANT)

```text
is_default BOOLEAN DEFAULT FALSE
auto_approve_if_no_flow BOOLEAN DEFAULT TRUE
```

### Meaning

```text
is_default → fallback flow if no specific match found

auto_approve_if_no_flow:
TRUE  → auto approve if no flow found
FALSE → block submission (must configure flow)
```

---

## 6. Flow Matching Logic (Dynamic Selection)

When user submits document → system finds flow in this priority:

```text
1. company_id + department_id + category_id
2. company_id + category_id
3. category_id only
4. default flow (is_default = true)
```

If still not found:

```text
IF auto_approve_if_no_flow = TRUE → auto approve
IF FALSE → throw error: "Approval flow not configured"
```

---

## 7. Dynamic Step Configuration

Table: `approval_steps`

### You can:

```text
✔ Add unlimited steps
✔ Remove steps anytime
✔ Reorder steps
✔ Assign user OR role
✔ Make step optional
```

### New Fields (IMPORTANT)

```text
can_edit_document BOOLEAN DEFAULT FALSE
can_download_document BOOLEAN DEFAULT FALSE
requires_comment_on_reject BOOLEAN DEFAULT TRUE
```

### Example 1 — 1 Step Flow

```text
Step 1: Department Head
```

### Example 2 — 3 Step Flow

```text
Step 1: Legal Officer
Step 2: Legal Manager
Step 3: Director
```

### Example 3 — 5 Step Flow

```text
Step 1: Officer
Step 2: Manager
Step 3: Senior Manager
Step 4: CFO
Step 5: MD
```

👉 No code change needed. Only DB update.

---

## 8. Step Assignment Logic

Each step can be:

```text
approver_type = user OR role
```

### Case 1 — User Based

```text
Only specific user can approve
```

### Case 2 — Role Based

```text
All users with that role can see task
Any one can approve
```

---

## 9. Approval Lifecycle (Updated)

## 9.1 Draft

```text
status = draft
```

Editable.

---

## 9.2 Submit for Approval

```text
status = pending_approval
```

System:

```text
1. Resolve flow
2. Create document_approvals rows
3. Activate first step
```

---

## 9.3 Step Execution

```text
Only ONE step active at a time
```

When approved:

```text
→ mark step approved
→ activate next step
```

---

## 9.4 Final Approval

```text
status = approved
```

---

## 9.5 Rejection

```text
status = rejected
```

Behavior:

```text
- Must save comment if requires_comment_on_reject = TRUE
- Sent back to uploader
- Editable again
```

---

## 10. Re-Submission (Important)

```text
Rejected → Edit → Submit again
```

System:

```text
Creates NEW approval cycle
Old history remains
```

---

## 11. Version Control + Approval

Rule:

```text
New version → new approval required (if category requires)
```

---

## 12. Dynamic Control Summary

You can control EVERYTHING from DB:

```text
✔ Enable/disable approval per category
✔ Define flow per company
✔ Define flow per department
✔ Define flow per category
✔ Add/remove steps
✔ Change approver anytime
✔ Use role/user-based approval
✔ Control reject behavior
✔ Control fallback behavior
```

---

## 13. Edge Case Handling (Updated)

### Case: No Flow Found

```text
IF auto_approve_if_no_flow = TRUE
→ auto approve

IF FALSE
→ block submission
```

---

### Case: Single Step

```text
Approve → instantly approved
```

---

### Case: Optional Step

```text
is_required = FALSE
→ can skip
```

---

## 14. Services (Updated)

Create:

```text
ApprovalFlowResolverService
DocumentApprovalService
ApprovalActionService
```

### Key Methods

```text
resolveFlow(document)
createApprovalSteps(document, flow)
approveStep(document, user)
rejectStep(document, user, comment)
moveToNextStep(document)
finalizeApproval(document)
```

---

## 15. Codex Rules (Critical)

1. Never hardcode approval logic
2. Always resolve flow dynamically
3. Steps must come from DB
4. Only one active step
5. Preserve history always
6. Use DB transaction for approval
7. Do not mix approval logic in controller

---

## 16. Final Confirmation (Answer to Your Question)

YES — This design is now **100% dynamic**:

```text
✔ You can decide which category needs approval
✔ You can define 0, 1, or N steps
✔ You can change steps anytime
✔ You can assign user or role
✔ You can control fallback behavior
✔ No code change required for workflow changes
```

---

## 17. Summary

Approval workflow is now fully configurable, scalable, and enterprise-ready.

This ensures your system can adapt to real business needs without redeployment or code changes.

