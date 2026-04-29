# Document Management System (DMS) — Frontend Vue + Blade Codex Prompts

## 1. Purpose

This document contains ready-to-use Codex prompts for implementing the DMS frontend using Laravel Blade + Vue 3 hybrid approach.

The goal is to build a professional, enterprise-style UI while keeping the frontend clean, modular, permission-aware, and easy to maintain.

Codex should use this document with:

```text
12-api-route-design
13-ui-ux-screen-flow-design
15-implementation-rules-and-coding-standard
18-controller-request-policy-codex-prompts
```

---

## 2. Frontend Strategy

Use:

```text
Blade + Vue 3 Hybrid
```

Blade responsibility:

```text
- Admin layout
- Sidebar
- Page shell
- Breadcrumb
- Permission-based menus
- Basic CRUD pages
```

Vue responsibility:

```text
- Dynamic document upload form
- Dynamic metadata fields
- Advanced document search/filter
- Approval flow step builder
- Document details interactive sections
- Timeline components
- Version upload modal
```

---

## 3. Global Frontend Rules

Codex must follow these rules:

```text
1. Use Blade for layout and page shell.
2. Use Vue 3 for complex/dynamic UI only.
3. Do not expose private file paths.
4. Hide unavailable buttons based on backend-provided permissions.
5. Backend permission checks remain mandatory.
6. Use reusable Vue components.
7. Use clean loading/error/empty states.
8. Keep forms user-friendly for non-technical users.
9. Keep UI desktop-first but responsive.
10. Do not hardcode approval steps in frontend.
```

---

## 4. Recommended Vue Structure

```text
resources/js/dms/
 ├── app.js
 ├── api/
 │    ├── http.js
 │    ├── documentsApi.js
 │    ├── metadataApi.js
 │    ├── approvalApi.js
 │    └── lookupApi.js
 │
 ├── components/
 │    ├── common/
 │    │    ├── StatusBadge.vue
 │    │    ├── LoadingSpinner.vue
 │    │    ├── EmptyState.vue
 │    │    ├── ConfirmModal.vue
 │    │    └── Pagination.vue
 │    │
 │    ├── documents/
 │    │    ├── DocumentUploadForm.vue
 │    │    ├── DynamicMetadataFields.vue
 │    │    ├── DocumentSearchFilters.vue
 │    │    ├── DocumentSearchResults.vue
 │    │    ├── DocumentActionButtons.vue
 │    │    ├── DocumentVersionHistory.vue
 │    │    └── VersionUploadModal.vue
 │    │
 │    ├── approval/
 │    │    ├── ApprovalStepBuilder.vue
 │    │    ├── ApprovalTimeline.vue
 │    │    ├── ApprovalActionModal.vue
 │    │    └── PendingApprovalTable.vue
 │    │
 │    └── audit/
 │         └── ActivityTimeline.vue
 │
 └── pages/
      ├── DocumentCreatePage.vue
      ├── DocumentEditPage.vue
      ├── DocumentSearchPage.vue
      ├── DocumentShowPage.vue
      ├── ApprovalFlowEditPage.vue
      └── PendingApprovalPage.vue
```

---

# 5. Prompt 01 — Admin Layout & Sidebar

```text
Read 13-ui-ux-screen-flow-design and 15-implementation-rules-and-coding-standard.

Implement the DMS admin layout using Blade.

Requirements:
- Topbar
- Sidebar
- Content area
- Breadcrumb section
- Page title section
- Flash success/error messages
- Permission-aware sidebar menu

Sidebar items:
- Dashboard
- Documents
  - All Documents
  - Upload Document
  - Pending Approval
  - Archived Documents
- Master Setup
  - Companies
  - Departments
  - Document Categories
  - Metadata Fields
- Approval Setup
  - Approval Flows
- Access Control
  - User Company Access
  - User Department Access
- Reports / Logs
  - Audit Logs
- Settings

Rules:
- Use Laravel permission checks to show/hide menus.
- Do not rely only on frontend permission hiding.
- Keep layout reusable.
```

---

# 6. Prompt 02 — Basic CRUD Blade Pages

