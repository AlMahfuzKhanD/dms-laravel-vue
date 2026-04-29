# Document Management System (DMS) — Permission & Access Control Design

## 1. Purpose

This document defines the permission and access control design for the Document Management System.

The DMS will store sensitive company documents such as legal papers, land documents, licenses, agreements, HR files, finance documents, and administrative records. Therefore, access control must be designed carefully from the beginning.

This document is written to guide implementation using Laravel, Spatie Laravel Permission, policies, query scopes, and custom document access rules.

---

## 2. Access Control Goal

The goal is to ensure that every user can only perform actions they are authorized to perform.

A user should only access documents based on:

```text
1. Role permission
2. Company access
3. Department access
4. Category access
5. Document confidentiality level
6. Document-level override rules
7. Approval responsibility
```

Access control must be applied in:

```text
- Document listing
- Document search
- Document preview
- Document download
- Document edit
- Document delete/archive
- Approval actions
- Permission management
```

---

## 3. Access Control Layers

The system should use layered access control.

```text
Layer 1: Authentication
Layer 2: Global Role Permission
Layer 3: Company Access
Layer 4: Department Access
Layer 5: Category Access
Layer 6: Document Confidentiality
Layer 7: Document-Level Override
Layer 8: Approval Assignment
```

A user must pass all required layers to access a document.

---

## 4. Role-Based Permission

Use Spatie Laravel Permission for general role and permission management.

### Example Roles

```text
super_admin
company_admin
department_admin
document_uploader
document_approver
document_viewer
```

### Example Permissions

```text
company.view
company.create
company.edit
company.delete

department.view
department.create
department.edit
department.delete

document_category.view
document_category.create
document_category.edit
document_category.delete

document.view
document.upload
document.edit
document.delete
document.archive
document.download
document.preview
document.submit_for_approval
document.approve
document.reject
document.manage_permissions

approval_flow.view
approval_flow.create
approval_flow.edit
approval_flow.delete

audit_log.view
user.manage
role.manage
permission.manage
```

---

## 5. Role Responsibility Matrix

## 5.1 Super Admin

Super Admin has full system access.

Allowed actions:

```text
- Manage all companies
- Manage all departments
- Manage all users
- Manage roles and permissions
- Manage all document categories
- View all documents
- Download all documents
- Manage approval flows
- View audit logs
- Override document permissions
```

Super Admin should bypass company and department restrictions.

---

## 5.2 Company Admin

Company Admin manages documents and users within assigned company/companies.

Allowed actions:

```text
- View assigned companies
- Manage departments under assigned companies
- Manage users under assigned companies if allowed
- View documents under assigned companies
- Upload documents under assigned companies
- Edit documents under assigned companies if permitted
- Manage approval flows for assigned companies if permitted
- View audit logs for assigned companies if permitted
```

Company Admin cannot access documents from unassigned companies.

---

## 5.3 Department Admin

Department Admin manages documents inside assigned department/departments.

Allowed actions:

```text
- View assigned departments
- View documents under assigned departments
- Upload documents under assigned departments
- Edit documents under assigned departments if permitted
- Submit documents for approval
- View department-level audit logs if permitted
```

Department Admin cannot access documents from unassigned departments.

---

## 5.4 Document Uploader

Document Uploader can upload and manage own submitted documents depending on status.

Allowed actions:

```text
- Upload documents in assigned company/department
- Edit draft documents created by self
- Submit document for approval
- View own uploaded documents
- View approval status of own documents
```

Uploader should not approve own document unless explicitly configured.

---

## 5.5 Document Approver

Document Approver reviews and approves/rejects assigned documents.

Allowed actions:

```text
- View documents assigned for approval
- Preview documents assigned for approval
- Approve assigned approval step
- Reject assigned approval step
- Add approval comments
```

Approver should not automatically see all documents unless other permissions allow it.

---

## 5.6 Document Viewer

Document Viewer can search and view/download documents based on assigned company, department, category, and document confidentiality.

Allowed actions:

```text
- Search allowed documents
- Preview allowed documents
- Download allowed documents if permission exists
```

---

## 6. Company-Level Access

Company-level access is controlled by `user_company_accesses` table.

### Table

```text
user_company_accesses
- id
- user_id
- company_id
- created_by
- created_at
- updated_at
```

### Rule

A user can access documents of a company only if:

