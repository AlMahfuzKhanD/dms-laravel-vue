# Document Management System (DMS) — Controller, Request & Policy Codex Prompts

## 1. Purpose

This document contains ready-to-use Codex prompts for implementing controllers, Form Requests, and Policies for the DMS.

The goal is to keep the HTTP layer clean, secure, and consistent.

Codex should use this document together with:

```text
03-module-breakdown
04-database-design
05-permission-access-control-design
06-approval-workflow-design
07-search-architecture-design
08-file-storage-versioning-design
09-audit-log-activity-design
11-system-settings-configuration-design
12-api-route-design
13-ui-ux-screen-flow-design
15-implementation-rules-and-coding-standard
17-service-repository-codex-prompts
```

---

## 2. Global HTTP Layer Rules

Codex must follow these rules:

```text
1. Controllers must be thin.
2. Controllers must not contain business logic.
3. Use Form Requests for validation.
4. Use Policies for model-specific authorization.
5. Use middleware for broad permission checks.
6. Use Services for workflow/business operations.
7. Use Repositories for complex listing/search queries.
8. Never expose private file paths.
9. Never return unscoped document queries.
10. Always return safe and user-friendly responses.
```

---

## 3. Controller Responsibility

Controllers should only:

```text
1. Receive request
2. Authorize action
3. Use validated data
4. Call service/repository
5. Return view, redirect, or JSON response
```

Bad:

```text
Controller directly stores file, calculates version, creates approval rows, and writes audit logs.
```

Good:

```text
Controller validates request and calls DocumentCreateService.
```

---

## 4. Form Request Responsibility

Form Requests should handle:

```text
- Required field validation
- Type validation
- File validation
- Enum validation
- Foreign key existence validation
- Conditional validation
- Basic authorization if appropriate
```

Complex business rules should be handled by services.

---

## 5. Policy Responsibility

Policies should answer:

```text
Can this user perform this action on this model?
```

Policies may call access services.

Example:

```text
DocumentPolicy calls DocumentAccessService.
```

---

# 6. Prompt 01 — Company Controller, Requests, Policy

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement CompanyController, StoreCompanyRequest, UpdateCompanyRequest, and CompanyPolicy.

Controller actions:
- index
- create
- store
- edit
- update
- destroy

Rules:
- Controller must be thin.
- Use CompanyService for create/update/delete.
- Use CompanyRepository for listing.
- Use Form Requests for validation.
- Use CompanyPolicy or permission middleware for authorization.
- Return Blade views for index/create/edit.
- Redirect with success/error messages.

Validation:
- name required string max 150
- code nullable string max 50 unique companies code
- email nullable email max 100
- phone nullable string max 50
- status required boolean/integer

Policy methods:
- viewAny
- create
- update
- delete

Do not implement unrelated modules.
```

---

# 7. Prompt 02 — Department Controller, Requests, Policy

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement DepartmentController, StoreDepartmentRequest, UpdateDepartmentRequest, and DepartmentPolicy.

Controller actions:
- index
- create
- store
- edit
- update
- destroy

Rules:
- Use DepartmentService for create/update/delete.
- Use DepartmentRepository for listing.
- Controller must be thin.
- Use policy/permission checks.
- Return Blade views.

Validation:
- company_id required exists companies id
- name required string max 150
- code nullable string max 50 unique per company
- status required boolean/integer

Policy methods:
- viewAny
- create
- update
- delete

Do not implement document logic here.
```

---

# 8. Prompt 03 — Document Category Controller, Requests, Policy

```text
Read 03-module-breakdown, 04-database-design, 06-approval-workflow-design, 12-api-route-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentCategoryController, StoreDocumentCategoryRequest, UpdateDocumentCategoryRequest, and DocumentCategoryPolicy.

Controller actions:
- index
- create
- store
- edit
- update
- destroy

Rules:
- Use DocumentCategoryService for create/update/delete.
- Use DocumentCategoryRepository for listing.
- Controller must be thin.
- Support parent-child category.
- Support company/department nullable category.
- Include requires_approval boolean.

Validation:
- parent_id nullable exists document_categories id
- company_id nullable exists companies id
- department_id nullable exists departments id
- name required string max 150
- code nullable string max 50
- description nullable string
- requires_approval boolean
- status required boolean/integer

Policy methods:
- viewAny
- create
- update
- delete

Important:
- requires_approval only controls whether approval is required.
- Do not hardcode approval steps in category controller.
```