```text
Read 13-ui-ux-screen-flow-design and 18-controller-request-policy-codex-prompts.

Create Blade CRUD pages for:
- Companies
- Departments
- Document Categories
- Metadata Fields

Requirements:
- List page with table and filters where needed
- Create form
- Edit form
- Delete/deactivate confirmation
- Status badge
- Success/error flash messages

Rules:
- Use route names from 12-api-route-design.
- Do not put business logic in Blade.
- Use backend validation errors.
- Keep UI consistent across CRUD pages.
```

---

# 7. Prompt 03 — Vue App Bootstrap

```text
Set up Vue 3 entry for DMS pages.

Requirements:
- Create resources/js/dms/app.js
- Mount Vue components only if matching root element exists
- Support multiple independent page mounts
- Register common components if needed
- Configure axios/fetch wrapper with CSRF token
- Use Laravel Vite

Mount examples:
- #dms-document-create-app → DocumentCreatePage
- #dms-document-edit-app → DocumentEditPage
- #dms-document-search-app → DocumentSearchPage
- #dms-document-show-app → DocumentShowPage
- #dms-approval-flow-edit-app → ApprovalFlowEditPage

Rules:
- Do not create SPA router.
- This is Blade + Vue hybrid.
- Props should come from Blade data attributes or JSON script safely.
```

---

# 8. Prompt 04 — Common Vue Components

```text
Read 13-ui-ux-screen-flow-design.

Create reusable Vue 3 common components:

1. StatusBadge.vue
- Props: status, type optional
- Support document statuses: draft, pending_approval, approved, rejected, archived
- Support expiry badges: expired, expiring_soon

2. LoadingSpinner.vue
- Simple loading indicator

3. EmptyState.vue
- Props: title, message, actionLabel, actionUrl optional

4. ConfirmModal.vue
- Props: title, message, confirmText, cancelText, danger
- Emits confirm/cancel

5. Pagination.vue
- Props: meta
- Emits page-change

Rules:
- Keep components generic.
- No business logic inside common components.
```

---

# 9. Prompt 05 — Document Upload Form

```text
Read 13-ui-ux-screen-flow-design, 08-file-storage-versioning-design, 06-approval-workflow-design, and 18-controller-request-policy-codex-prompts.

Create Vue components for document upload:

- DocumentCreatePage.vue
- DocumentUploadForm.vue
- DynamicMetadataFields.vue

Form sections:
1. Basic Information
2. Classification
3. Dynamic Metadata
4. File Upload
5. Tags & Remarks
6. Submit Actions

Fields:
- title
- document_no
- document_date
- expiry_date
- confidentiality_level
- summary
- company_id
- department_id
- category_id
- metadata dynamic fields
- file
- tags
- remarks

Dynamic behavior:
- Select company → load departments
- Select department → load categories
- Select category → load metadata fields
- If category requires approval → show Save as Draft and Submit for Approval
- If category does not require approval → show Save Document

Rules:
- Do not hardcode metadata fields.
- Do not hardcode approval steps.
- Show allowed file types and max file size from backend props/settings.
- Show validation errors from backend.
- Submit using normal form or axios, but keep UX clean.
- Do not expose file path.
```

---

# 10. Prompt 06 — Dynamic Metadata Fields Component

```text
Read 04-database-design, 13-ui-ux-screen-flow-design, and 17-service-repository-codex-prompts.

Implement DynamicMetadataFields.vue.

Props:
- fields
- modelValue
- errors

Supported field types:
- text
- textarea
- number
- date
- select
- multi_select
- checkbox
- boolean

Behavior:
- Render input based on field_type.
- Show required indicator.
- Show validation errors.
- Emit updated metadata object.
- Use field_key as metadata key.

Rules:
- No hardcoded category fields.
- Options should come from metadata field options JSON.
- Keep component reusable for create, edit, and advanced search.
```

---

# 11. Prompt 07 — Document Search Page