```text
- User is super_admin
OR
- User has company access in user_company_accesses
```

### Example

```text
User A has access to Company 1 and Company 3.
User A cannot see documents from Company 2.
```

---

## 7. Department-Level Access

Department-level access is controlled by `user_department_accesses` table.

### Table

```text
user_department_accesses
- id
- user_id
- department_id
- created_by
- created_at
- updated_at
```

### Rule

A user can access documents of a department only if:

```text
- User is super_admin
OR
- User has access to the document's department
OR
- User has company-level admin permission and the department belongs to assigned company
```

---

## 8. Category-Level Access

For the initial version, category-level access can be handled through permissions and company/department access.

For future strict control, a dedicated table can be added:

```text
user_category_accesses
- id
- user_id
- category_id
- access_type
- created_by
- created_at
- updated_at
```

Initial recommendation:

Do not add category-level user access in MVP unless required. Start with company + department + role permission. Use document-level override for special cases.

---

## 9. Document Confidentiality Level

Each document has a confidentiality level.

```text
public
internal
confidential
restricted
```

### 9.1 public

Accessible to users who have basic document view permission and company/department access.

### 9.2 internal

Default level. Accessible to authorized internal users with company/department access.

### 9.3 confidential

Requires additional permission:

```text
document.view_confidential
```

### 9.4 restricted

Requires explicit document-level access or super admin permission.

Restricted documents should not appear in search results for unauthorized users.

---

## 10. Document-Level Override

Some documents may require special access rules. Use `document_access_rules` table.

### Table

```text
document_access_rules
- id
- document_id
- user_id
- role_id
- access_type
- can_access
- created_by
- created_at
- updated_at
```

### Access Types

```text
view
download
edit
delete
approve
manage_permissions
```

### Rule

Document-level rule can allow or deny a specific action for a user or role.

Examples:

```text
Allow User A to view Document 10.
Deny Role Viewer from downloading Document 15.
Allow Legal Manager role to manage permission for Document 20.
```

### Priority

Deny should take priority over allow.

Recommended evaluation order:

```text
1. Super Admin bypass
2. Explicit deny rule
3. Explicit allow rule
4. Role permission
5. Company access
6. Department access
7. Confidentiality rule
```

---

## 11. Approval-Based Access

Approvers need temporary or contextual access to documents assigned to them.

A user can view a document for approval if:

```text
- There is a pending document_approvals row assigned to the user
OR
- There is a pending document_approvals row assigned to one of the user's roles
```

Approval access should allow:

```text
- View document
- Preview document
- Add approval comment
- Approve/reject assigned step
```

Approval access should not automatically allow:

```text
- Edit document metadata
- Delete document
- Download document unless allowed
- Manage document permissions
```

---

## 12. Access Decision Examples

## 12.1 User Can View Document

A user can view a document if:

```text
- User has document.view permission
AND
- User has company access
AND
- User has department access
AND
- Document confidentiality rules allow it
AND
- No explicit deny rule exists
```

OR

```text
- User is assigned as current approver for that document
```

OR

```text
- User has explicit document-level view allow rule
```

---

## 12.2 User Can Download Document

A user can download if:

```text
- User can view the document
AND
- User has document.download permission
AND
- No explicit download deny rule exists
```

Restricted documents may require explicit download permission at document level.

---

## 12.3 User Can Edit Document

A user can edit if:

```text
- User has document.edit permission
AND
- User has company/department access
AND
- Document status is draft or rejected
AND
- User is uploader or admin
AND
- No explicit edit deny rule exists
```

Approved documents should not be edited directly. A new version should be uploaded instead.

---

## 12.4 User Can Delete Document

A user can delete/archive if:

```text
- User has document.delete or document.archive permission
AND
- User has company/department access
AND
- Document is not approved OR user has admin permission
AND
- No explicit delete deny rule exists
```

Hard delete should not be allowed in normal workflow.

---

## 12.5 User Can Approve Document

A user can approve if:

```text
- User has document.approve permission
AND
- User is assigned to the current pending approval step by user or role
AND
- Document status is pending_approval
AND
- No explicit approve deny rule exists
```

---

## 13. Laravel Implementation Strategy

## 13.1 Spatie Permission

Use Spatie Permission for:

```text
- roles
- permissions
- role_has_permissions
- model_has_roles
- model_has_permissions
```

Roles and permissions should be seeded initially.

---