---

# 9. Prompt 04 — Metadata Field Controller and Requests

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement MetadataFieldController, StoreMetadataFieldRequest, UpdateMetadataFieldRequest, and policy if needed.

Controller actions:
- index by category
- store by category
- edit
- update
- destroy

Rules:
- Use MetadataFieldService.
- Controller must be thin.
- Field key must be unique per category.
- Field type must use MetadataFieldType enum.
- Options are required only for select/multi_select if needed.

Validation:
- label required string max 150
- field_key required string max 100 regex snake_case
- field_type required valid enum
- placeholder nullable string max 150
- options nullable array/json
- is_required boolean
- is_searchable boolean
- sort_order integer
- status boolean/integer

Do not hardcode category-specific metadata fields.
```

---

# 10. Prompt 05 — Metadata API Controllers

```text
Read 12-api-route-design and 13-ui-ux-screen-flow-design.

Implement API controllers for dependent dropdowns and dynamic metadata fields:

1. CategoryMetadataApiController@index
Route: GET /admin/dms/api/categories/{category}/metadata-fields
Returns active metadata fields for selected category.

2. CompanyDepartmentApiController@index
Route: GET /admin/dms/api/companies/{company}/departments
Returns active departments by company.

3. DepartmentCategoryApiController@index
Route: GET /admin/dms/api/departments/{department}/categories
Returns active categories by department, including global categories if applicable.

Rules:
- Return JSON.
- Apply access control where needed.
- Do not expose sensitive data.
- Keep controllers thin.
```

---

# 11. Prompt 06 — Document Controller and Requests

```text
Read 04-database-design, 05-permission-access-control-design, 08-file-storage-versioning-design, 12-api-route-design, 13-ui-ux-screen-flow-design, 15-implementation-rules-and-coding-standard, and 17-service-repository-codex-prompts.

Implement DocumentController, StoreDocumentRequest, and UpdateDocumentRequest.

Controller actions:
- index
- create
- store
- show
- edit
- update
- destroy

Rules:
- Use DocumentRepository for index/list.
- Use DocumentCreateService for store.
- Use DocumentUpdateService for update.
- Use DocumentPolicy for show/edit/update/destroy.
- Controller must be thin.
- Never store files directly in controller.
- Never expose private file path.
- Document list must be access-scoped.

StoreDocumentRequest validation:
- company_id required exists companies id
- department_id required exists departments id
- category_id required exists document_categories id
- title required string max 255
- document_no nullable string max 100
- document_date nullable date
- expiry_date nullable date after_or_equal document_date if both present
- confidentiality_level required valid enum
- summary nullable string
- remarks nullable string
- tags nullable array
- tags.* string max 100
- metadata nullable array
- file required file with allowed types and max size from SettingsService if practical
- action required in draft,submit

UpdateDocumentRequest validation:
- title required string max 255
- document_no nullable string max 100
- document_date nullable date
- expiry_date nullable date
- confidentiality_level required valid enum
- summary nullable string
- remarks nullable string
- tags nullable array
- metadata nullable array

Important:
- Dynamic metadata required validation can be delegated to MetadataValueService.
- Approval submission behavior must be service-driven.
```

---

# 12. Prompt 07 — Document File Controllers and Requests

```text
Read 08-file-storage-versioning-design, 05-permission-access-control-design, 12-api-route-design, 15-implementation-rules-and-coding-standard, and 17-service-repository-codex-prompts.

Implement:
- DocumentVersionController
- DocumentDownloadController
- DocumentPreviewController
- StoreDocumentVersionRequest

Routes:
- POST /admin/dms/documents/{document}/versions
- GET /admin/dms/documents/{document}/download
- GET /admin/dms/documents/{document}/files/{file}/download
- GET /admin/dms/documents/{document}/preview
- GET /admin/dms/documents/{document}/files/{file}/preview

Rules:
- Use DocumentVersionService for new versions.
- Use DocumentFileDownloadService for download.
- Use DocumentFilePreviewService for preview.
- Check DocumentPolicy / DocumentAccessService.
- Verify file belongs to document.
- Never expose storage path.
- Log/audit download and preview.

StoreDocumentVersionRequest validation:
- file required file allowed types max size from settings
- note nullable string max 1000
```

---

# 13. Prompt 08 — DocumentPolicy

```text
Read 05-permission-access-control-design and 15-implementation-rules-and-coding-standard.

Implement DocumentPolicy.

