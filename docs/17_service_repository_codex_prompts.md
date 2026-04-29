# Document Management System (DMS) — Service & Repository Codex Prompts

## 1. Purpose

This document contains ready-to-use Codex prompts for implementing services and repositories for the DMS.

The goal is to keep business logic out of controllers and make the codebase modular, testable, and maintainable.

Codex should use this document with:

```text
03-module-breakdown
04-database-design
05-permission-access-control-design
06-approval-workflow-design
07-search-architecture-design
08-file-storage-versioning-design
09-audit-log-activity-design
10-ocr-processing-design
11-system-settings-configuration-design
12-api-route-design
15-implementation-rules-and-coding-standard
16-database-migration-codex-prompts
```

---

## 2. Global Service & Repository Rules

Codex must follow these rules:

```text
1. Controllers must stay thin.
2. Business logic must live in services.
3. Complex queries must live in repositories.
4. Use DB transactions for multi-table writes.
5. Use policies/access services for authorization.
6. Dispatch events for audit/search/notification side effects.
7. Do not hardcode approval logic.
8. Do not expose private file paths.
9. Do not return unscoped document queries.
10. Keep classes small and focused.
```

---

## 3. Recommended Service/Repository List

```text
SettingsService
CompanyService
CompanyRepository
DepartmentService
DepartmentRepository
DocumentCategoryService
DocumentCategoryRepository
MetadataFieldService
MetadataValueService
DocumentCreateService
DocumentUpdateService
DocumentRepository
DocumentFileStorageService
DocumentVersionService
DocumentFileDownloadService
DocumentFilePreviewService
FileChecksumService
DocumentAccessService
DocumentSearchService
DocumentSearchRepository
ApprovalFlowResolverService
ApprovalFlowService
ApprovalStepService
DocumentApprovalService
ApprovalActionService
AuditLogService
DocumentOcrService
```

---

# 4. Prompt 01 — SettingsService

```text
Read 11-system-settings-configuration-design and 15-implementation-rules-and-coding-standard.

Implement SettingsService under app/Domains/Settings/Services.

Methods:
- get(string $key, mixed $default = null): mixed
- set(string $key, mixed $value, ?string $type = null): void
- getGroup(string $group): array
- forgetCache(?string $key = null): void
- isEnabled(string $key): bool

Requirements:
- Read settings from system_settings table.
- Cache all settings for performance.
- Cast value based on value_type.
- Clear cache after update.
- Do not store secrets in system_settings.
- Dispatch/audit setting update if AuditLogService exists.

Do not build UI in this task.
```

---

# 5. Prompt 02 — CompanyService and CompanyRepository

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement CompanyRepository and CompanyService.

CompanyRepository responsibilities:
- paginate(array $filters = [])
- findById(int $id): Company
- activeList()
- query with filters: name, code, status

CompanyService responsibilities:
- create(array $data, User $user): Company
- update(Company $company, array $data, User $user): Company
- delete(Company $company, User $user): bool

Rules:
- Use created_by and updated_by.
- Keep controller thin.
- Use soft delete.
- Dispatch audit event if available.
```

---

# 6. Prompt 03 — DepartmentService and DepartmentRepository

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, and 15-implementation-rules-and-coding-standard.

Implement DepartmentRepository and DepartmentService.

DepartmentRepository responsibilities:
- paginate(array $filters = [])
- findById(int $id): Department
- activeListByCompany(int $companyId)
- query filters: company_id, name, code, status

DepartmentService responsibilities:
- create(array $data, User $user): Department
- update(Department $department, array $data, User $user): Department
- delete(Department $department, User $user): bool

Rules:
- Department belongs to company.
- Prevent duplicate code within same company.
- Use created_by and updated_by.
- Use soft delete.
```

---

# 7. Prompt 04 — DocumentCategoryService and Repository

```text
Read 03-module-breakdown, 04-database-design, 06-approval-workflow-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentCategoryRepository and DocumentCategoryService.

Repository responsibilities:
- paginate(array $filters = [])
- findById(int $id): DocumentCategory
- activeList(array $filters = [])
- childrenOf(?int $parentId)
- query filters: company_id, department_id, parent_id, requires_approval, status

Service responsibilities:
- create(array $data, User $user): DocumentCategory
- update(DocumentCategory $category, array $data, User $user): DocumentCategory
- delete(DocumentCategory $category, User $user): bool

Rules:
- Support parent-child categories.
- Support company/department-specific or global categories.
- requires_approval controls approval requirement.
- Prevent category from being parent of itself.
- Do not put approval workflow logic here; only store approval requirement flag.
```

