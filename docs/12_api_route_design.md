# Document Management System (DMS) — API & Route Design

## 1. Purpose

This document defines the route and API structure for the Document Management System.

The system will use Laravel with Blade + Vue 3 hybrid UI. Most pages can be served through web routes, while dynamic operations can use JSON endpoints.

This document is written to help Codex generate controllers, route files, request classes, services, and frontend API calls consistently.

---

## 2. Architecture Style

Recommended style:

```text
Laravel Web App + Internal JSON APIs
```

Meaning:

```text
- Blade routes render main pages
- Vue components handle complex forms/interactions
- JSON endpoints provide dynamic data
- Backend services handle business logic
```

This is not a public API-first SaaS in the first version.

---

## 3. Route Organization

Recommended files:

```text
routes/web.php
routes/admin.php
routes/api.php
```

Preferred:

```text
routes/admin.php → authenticated admin/DMS routes
routes/api.php   → internal JSON endpoints if needed
```

All DMS routes should be protected by authentication.

---

## 4. Route Prefix

Recommended route prefix:

```text
/admin/dms
```

Example:

```text
/admin/dms/documents
/admin/dms/companies
/admin/dms/departments
```

Route name prefix:

```text
dms.
```

Example:

```text
dms.documents.index
dms.documents.create
dms.documents.store
```

---

## 5. Common Middleware

All DMS routes should use:

```text
auth
verified optional
permission middleware where needed
```

Example:

```php
Route::middleware(['auth'])->prefix('admin/dms')->name('dms.')->group(function () {
    // routes
});
```

Permission and policy must still be checked inside controllers for model-specific actions.

---

# 6. Page Routes

These routes return Blade pages.

## 6.1 Dashboard

```text
GET /admin/dms/dashboard
Name: dms.dashboard
Controller: DashboardController@index
Permission: dms.dashboard.view
```

---

## 6.2 Company Routes

```text
GET    /admin/dms/companies
GET    /admin/dms/companies/create
POST   /admin/dms/companies
GET    /admin/dms/companies/{company}/edit
PUT    /admin/dms/companies/{company}
DELETE /admin/dms/companies/{company}
```

Route names:

```text
dms.companies.index
dms.companies.create
dms.companies.store
dms.companies.edit
dms.companies.update
dms.companies.destroy
```

Controller:

```text
CompanyController
```

---

## 6.3 Department Routes

```text
GET    /admin/dms/departments
GET    /admin/dms/departments/create
POST   /admin/dms/departments
GET    /admin/dms/departments/{department}/edit
PUT    /admin/dms/departments/{department}
DELETE /admin/dms/departments/{department}
```

Route names:

```text
dms.departments.index
dms.departments.create
dms.departments.store
dms.departments.edit
dms.departments.update
dms.departments.destroy
```

Controller:

```text
DepartmentController
```

---

## 6.4 Document Category Routes

```text
GET    /admin/dms/categories
GET    /admin/dms/categories/create
POST   /admin/dms/categories
GET    /admin/dms/categories/{category}/edit
PUT    /admin/dms/categories/{category}
DELETE /admin/dms/categories/{category}
```

Route names:

```text
dms.categories.index
dms.categories.create
dms.categories.store
dms.categories.edit
dms.categories.update
dms.categories.destroy
```

Controller:

```text
DocumentCategoryController
```

---

## 6.5 Metadata Field Routes

Metadata fields are configured per document category.

```text
GET    /admin/dms/categories/{category}/metadata-fields
POST   /admin/dms/categories/{category}/metadata-fields
GET    /admin/dms/metadata-fields/{field}/edit
PUT    /admin/dms/metadata-fields/{field}
DELETE /admin/dms/metadata-fields/{field}
```

Route names:

```text
dms.categories.metadata-fields.index
dms.categories.metadata-fields.store
dms.metadata-fields.edit
dms.metadata-fields.update
dms.metadata-fields.destroy
```

Controller:

```text
MetadataFieldController
```

---

# 7. Document Routes

## 7.1 Document List/Search Page

```text
GET /admin/dms/documents
Name: dms.documents.index
Controller: DocumentController@index
Permission: document.view
```

Returns document listing/search page.

---

## 7.2 Document Create Page

```text
GET /admin/dms/documents/create
Name: dms.documents.create
Controller: DocumentController@create
Permission: document.upload
```

---

## 7.3 Store Document

```text
POST /admin/dms/documents
Name: dms.documents.store
Controller: DocumentController@store
Permission: document.upload
```

Uses:

```text
StoreDocumentRequest
DocumentCreateService
DocumentFileStorageService
DocumentVersionService
```

---

## 7.4 Show Document

```text
GET /admin/dms/documents/{document}
Name: dms.documents.show
Controller: DocumentController@show
Policy: DocumentPolicy@view
```

Shows:

```text
- document details
- metadata
- latest file
- version history
- approval timeline
- activity timeline
```

---

## 7.5 Edit Document