Methods:
- viewAny(User $user)
- view(User $user, Document $document)
- create(User $user)
- update(User $user, Document $document)
- delete(User $user, Document $document)
- archive(User $user, Document $document)
- download(User $user, Document $document)
- submitForApproval(User $user, Document $document)
- approve(User $user, Document $document)
- managePermissions(User $user, Document $document)

Rules:
- Use DocumentAccessService for document-specific checks.
- Super admin bypass can be handled in before() method.
- create requires document.upload permission.
- viewAny requires document.view permission.
- Never duplicate large access logic inside policy; delegate to DocumentAccessService.
```

---

# 14. Prompt 09 — Document Search Controller and Request

```text
Read 07-search-architecture-design, 05-permission-access-control-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentSearchController and DocumentSearchRequest.

Route:
GET /admin/dms/api/documents/search

Validation/query params:
- keyword nullable string max 255
- company_id nullable exists companies id
- department_id nullable exists departments id
- category_id nullable exists document_categories id
- status nullable string valid DocumentStatus
- confidentiality_level nullable string valid ConfidentialityLevel
- document_date_from nullable date
- document_date_to nullable date
- expiry_date_from nullable date
- expiry_date_to nullable date
- uploaded_by nullable exists users id
- tags nullable array
- metadata nullable array
- page nullable integer
- per_page nullable integer max 100

Rules:
- Use DocumentSearchService.
- Apply access control inside service/repository.
- Return paginated JSON.
- Do not return unauthorized documents.
- Do not implement Meilisearch yet unless instructed.
```

---

# 15. Prompt 10 — Approval Flow Controllers and Requests

```text
Read the latest 06-approval-workflow-design, 12-api-route-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement ApprovalFlowController, ApprovalStepController, StoreApprovalFlowRequest, UpdateApprovalFlowRequest, StoreApprovalStepRequest, UpdateApprovalStepRequest, and policies if needed.

ApprovalFlowController actions:
- index
- create
- store
- edit
- update
- destroy

ApprovalStepController actions:
- index by flow
- store
- update
- destroy
- reorder

Flow validation:
- company_id nullable exists companies id
- department_id nullable exists departments id
- category_id required exists document_categories id
- name required string max 150
- is_default boolean
- auto_approve_if_no_flow boolean
- status boolean/integer

Step validation:
- approval_flow_id required exists approval_flows id
- step_order required integer min 1
- step_name nullable string max 150
- approver_type required in user,role
- approver_user_id required_if approver_type user nullable exists users id
- approver_role_id required_if approver_type role nullable exists roles id
- is_required boolean
- can_edit_document boolean
- can_download_document boolean
- requires_comment_on_reject boolean

Critical rules:
- Approval must be 100% dynamic and DB-driven.
- Do not hardcode step count.
- Add/remove/reorder steps from DB.
- Controller must be thin and use ApprovalFlowService/ApprovalStepService.
```

---

# 16. Prompt 11 — Document Approval Action Controllers and Requests

```text
Read the latest 06-approval-workflow-design, 05-permission-access-control-design, 09-audit-log-activity-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement:
- DocumentApprovalController
- PendingApprovalController
- DocumentApprovalActionController
- ApproveDocumentRequest
- RejectDocumentRequest

Routes/actions:
- POST /admin/dms/documents/{document}/submit-approval
- GET /admin/dms/approvals/pending
- POST /admin/dms/documents/{document}/approve
- POST /admin/dms/documents/{document}/reject

Rules:
- Use DocumentApprovalService for submit.
- Use ApprovalActionService for approve/reject.
- Use DocumentPolicy@submitForApproval and DocumentPolicy@approve.
- Only current assigned approver can approve/reject.
- Rejection comment is required if current step requires_comment_on_reject = true.
- Controller must be thin.
- Use DB transactions in services.
- Preserve approval history.

ApproveDocumentRequest:
- comment nullable string max 1000

RejectDocumentRequest:
- comment required string max 1000, but conditional behavior may be checked in service based on step config.
```

---

# 17. Prompt 12 — Access Control Controllers

```text
Read 05-permission-access-control-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement:
- UserCompanyAccessController
- UserDepartmentAccessController
- DocumentAccessRuleController
- StoreDocumentAccessRuleRequest

Routes:
- GET/POST /admin/dms/users/{user}/company-access
- GET/POST /admin/dms/users/{user}/department-access
- GET/POST /admin/dms/documents/{document}/permissions
- DELETE /admin/dms/documents/{document}/permissions/{rule}