---

# 8. Prompt 05 — Metadata Services

```text
Read 03-module-breakdown, 04-database-design, 13-ui-ux-screen-flow-design, and 15-implementation-rules-and-coding-standard.

Implement MetadataFieldService and MetadataValueService.

MetadataFieldService methods:
- create(DocumentCategory $category, array $data, User $user): MetadataField
- update(MetadataField $field, array $data, User $user): MetadataField
- delete(MetadataField $field, User $user): bool
- getActiveFieldsByCategory(int $categoryId)

MetadataValueService methods:
- saveValues(Document $document, array $metadataValues): void
- validateRequiredFields(DocumentCategory $category, array $metadataValues): void
- normalizeValueByFieldType(MetadataField $field, mixed $value): array

Rules:
- Field key must be unique per category.
- Store date values in value_date.
- Store numeric values in value_number.
- Store general values in value.
- Respect field_type.
- Do not hardcode category-specific fields.
```

---

# 9. Prompt 06 — DocumentRepository

```text
Read 04-database-design, 05-permission-access-control-design, 07-search-architecture-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentRepository under app/Domains/Document/Repositories.

Methods:
- baseQuery(): Builder
- findById(int $id): Document
- findByUuid(string $uuid): Document
- paginateForUser(User $user, array $filters = [])
- latestDocumentsForUser(User $user, int $limit = 10)
- getExpiringDocumentsForUser(User $user, int $days = 30)

Rules:
- Use eager loading for company, department, category, uploadedBy, latestFile.
- Never return unscoped document list.
- Use DocumentAccessService or equivalent scope for user-facing list queries.
- Keep complex filters out of controller.
```

---

# 10. Prompt 07 — DocumentCreateService

```text
Read 04-database-design, 06-approval-workflow-design, 08-file-storage-versioning-design, 09-audit-log-activity-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentCreateService.

Responsibilities:
- Create document record.
- Save common fields.
- Save custom metadata values using MetadataValueService.
- Save tags.
- Store uploaded file using DocumentVersionService or DocumentFileStorageService.
- Set initial document status based on action and category approval requirement.
- Dispatch audit event.

Method:
- create(array $data, UploadedFile $file, User $user): Document

Rules:
- Use DB transaction.
- Generate UUID.
- Use SettingsService for default confidentiality level.
- If save as draft → status draft.
- If submit and category requires approval → status pending_approval and call approval service.
- If submit and category does not require approval → status approved.
- Do not hardcode approval steps.
- Do not store file in public folder.
```

---

# 11. Prompt 08 — DocumentUpdateService

```text
Read 04-database-design, 05-permission-access-control-design, 08-file-storage-versioning-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentUpdateService.

Responsibilities:
- Update document common fields.
- Update metadata values.
- Update tags.
- Enforce status rules.
- Dispatch audit event with old/new values.

Method:
- update(Document $document, array $data, User $user): Document

Rules:
- Use DB transaction.
- Approved documents should not be directly changed unless business rule allows metadata correction by admin.
- Pending approval documents should not be edited unless approval is cancelled/reset.
- Rejected/draft documents can be edited by authorized users.
- Use DocumentPolicy/DocumentAccessService before calling service.
```

---

# 12. Prompt 09 — File Storage Services

```text
Read 08-file-storage-versioning-design and 15-implementation-rules-and-coding-standard.

Implement these services:

1. DocumentFileStorageService
2. DocumentVersionService
3. FileChecksumService

DocumentFileStorageService:
- store(Document $document, UploadedFile $file, int $versionNo, User $user): array
- generateStoredName(Document $document, UploadedFile $file, int $versionNo): string
- buildStoragePath(Document $document, int $versionNo): string

DocumentVersionService:
- createInitialVersion(Document $document, UploadedFile $file, User $user): DocumentFile
- uploadNewVersion(Document $document, UploadedFile $file, User $user, ?string $note = null): DocumentFile
- getNextVersionNo(Document $document): int
- markPreviousVersionsNotLatest(Document $document): void

FileChecksumService:
- generate(UploadedFile|string $file): string
- isDuplicateForDocument(Document $document, string $checksum): bool

Rules:
- Use private storage disk.
- Do not expose file path.
- Only one is_latest version.
- Approved documents are not overwritten.
- New upload creates new version.
- Use checksum.
- Dispatch audit and OCR/search events if available.
```