```text
GET /admin/dms/documents/{document}/edit
Name: dms.documents.edit
Controller: DocumentController@edit
Policy: DocumentPolicy@update
```

---

## 7.6 Update Document

```text
PUT /admin/dms/documents/{document}
Name: dms.documents.update
Controller: DocumentController@update
Policy: DocumentPolicy@update
```

Uses:

```text
UpdateDocumentRequest
DocumentUpdateService
```

---

## 7.7 Delete/Archive Document

Recommended first behavior: archive or soft delete, not hard delete.

```text
DELETE /admin/dms/documents/{document}
Name: dms.documents.destroy
Controller: DocumentController@destroy
Policy: DocumentPolicy@delete
```

Optional archive route:

```text
POST /admin/dms/documents/{document}/archive
Name: dms.documents.archive
Controller: DocumentArchiveController@store
Policy: DocumentPolicy@archive
```

---

# 8. Document File Routes

## 8.1 Upload New Version

```text
POST /admin/dms/documents/{document}/versions
Name: dms.documents.versions.store
Controller: DocumentVersionController@store
Policy: DocumentPolicy@update
```

Uses:

```text
StoreDocumentVersionRequest
DocumentVersionService
```

---

## 8.2 Download Latest File

```text
GET /admin/dms/documents/{document}/download
Name: dms.documents.download
Controller: DocumentDownloadController@latest
Policy: DocumentPolicy@download
```

---

## 8.3 Download Specific File Version

```text
GET /admin/dms/documents/{document}/files/{file}/download
Name: dms.documents.files.download
Controller: DocumentDownloadController@file
Policy: DocumentPolicy@download
```

---

## 8.4 Preview Latest File

```text
GET /admin/dms/documents/{document}/preview
Name: dms.documents.preview
Controller: DocumentPreviewController@latest
Policy: DocumentPolicy@view
```

---

## 8.5 Preview Specific File Version

```text
GET /admin/dms/documents/{document}/files/{file}/preview
Name: dms.documents.files.preview
Controller: DocumentPreviewController@file
Policy: DocumentPolicy@view
```

Important:

```text
Never expose direct storage path.
```

---

# 9. Approval Routes

## 9.1 Approval Flow Setup Routes

```text
GET    /admin/dms/approval-flows
GET    /admin/dms/approval-flows/create
POST   /admin/dms/approval-flows
GET    /admin/dms/approval-flows/{flow}/edit
PUT    /admin/dms/approval-flows/{flow}
DELETE /admin/dms/approval-flows/{flow}
```

Controller:

```text
ApprovalFlowController
```

---

## 9.2 Approval Step Routes

```text
GET    /admin/dms/approval-flows/{flow}/steps
POST   /admin/dms/approval-flows/{flow}/steps
PUT    /admin/dms/approval-steps/{step}
DELETE /admin/dms/approval-steps/{step}
POST   /admin/dms/approval-flows/{flow}/steps/reorder
```

Controller:

```text
ApprovalStepController
```

---

## 9.3 Submit Document for Approval

```text
POST /admin/dms/documents/{document}/submit-approval
Name: dms.documents.submit-approval
Controller: DocumentApprovalController@submit
Policy: DocumentPolicy@submitForApproval
```

Uses:

```text
DocumentApprovalService
ApprovalFlowResolverService
```

---

## 9.4 Pending Approval List

```text
GET /admin/dms/approvals/pending
Name: dms.approvals.pending
Controller: PendingApprovalController@index
Permission: document.approve
```

Shows documents assigned to current user or current user's roles.

---

## 9.5 Approve Document Step

```text
POST /admin/dms/documents/{document}/approve
Name: dms.documents.approve
Controller: DocumentApprovalActionController@approve
Policy: DocumentPolicy@approve
```

---

## 9.6 Reject Document Step

```text
POST /admin/dms/documents/{document}/reject
Name: dms.documents.reject
Controller: DocumentApprovalActionController@reject
Policy: DocumentPolicy@approve
```

---

# 10. Search Routes

## 10.1 Document Search JSON Endpoint

```text
GET /admin/dms/api/documents/search
Name: dms.api.documents.search
Controller: DocumentSearchController@index
Permission: document.view
```

Returns JSON for Vue/DataTable/search UI.

Query parameters:

```text
keyword
company_id
department_id
category_id
status
confidentiality_level
document_date_from
document_date_to
expiry_date_from
expiry_date_to
uploaded_by
tags[]
metadata[key]
page
per_page
```

Important:

```text
Must apply DocumentAccessService visible scope.
```

---

## 10.2 Dynamic Metadata Fields by Category

```text
GET /admin/dms/api/categories/{category}/metadata-fields
Name: dms.api.categories.metadata-fields
Controller: CategoryMetadataApiController@index
```

Used by document create/edit/search forms.

---

## 10.3 Departments by Company

```text
GET /admin/dms/api/companies/{company}/departments
Name: dms.api.companies.departments
Controller: CompanyDepartmentApiController@index
```

Used for dependent dropdowns.

---