```text
Read 07-search-architecture-design, 13-ui-ux-screen-flow-design, 05-permission-access-control-design, and 18-controller-request-policy-codex-prompts.

Create Vue components:
- DocumentSearchPage.vue
- DocumentSearchFilters.vue
- DocumentSearchResults.vue
- DocumentActionButtons.vue

Features:
- Global keyword search
- Company filter
- Department filter
- Category filter
- Status filter
- Confidentiality filter
- Document date range
- Expiry date range
- Uploaded by filter
- Tags filter
- Advanced metadata filters
- Reset filters
- Pagination

Rules:
- Use /admin/dms/api/documents/search endpoint.
- Search results must come from backend only.
- Backend handles access control.
- UI should hide actions based on allowed actions returned by backend.
- Do not expose private file paths.
- Show empty state when no documents found.
```

---

# 12. Prompt 08 — Document Action Buttons

```text
Read 05-permission-access-control-design and 13-ui-ux-screen-flow-design.

Create DocumentActionButtons.vue.

Props:
- document
- permissions or allowed_actions

Actions:
- View
- Preview
- Download
- Edit
- Upload New Version
- Submit for Approval
- Approve
- Reject
- Archive
- Delete

Rules:
- Only show buttons if allowed_actions says true.
- Do not calculate sensitive permissions only in frontend.
- Use backend route URLs from document payload or props.
- Use ConfirmModal for destructive actions.
- Backend must still enforce authorization.
```

---

# 13. Prompt 09 — Document Details Page

```text
Read 13-ui-ux-screen-flow-design, 08-file-storage-versioning-design, 09-audit-log-activity-design, and 06-approval-workflow-design.

Create DocumentShowPage.vue and related components:
- DocumentVersionHistory.vue
- ApprovalTimeline.vue
- ActivityTimeline.vue
- VersionUploadModal.vue
- DocumentActionButtons.vue

Page sections:
1. Header summary
2. Document information
3. Custom metadata
4. File preview area
5. Version history
6. Approval timeline
7. Activity timeline
8. Permission panel if authorized

Rules:
- Show status/confidentiality badges.
- Show preview only through secure backend preview route.
- Download only through secure backend download route.
- Do not expose storage paths.
- Version upload creates new version, not overwrite.
- Approval timeline should read dynamic approval data.
```

---

# 14. Prompt 10 — Version Upload Modal

```text
Read 08-file-storage-versioning-design and 13-ui-ux-screen-flow-design.

Create VersionUploadModal.vue.

Fields:
- file
- note/remarks

Behavior:
- Uploads to POST /admin/dms/documents/{document}/versions
- Shows selected file name
- Shows allowed file types and max file size
- Shows backend validation errors
- Shows confirmation that new version will be created

Rules:
- Never overwrite approved document.
- Backend handles versioning and approval reset.
- Do not expose file path.
```

---

# 15. Prompt 11 — Approval Flow Builder

```text
Read the latest 06-approval-workflow-design and 13-ui-ux-screen-flow-design.

Create ApprovalFlowEditPage.vue and ApprovalStepBuilder.vue.

Approval flow fields:
- name
- company_id optional
- department_id optional
- category_id required
- is_default
- auto_approve_if_no_flow
- status

Approval step fields:
- step_order
- step_name
- approver_type user/role
- approver_user_id
- approver_role_id
- is_required
- can_edit_document
- can_download_document
- requires_comment_on_reject

Features:
- Add step
- Remove step
- Reorder steps
- Change approver type dynamically
- Save flow

Critical rules:
- Approval is 100% dynamic and DB-driven.
- Do not hardcode number of steps.
- Step count can be 1 to N.
- Approver can be user or role.
- UI must allow changing steps without code changes.
```

---

# 16. Prompt 12 — Pending Approval Page

```text
Read 06-approval-workflow-design, 13-ui-ux-screen-flow-design, and 18-controller-request-policy-codex-prompts.

Create PendingApprovalPage.vue and PendingApprovalTable.vue.

Features:
- List documents assigned to current user or user's roles
- Columns: title, document_no, company, department, category, current_step, submitted_by, submitted_at, actions
- Actions: view, approve, reject
- ApprovalActionModal for approve/reject

Rules:
- Approval assignment comes from backend.
- Do not calculate approver eligibility in frontend.
- Reject comment required if backend/current step requires it.
- Backend must enforce current step approval.
```

