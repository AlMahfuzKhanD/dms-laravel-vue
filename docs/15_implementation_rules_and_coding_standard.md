# Document Management System (DMS) — Implementation Rules & Coding Standard

## 1. Purpose

This document defines coding standards, implementation rules, naming conventions, and architectural guidelines for the Document Management System.

The goal is to ensure that all code generated manually or by Codex AI agent remains clean, consistent, secure, maintainable, and scalable.

---

## 2. Main Implementation Principle

```text
Controllers handle HTTP.
Services handle business logic.
Repositories handle complex queries.
Policies handle authorization.
Requests handle validation.
Events/listeners handle side effects.
```

No single class should do everything.

---

## 3. Technology Stack

```text
Backend: Laravel 12
Frontend: Blade + Vue 3 Hybrid
Database: MySQL 8
Search MVP: MySQL
Search Later: Laravel Scout + Meilisearch
Queue: Redis
File Storage: Laravel private storage / S3 later
Permission: Spatie Laravel Permission + custom access service
OCR Later: Tesseract OCR
CSS/UI: Tailwind CSS or project admin theme
```

---

## 4. Folder Structure Standard

Use domain-based modular structure.

```text
app/
 ├── Domains/
 │    ├── Company/
 │    │    ├── Controllers/
 │    │    ├── Models/
 │    │    ├── Requests/
 │    │    ├── Services/
 │    │    ├── Repositories/
 │    │    └── Policies/
 │    │
 │    ├── Department/
 │    ├── DocumentCategory/
 │    ├── Metadata/
 │    ├── Document/
 │    ├── AccessControl/
 │    ├── Approval/
 │    ├── Search/
 │    ├── AuditLog/
 │    ├── Settings/
 │    └── Ocr/
 │
 ├── Shared/
 │    ├── Enums/
 │    ├── DTOs/
 │    ├── Helpers/
 │    └── Traits/
 │
 └── Support/
      ├── FileStorage/
      ├── SearchIndexing/
      └── Ocr/
```

Create folders only when needed.

---

## 5. Naming Convention

## 5.1 Classes

```text
Controller: DocumentController
Service: DocumentCreateService
Repository: DocumentRepository
Request: StoreDocumentRequest
Policy: DocumentPolicy
Enum: DocumentStatus
Job: ProcessDocumentOcrJob
Event: DocumentUploaded
Listener: WriteAuditLog
```

---

## 5.2 Database Tables

Use snake_case plural names.

```text
companies
departments
document_categories
documents
document_files
document_metadata_values
approval_flows
approval_steps
document_approvals
audit_logs
system_settings
```

---

## 5.3 Columns

Use snake_case.

```text
company_id
department_id
category_id
document_no
uploaded_by
approved_at
is_latest
created_by
updated_by
```

---

## 5.4 Route Names

Use clear names with `dms.` prefix.

```text
dms.documents.index
dms.documents.store
dms.documents.show
dms.documents.download
dms.approvals.pending
dms.settings.update
```

---

## 6. Controller Rules

Controllers must be thin.

Controller responsibilities:

```text
1. Receive request
2. Authorize action
3. Pass validated data to service
4. Return view/redirect/JSON response
```

Controllers must not contain:

```text
- Complex DB queries
- File storage logic
- Approval step logic
- Search filtering logic
- Permission calculation logic
- OCR processing logic
```

### Example Controller Pattern

```php
public function store(StoreDocumentRequest $request, DocumentCreateService $service)
{
    $document = $service->create($request->validated(), $request->user());

    return redirect()
        ->route('dms.documents.show', $document)
        ->with('success', 'Document created successfully.');
}
```

---

## 7. Service Layer Rules

Services contain business logic.

Examples:

```text
DocumentCreateService
DocumentUpdateService
DocumentVersionService
DocumentFileStorageService
DocumentAccessService
DocumentApprovalService
ApprovalFlowResolverService
DocumentSearchService
AuditLogService
SettingsService
```

### Service Rules

```text
1. Services should be reusable.
2. Services should not directly return Blade views.
3. Services may use repositories.
4. Services should use transactions for multi-table writes.
5. Services should dispatch events for side effects.
```