## 10.4 Categories by Department

```text
GET /admin/dms/api/departments/{department}/categories
Name: dms.api.departments.categories
Controller: DepartmentCategoryApiController@index
```

---

# 11. Access Control Routes

## 11.1 User Company Access

```text
GET  /admin/dms/users/{user}/company-access
POST /admin/dms/users/{user}/company-access
```

Controller:

```text
UserCompanyAccessController
```

---

## 11.2 User Department Access

```text
GET  /admin/dms/users/{user}/department-access
POST /admin/dms/users/{user}/department-access
```

Controller:

```text
UserDepartmentAccessController
```

---

## 11.3 Document Permission Override

```text
GET  /admin/dms/documents/{document}/permissions
POST /admin/dms/documents/{document}/permissions
DELETE /admin/dms/documents/{document}/permissions/{rule}
```

Controller:

```text
DocumentAccessRuleController
```

Policy:

```text
DocumentPolicy@managePermissions
```

---

# 12. Audit Log Routes

## 12.1 Global Audit Log

```text
GET /admin/dms/audit-logs
Name: dms.audit-logs.index
Controller: AuditLogController@index
Permission: audit_log.view
```

---

## 12.2 Document Timeline

```text
GET /admin/dms/documents/{document}/timeline
Name: dms.documents.timeline
Controller: DocumentTimelineController@index
Policy: DocumentPolicy@view
```

Can return Blade partial or JSON.

---

# 13. OCR Routes — Later Phase

## 13.1 OCR Status

```text
GET /admin/dms/documents/{document}/ocr-status
Name: dms.documents.ocr-status
Controller: DocumentOcrController@status
Policy: DocumentPolicy@view
```

---

## 13.2 Reprocess OCR

```text
POST /admin/dms/documents/{document}/ocr-reprocess
Name: dms.documents.ocr-reprocess
Controller: DocumentOcrController@reprocess
Policy: DocumentPolicy@update
```

---

# 14. System Settings Routes

```text
GET /admin/dms/settings
PUT /admin/dms/settings
```

Route names:

```text
dms.settings.index
dms.settings.update
```

Controller:

```text
SystemSettingController
```

Permission:

```text
system_settings.manage
```

---

# 15. Request Classes

Recommended Laravel Form Requests:

```text
StoreCompanyRequest
UpdateCompanyRequest
StoreDepartmentRequest
UpdateDepartmentRequest
StoreDocumentCategoryRequest
UpdateDocumentCategoryRequest
StoreMetadataFieldRequest
UpdateMetadataFieldRequest
StoreDocumentRequest
UpdateDocumentRequest
StoreDocumentVersionRequest
StoreApprovalFlowRequest
UpdateApprovalFlowRequest
StoreApprovalStepRequest
UpdateApprovalStepRequest
ApproveDocumentRequest
RejectDocumentRequest
DocumentSearchRequest
UpdateSystemSettingRequest
```

---

# 16. Controller Rule

Controllers must be thin.

Controller responsibility:

```text
- Authorize request
- Validate request
- Call service
- Return response/view/redirect
```

Controllers should not contain:

```text
- Complex document upload logic
- Approval step logic
- Search query logic
- Permission calculation logic
- File storage logic
```

---

# 17. API Response Standard

For JSON endpoints, use consistent response format.

Success:

```json
{
  "success": true,
  "message": "Document uploaded successfully.",
  "data": {}
}
```

Validation error:

```json
{
  "success": false,
  "message": "Validation failed.",
  "errors": {}
}
```

Authorization error:

```json
{
  "success": false,
  "message": "You are not authorized to perform this action."
}
```

---

# 18. Pagination Standard

Search/list API should return:

```json
{
  "data": [],
  "meta": {
    "total": 100,
    "per_page": 20,
    "current_page": 1,
    "last_page": 5
  }
}
```

---

# 19. Codex Rules

Codex must follow these rules:

1. Use route names consistently.
2. Keep controllers thin.
3. Use Form Requests for validation.
4. Use Policies for model authorization.
5. Use Services for business logic.
6. Use Repositories for complex queries.
7. Never expose direct file paths.
8. Always apply access control to search/list routes.
9. Do not build public API unless instructed.
10. Keep web pages and JSON endpoints separated clearly.

---

# 20. Recommended Implementation Order

```text
1. Register admin DMS route group
2. Build company routes/controllers
3. Build department routes/controllers
4. Build category routes/controllers
5. Build metadata field routes/controllers
6. Build document routes/controllers
7. Build file preview/download routes
8. Build approval flow setup routes
9. Build approval action routes
10. Build search JSON routes
11. Build access-control routes
12. Build audit routes
13. Build settings routes
14. Add OCR routes later
```

---

# 21. Summary

This route design supports a Laravel Blade + Vue hybrid DMS. It separates page routes, JSON endpoints, file actions, approval actions, search, access control, audit logs, OCR, and settings.

The most important rule is:

```text
Routes should be clean, predictable, protected, and backed by service-layer business logic.
```