---

# 13. Prompt 10 — Preview and Download Services

```text
Read 05-permission-access-control-design, 08-file-storage-versioning-design, 09-audit-log-activity-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentFileDownloadService and DocumentFilePreviewService.

DocumentFileDownloadService methods:
- downloadLatest(Document $document, User $user): Symfony Response
- downloadFile(Document $document, DocumentFile $file, User $user): Symfony Response

DocumentFilePreviewService methods:
- previewLatest(Document $document, User $user): Symfony Response
- previewFile(Document $document, DocumentFile $file, User $user): Symfony Response

Rules:
- Check permission using DocumentAccessService/Policy.
- Never expose file_path to frontend.
- Verify file belongs to document.
- Log preview/download actions.
- Return proper response headers.
- Handle missing file gracefully.
```

---

# 14. Prompt 11 — DocumentAccessService

```text
Read 05-permission-access-control-design carefully and 15-implementation-rules-and-coding-standard.

Implement DocumentAccessService under app/Domains/AccessControl/Services.

Methods:
- canView(User $user, Document $document): bool
- canDownload(User $user, Document $document): bool
- canEdit(User $user, Document $document): bool
- canDelete(User $user, Document $document): bool
- canArchive(User $user, Document $document): bool
- canApprove(User $user, Document $document): bool
- canManagePermissions(User $user, Document $document): bool
- applyVisibleDocumentsScope(Builder $query, User $user): Builder

Rules:
- Super admin bypasses normal restrictions.
- Explicit deny rule has priority over allow.
- Explicit allow rule should be considered.
- Check Spatie permissions.
- Check company access.
- Check department access.
- Check confidentiality level.
- Check current approval assignment for approvers.
- Do not return unscoped document queries.
```

---

# 15. Prompt 12 — DocumentSearchService and Repository

```text
Read 07-search-architecture-design, 05-permission-access-control-design, and 15-implementation-rules-and-coding-standard.

Implement DocumentSearchRepository and DocumentSearchService for MySQL MVP search.

Search filters:
- keyword
- company_id
- department_id
- category_id
- status
- confidentiality_level
- document_date_from
- document_date_to
- expiry_date_from
- expiry_date_to
- uploaded_by
- tags
- metadata fields

Rules:
- Apply DocumentAccessService visible scope first or safely inside query.
- Search title, document_no, summary, remarks.
- Search metadata values.
- Search tags.
- Return paginated results.
- Do not implement Meilisearch yet.
- Do not return unauthorized documents.
```

---

# 16. Prompt 13 — Approval Services

```text
Read the latest 06-approval-workflow-design carefully. Approval must be 100% dynamic and DB-driven.

Implement these services:

1. ApprovalFlowResolverService
2. ApprovalFlowService
3. ApprovalStepService
4. DocumentApprovalService
5. ApprovalActionService

ApprovalFlowResolverService:
- resolveForDocument(Document $document): ?ApprovalFlow
- Apply priority:
  1. company_id + department_id + category_id
  2. company_id + category_id
  3. category_id only
  4. default flow is_default = true

DocumentApprovalService:
- submit(Document $document, User $user): void
- createApprovalCycle(Document $document, ApprovalFlow $flow): void
- getCurrentPendingStep(Document $document): ?DocumentApproval

ApprovalActionService:
- approve(Document $document, User $user, ?string $comment = null): void
- reject(Document $document, User $user, string $comment): void
- moveToNextStep(Document $document): void
- finalizeApproval(Document $document, User $user): void

Rules:
- No approval step is hardcoded.
- Category requires_approval controls whether approval is needed.
- Approval steps come from approval_steps table.
- Flow matching is dynamic.
- Support 0, 1, or N steps.
- Support user or role approver.
- Only current pending step can be approved/rejected.
- Rejection comment required if step requires_comment_on_reject = true.
- Preserve old approval history.
- Re-submission creates new approval_cycle.
- Use DB transaction.
- Dispatch audit/notification events if available.
```

---

# 17. Prompt 14 — AuditLogService