## 13.2 Policies

Create policies for major models.

Recommended policies:

```text
CompanyPolicy
DepartmentPolicy
DocumentCategoryPolicy
DocumentPolicy
ApprovalFlowPolicy
AuditLogPolicy
```

`DocumentPolicy` is the most important.

Example policy methods:

```text
viewAny(User $user)
view(User $user, Document $document)
upload(User $user)
update(User $user, Document $document)
delete(User $user, Document $document)
download(User $user, Document $document)
approve(User $user, Document $document)
managePermissions(User $user, Document $document)
```

---

## 13.3 Access Services

Create a service to centralize access logic.

```text
app/Domains/AccessControl/Services/DocumentAccessService.php
```

Recommended methods:

```text
canView(User $user, Document $document): bool
canDownload(User $user, Document $document): bool
canEdit(User $user, Document $document): bool
canDelete(User $user, Document $document): bool
canApprove(User $user, Document $document): bool
canManagePermissions(User $user, Document $document): bool
applyVisibleDocumentsScope(Builder $query, User $user): Builder
```

Controllers and repositories should use this service instead of duplicating access logic.

---

## 13.4 Query Scoping

Document listing and search must apply user visibility filter.

Example:

```text
DocumentQuery
    → apply company access
    → apply department access
    → apply confidentiality access
    → apply document-level allow/deny rules
```

No document listing/search endpoint should return unscoped documents.

---

## 14. Search Access Control

Search results must respect access control.

Important rule:

```text
A user must never see unauthorized document title, metadata, or OCR text in search results.
```

### Recommended Approach

For MVP:

```text
Use MySQL query with access scopes.
```

For Meilisearch phase:

```text
1. Search in Meilisearch
2. Get matching document IDs
3. Apply database access control filter before returning final results
```

Alternative future approach:

```text
Use tenant/filter tokens or indexed access fields carefully.
```

But for initial implementation, database-level final access filtering is safer.

---

## 15. UI Access Control

Frontend should hide buttons based on permission, but backend must always enforce permission.

Example:

```text
If user cannot download:
- Hide Download button in UI
- Also block download route in backend
```

Never trust frontend-only access control.

---

## 16. Route Protection

Use middleware for general permission checks.

Example:

```text
Route::middleware(['auth', 'permission:document.view'])->group(function () {
    Route::get('/documents', [DocumentController::class, 'index']);
});
```

Use policies for model-specific checks.

Example:

```text
$this->authorize('view', $document);
```

---

## 17. Audit for Security Events

The following access-related actions should be audited:

```text
- User role changed
- User company access changed
- User department access changed
- Document permission changed
- Unauthorized access attempt if practical
- Document viewed
- Document downloaded
- Document approved/rejected
```

---

## 18. Initial Seed Data

## 18.1 Roles

```text
super_admin
company_admin
department_admin
document_uploader
document_approver
document_viewer
```

## 18.2 Permission Groups

```text
company permissions
department permissions
document category permissions
document permissions
approval permissions
audit log permissions
user/role permissions
```

## 18.3 Default Super Admin

Create one default super admin user during setup.

---

## 19. Common Mistakes to Avoid

```text
Do not rely only on roles.
Do not expose all documents and hide in frontend.
Do not use only company_id if department access is required.
Do not allow approved documents to be edited directly.
Do not let approvers access unrelated documents.
Do not allow download just because user can view.
Do not skip audit logs for permission changes.
Do not mix access logic inside controllers.
```

---

## 20. Codex Implementation Rules

Codex must follow these rules:

1. Install and configure Spatie Laravel Permission.
2. Create seeders for roles and permissions.
3. Create access mapping tables for company and department.
4. Create DocumentAccessService.
5. Create DocumentPolicy and use it in controllers.
6. Apply visibility scope in document list/search repositories.
7. Never return unscoped document queries.
8. Keep access logic reusable and testable.
9. Do not duplicate access logic in Blade/Vue.
10. Backend permission checks are mandatory.

---

## 21. Summary

The DMS requires layered access control, not simple role checks. The system must combine Spatie role permissions with company, department, confidentiality, approval assignment, and document-level override rules.

The most important implementation rule is:

```text
Every document query must be scoped by the authenticated user's access rights.
```

If access control is designed correctly from the start, the system can safely handle sensitive legal, HR, finance, land, and management documents across all concerns of the group company.