---

## 8. Repository Rules

Repositories handle complex database queries.

Use repositories when:

```text
- Query has many joins
- Query has many filters
- Query is reused
- Query includes access scopes
- Query is hard to keep inside model scope
```

Do not create repositories for every tiny query if not needed.

Example repositories:

```text
DocumentRepository
DocumentSearchRepository
ApprovalRepository
AuditLogRepository
```

---

## 9. Request Validation Rules

Use Laravel Form Request classes.

Examples:

```text
StoreDocumentRequest
UpdateDocumentRequest
StoreApprovalFlowRequest
DocumentSearchRequest
UpdateSystemSettingRequest
```

Validation should include:

```text
- Required fields
- Data type
- File type
- File size
- Foreign key existence
- Enum values
- Conditional metadata fields
```

Do not validate complex business rules only in controller.

---

## 10. Authorization Rules

Use multiple layers:

```text
1. Middleware for general permission
2. Policy for model-specific permission
3. DocumentAccessService for document visibility/action rules
```

Important:

```text
Frontend permission hiding is not security.
Backend authorization is mandatory.
```

### Must-Protect Actions

```text
View document
Preview document
Download document
Edit document
Delete/archive document
Submit for approval
Approve/reject document
Manage document permissions
View audit logs
Update settings
```

---

## 11. Document Query Rule

Most important rule:

```text
Never return unscoped document queries.
```

Every document list/search query must use:

```text
DocumentAccessService::applyVisibleDocumentsScope($query, $user)
```

or equivalent access-aware scope.

---

## 12. File Handling Rules

```text
1. Store files in private storage.
2. Never store files in public folder.
3. Never expose file_path to frontend.
4. Always check permission before preview/download.
5. Generate stored file name safely.
6. Preserve original file name in DB.
7. Use checksum.
8. Use versioning.
9. Do not overwrite approved documents.
```

---

## 13. Approval Rules

Approval must follow the latest **06-approval-workflow-design**.

Critical rules:

```text
1. Approval is 100% dynamic.
2. Do not hardcode approval steps.
3. Category controls whether approval is required.
4. approval_flows controls applicable workflow.
5. approval_steps controls number of steps and approvers.
6. Step approver can be user or role.
7. Only one step is active at a time.
8. Rejected document can be edited and resubmitted.
9. Old approval history must remain.
10. New version requires new approval if category requires approval.
```

---

## 14. Transaction Rules

Use DB transactions for operations that write multiple tables.

Examples:

```text
- Create document + metadata + file
- Upload new version
- Submit for approval
- Approve/reject step
- Change document permissions
- Delete/archive document
```

Example:

```php
return DB::transaction(function () use ($data, $user) {
    // multi-table operation
});
```

---

## 15. Event & Listener Rules

Use events/listeners for side effects.

Examples:

```text
DocumentCreated
DocumentUpdated
DocumentFileUploaded
DocumentDownloaded
DocumentSubmittedForApproval
DocumentApproved
DocumentRejected
DocumentPermissionChanged
SystemSettingUpdated
```

Listeners:

```text
WriteAuditLog
SendNotification
UpdateSearchIndex
DispatchOcrJob
```

Do not mix all side effects directly inside controller.

---

## 16. Enum / Constant Rules

Use enums or constants for fixed values.

Examples:

```text
DocumentStatus
ApprovalStatus
ConfidentialityLevel
MetadataFieldType
OcrStatus
AccessType
```

Avoid magic strings scattered everywhere.

Bad:

```php
$document->status = 'pending_approval';
```

Better:

```php
$document->status = DocumentStatus::PENDING_APPROVAL->value;
```

---

## 17. Error Handling Rules

Use clear, user-friendly errors.

Examples:

```text
You are not authorized to download this document.
Approval flow not configured for this category.
This file type is not allowed.
Document cannot be edited while pending approval.
Approved document cannot be overwritten. Upload a new version instead.
```

Do not expose system errors to normal users.

---

## 18. JSON Response Standard

Success:

```json
{
  "success": true,
  "message": "Operation completed successfully.",
  "data": {}
}
```

Error:

```json
{
  "success": false,
  "message": "Something went wrong.",
  "errors": {}
}
```

---

## 19. Blade + Vue Rules

```text
1. Blade handles layout/page shell.
2. Vue handles dynamic forms and interactive sections.
3. Pass initial data through JSON-safe props/data attributes.
4. Do not hardcode permission-only UI state in Vue.
5. Backend must return allowed actions if needed.
6. Use reusable Vue components for metadata fields, approval step builder, search filters, timelines.
```

---

## 20. UI Permission Rules

UI should hide unavailable actions.

But backend must always enforce.

Example:

```text
If user cannot download:
- Hide Download button
- Also block download route
```

---

## 21. Search Implementation Rules

MVP:

```text
Use MySQL search first.
```

Later:

```text
Laravel Scout + Meilisearch.
```

Critical rule:

```text
Search result must pass access control before returning.
```

Do not return raw Meilisearch results directly.

---

## 22. OCR Implementation Rules

OCR is later phase.

Rules:

```text
1. OCR must run in queue.
2. Upload must not wait for OCR.
3. OCR text belongs to document_file_id.
4. OCR failure must not break document upload.
5. OCR text must respect document access control.
```

---

## 23. Audit Log Rules

Audit these actions:

```text
Document create/update/delete/archive
File upload/preview/download
Approval submit/approve/reject
Permission changes
Settings changes
Access changes
```

Audit logs must be append-only.

Normal users must not edit/delete audit logs.

---

## 24. Settings Rules

Use SettingsService for configurable values.

Do not hardcode operational values like:

```text
max file size
allowed file types
OCR enabled
search driver
approval fallback
records per page
```

---

## 25. Migration Rules

```text
1. Use proper foreign keys where practical.
2. Add indexes for searchable/filterable columns.
3. Use soft deletes for important business tables.
4. Keep files separate from document records.
5. Keep approval setup separate from approval execution.
6. Keep metadata definitions separate from metadata values.
```

---

## 26. Model Relationship Rules

Define clear relationships.

Examples:

```php
public function company()
{
    return $this->belongsTo(Company::class);
}

public function files()
{
    return $this->hasMany(DocumentFile::class);
}

public function latestFile()
{
    return $this->hasOne(DocumentFile::class)->where('is_latest', true);
}
```

---

## 27. Testing Rules

Prioritize tests for critical business logic.

Important tests:

```text
User cannot see unassigned company document
User cannot download without permission
Approved document cannot be overwritten
New version increments version number
Approval flow supports dynamic steps
Only current approver can approve
Search does not return unauthorized documents
Explicit deny overrides allow
```

---

## 28. Security Rules

```text
1. Validate every request.
2. Authorize every sensitive action.
3. Do not trust frontend.
4. Do not expose storage paths.
5. Do not store secrets in system_settings.
6. Do not show restricted documents in search.
7. Sanitize file names.
8. Limit allowed file types.
9. Audit security-sensitive changes.
```

---

## 29. Codex-Specific Rules

Codex must:

```text
1. Work one phase at a time.
2. Avoid large uncontrolled refactors.
3. Read relevant docs before implementation.
4. Not invent new architecture.
5. Not introduce unrelated packages without approval.
6. Keep generated code consistent with this standard.
7. Explain changed files after each task.
8. Include migration/model/service/request/controller where required.
9. Avoid putting everything into one class.
10. Preserve existing project structure.
```

---

## 30. Code Review Checklist

Before accepting generated code, check:

```text
- Does it follow domain folder structure?
- Is controller thin?
- Is validation in Form Request?
- Is authorization applied?
- Is business logic in service?
- Are document queries access-scoped?
- Are files private?
- Are statuses using enum/constants?
- Are transactions used where needed?
- Are audit events dispatched?
- Is there any hardcoded approval step?
- Is there any direct file path exposed?
```

---

## 31. Summary

This coding standard exists to keep the DMS enterprise-grade and maintainable.

The most important rule is:

```text
Do not build a controller-heavy file upload system. Build a secure, modular, workflow-driven document management platform.
```

If Codex follows these rules, the project will remain clean, scalable, and safe for real company use.