```text
Read 09-audit-log-activity-design and 15-implementation-rules-and-coding-standard.

Implement AuditLogService under app/Domains/AuditLog/Services.

Methods:
- log(string $event, ?Model $auditable = null, array $oldValues = [], array $newValues = [], array $metadata = []): AuditLog
- logDocumentAction(string $event, Document $document, array $metadata = []): AuditLog
- logFileAction(string $event, DocumentFile $file, array $metadata = []): AuditLog
- logPermissionChange(Model $target, array $oldValues, array $newValues): AuditLog

Rules:
- Include authenticated user if available.
- Include IP address and user agent if request exists.
- Do not store huge OCR text in audit logs.
- Audit logs are append-only.
- Do not allow normal edit/delete.
```

---

# 18. Prompt 15 — User Access Mapping Services

```text
Read 05-permission-access-control-design and 15-implementation-rules-and-coding-standard.

Implement UserCompanyAccessService and UserDepartmentAccessService.

UserCompanyAccessService:
- syncCompanies(User $user, array $companyIds, User $actor): void
- getCompanyIds(User $user): array

UserDepartmentAccessService:
- syncDepartments(User $user, array $departmentIds, User $actor): void
- getDepartmentIds(User $user): array

Rules:
- Prevent duplicate access rows.
- Use sync behavior.
- Audit changes.
- Validate that departments belong to accessible companies if needed.
```

---

# 19. Prompt 16 — DocumentAccessRuleService

```text
Read 05-permission-access-control-design and 15-implementation-rules-and-coding-standard.

Implement DocumentAccessRuleService.

Methods:
- createRule(Document $document, array $data, User $actor): DocumentAccessRule
- deleteRule(DocumentAccessRule $rule, User $actor): bool
- getRulesForDocument(Document $document)
- hasExplicitDeny(User $user, Document $document, string $accessType): bool
- hasExplicitAllow(User $user, Document $document, string $accessType): bool

Rules:
- Rule can target user_id or role_id.
- Either user_id or role_id must be provided.
- Explicit deny takes priority.
- Audit permission changes.
```

---

# 20. Prompt 17 — OCR Service Later

```text
Read 10-ocr-processing-design and 15-implementation-rules-and-coding-standard.

Implement DocumentOcrService for later phase only.

Methods:
- createPendingOcrRecord(DocumentFile $file): DocumentOcrText
- process(DocumentFile $file): void
- markProcessing(DocumentOcrText $ocr): void
- markCompleted(DocumentOcrText $ocr, string $text): void
- markFailed(DocumentOcrText $ocr, string $error): void
- reprocess(DocumentOcrText $ocr, User $user): void

Rules:
- OCR must be queue-based.
- Upload must not wait for OCR.
- Link OCR to document_file_id.
- Store OCR text separately.
- Do not expose OCR text without permission.
- Do not store OCR text in audit logs.
```

---

# 21. Prompt 18 — Service Review Prompt

Use after Codex implements service layer.

```text
Review all DMS services and repositories against:
- 03-module-breakdown
- 05-permission-access-control-design
- 06-approval-workflow-design
- 07-search-architecture-design
- 08-file-storage-versioning-design
- 09-audit-log-activity-design
- 15-implementation-rules-and-coding-standard

Check:
1. Controllers are thin.
2. Business logic is inside services.
3. Complex queries are inside repositories.
4. Document queries are access-scoped.
5. Approval is fully dynamic and DB-driven.
6. Approved files are not overwritten.
7. File paths are not exposed.
8. Transactions are used for multi-table writes.
9. Audit events/logs exist for sensitive actions.
10. No hardcoded approval steps or magic statuses.

Return a checklist and apply fixes if needed.
```

---

# 22. Recommended Implementation Order

```text
1. SettingsService
2. CompanyService + Repository
3. DepartmentService + Repository
4. DocumentCategoryService + Repository
5. MetadataFieldService + MetadataValueService
6. DocumentRepository
7. DocumentCreateService
8. DocumentUpdateService
9. File storage/versioning services
10. Preview/download services
11. DocumentAccessService
12. DocumentSearchService + Repository
13. AuditLogService
14. Approval services
15. Access rule services
16. OCR service later
```

---

## 23. Summary

This document gives Codex exact prompts for implementing service and repository layers.

The most important rule is:

```text
Codex must not place enterprise DMS business logic inside controllers. Services and repositories must own the complexity.
```