Rules:
- Use UserCompanyAccessService.
- Use UserDepartmentAccessService.
- Use DocumentAccessRuleService.
- Only authorized users can manage access.
- DocumentAccessRuleController must use DocumentPolicy@managePermissions.
- Audit permission/access changes.

Document access rule validation:
- target_type required in user,role
- user_id required_if target_type user nullable exists users id
- role_id required_if target_type role nullable exists roles id
- access_type required valid AccessType enum
- can_access required boolean
```

---

# 18. Prompt 13 — Audit Log Controllers

```text
Read 09-audit-log-activity-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement:
- AuditLogController
- DocumentTimelineController

AuditLogController@index:
- Shows global audit log page/list.
- Supports filters: event, user_id, auditable_type, auditable_id, date_from, date_to.
- Applies audit log access control.

DocumentTimelineController@index:
- Shows audit timeline for a specific document.
- Uses DocumentPolicy@view.
- Returns Blade partial or JSON.

Rules:
- Normal users cannot edit/delete audit logs.
- Do not expose sensitive old/new values to unauthorized users.
- Controller must use repository/service if query is complex.
```

---

# 19. Prompt 14 — System Settings Controller and Request

```text
Read 11-system-settings-configuration-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement SystemSettingController and UpdateSystemSettingRequest.

Routes:
- GET /admin/dms/settings
- PUT /admin/dms/settings

Rules:
- Only super admin or users with system_settings.manage can update settings.
- Use SettingsService.
- Validate values based on setting value_type.
- Do not expose private settings to frontend.
- Audit setting changes.
- Clear settings cache after update.

Do not store secrets in system_settings.
```

---

# 20. Prompt 15 — OCR Controllers Later

```text
Read 10-ocr-processing-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement OCR controllers later only:
- DocumentOcrController@status
- DocumentOcrController@reprocess

Routes:
- GET /admin/dms/documents/{document}/ocr-status
- POST /admin/dms/documents/{document}/ocr-reprocess

Rules:
- Use DocumentPolicy@view for status.
- Use DocumentPolicy@update or specific permission for reprocess.
- Use DocumentOcrService.
- OCR must be queued.
- Do not expose OCR text to unauthorized users.
```

---

# 21. Prompt 16 — Route Registration Prompt

```text
Read 12-api-route-design and 15-implementation-rules-and-coding-standard.

Register DMS routes under prefix /admin/dms and name prefix dms.

Use auth middleware.
Group routes logically:
- dashboard
- companies
- departments
- categories
- metadata fields
- documents
- document files
- approval setup
- approval actions
- search APIs
- access control
- audit logs
- settings
- OCR later

Rules:
- Use route model binding.
- Use named routes.
- Use permission middleware for broad access where appropriate.
- Use policies inside controllers for model-specific actions.
- Do not expose direct file paths.
```

---

# 22. Prompt 17 — HTTP Layer Review Prompt

Use this after Codex creates controllers, requests, and policies.

```text
Review all DMS controllers, Form Requests, and Policies against:
- 05-permission-access-control-design
- 06-approval-workflow-design
- 08-file-storage-versioning-design
- 12-api-route-design
- 15-implementation-rules-and-coding-standard

Check:
1. Controllers are thin.
2. No business logic is inside controllers.
3. Form Requests validate properly.
4. Policies are used for model-specific actions.
5. DocumentPolicy delegates to DocumentAccessService.
6. Document list/search queries are access-scoped.
7. Preview/download never expose file paths.
8. Approval controllers do not hardcode steps.
9. Rejection comment follows dynamic step config.
10. Unauthorized users cannot access sensitive actions.

Return a checklist and apply fixes if needed.
```

---

# 23. Recommended Implementation Order

```text
1. Company controller/request/policy
2. Department controller/request/policy
3. Category controller/request/policy
4. Metadata controller/request/API
5. Document controller/request/policy
6. File preview/download/version controllers
7. Search controller/request
8. Approval flow/step controllers/requests
9. Approval action controllers/requests
10. Access control controllers/requests
11. Audit log controllers
12. Settings controller/request
13. OCR controllers later
14. Route registration review
```

---

## 24. Summary

This document gives Codex exact prompts for building the HTTP layer safely.

The most important rule is:

```text
Controllers must coordinate; they must not own DMS business logic.
```

If Codex follows this document, validation, authorization, and request handling will remain clean and secure.