---

# 17. Prompt 13 — Approval Timeline Component

```text
Read 06-approval-workflow-design and 13-ui-ux-screen-flow-design.

Create ApprovalTimeline.vue.

Props:
- approvals

Display:
- Step order
- Step name
- Assigned user/role
- Status
- Action by
- Action time
- Comment

States:
- approved
- rejected
- pending
- waiting
- skipped

Rules:
- Timeline must support dynamic number of steps.
- Do not assume exactly 3 steps.
- Show current active step clearly.
```

---

# 18. Prompt 14 — Activity Timeline Component

```text
Read 09-audit-log-activity-design and 13-ui-ux-screen-flow-design.

Create ActivityTimeline.vue.

Props:
- activities

Display:
- Date/time
- User
- Human-readable event message
- Optional metadata/details

Rules:
- Do not expose sensitive old/new values unless backend provides them.
- Timeline should be clean and readable.
- Support empty state.
```

---

# 19. Prompt 15 — Settings UI

```text
Read 11-system-settings-configuration-design and 13-ui-ux-screen-flow-design.

Create settings Blade/Vue UI.

Groups/tabs:
- General
- File Upload
- Security
- Approval
- OCR
- Search
- Notification
- Retention
- Backup

Rules:
- Only editable settings should be editable.
- Private settings should not be exposed if backend marks is_public false unless user is authorized.
- Validate before saving.
- Show warning for risky settings like auto approval and physical file deletion.
- Use SettingsService backend.
```

---

# 20. Prompt 16 — Audit Log UI

```text
Read 09-audit-log-activity-design and 13-ui-ux-screen-flow-design.

Create Audit Log UI.

Features:
- Filter by event
- Filter by user
- Filter by date range
- Filter by auditable type
- Table columns: date/time, user, event, target, IP address, details

Rules:
- Use backend access control.
- Do not expose sensitive details to unauthorized users.
- No edit/delete actions for audit logs.
```

---

# 21. Prompt 17 — OCR UI Later

```text
Read 10-ocr-processing-design and 13-ui-ux-screen-flow-design.

Create OCR UI later only.

Features:
- Show OCR status on document details page
- Show processed_at
- Show error message if failed
- Reprocess OCR button if authorized
- Optional OCR text viewer if authorized

Rules:
- Do not expose OCR text without backend permission.
- OCR reprocess must call backend route and queue job.
- Show warning that OCR text may contain errors.
```

---

# 22. Prompt 18 — Frontend Review Prompt

Use after Codex creates frontend pages/components.

```text
Review DMS frontend against:
- 13-ui-ux-screen-flow-design
- 05-permission-access-control-design
- 06-approval-workflow-design
- 08-file-storage-versioning-design
- 15-implementation-rules-and-coding-standard

Check:
1. Blade handles layout/page shell.
2. Vue handles dynamic parts only.
3. No private file paths are exposed.
4. Buttons are permission-aware.
5. Backend authorization is still used.
6. Dynamic metadata is not hardcoded.
7. Approval step builder supports 1 to N steps.
8. Approval UI is fully dynamic.
9. Search page uses backend results only.
10. Empty/loading/error states exist.
11. UI is professional and usable.

Return a checklist and apply fixes if needed.
```

---

# 23. Recommended Frontend Implementation Order

```text
1. Admin layout and sidebar
2. Basic CRUD Blade pages
3. Vue app bootstrap
4. Common Vue components
5. Dynamic metadata fields
6. Document upload page
7. Document search page
8. Document details page
9. Version upload modal
10. Approval flow builder
11. Pending approval page
12. Approval timeline
13. Activity timeline
14. Audit log UI
15. Settings UI
16. OCR UI later
```

---

## 24. Summary

This document gives Codex frontend prompts for building the DMS using Blade + Vue 3 hybrid architecture.

The most important rule is:

```text
Frontend should improve usability, but backend remains the source of truth for permission, workflow, file access, and security.
```

