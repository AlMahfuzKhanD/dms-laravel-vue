# Document Management System (DMS) — Database Migration Codex Prompts

## 1. Purpose

This document contains ready-to-use Codex prompts for generating Laravel migrations for the DMS database.

Codex should use this document together with:

```text
04-database-design
05-permission-access-control-design
06-approval-workflow-design
08-file-storage-versioning-design
10-ocr-processing-design
11-system-settings-configuration-design
15-implementation-rules-and-coding-standard
```

The goal is to generate migrations step-by-step safely, not all at once.

---

## 2. Global Migration Rules for Codex

Use these rules in every migration task:

```text
1. Use Laravel migrations.
2. Use unsigned big integer IDs by default.
3. Use foreign keys where safe and practical.
4. Add indexes for searchable/filterable fields.
5. Use soft deletes for important business tables.
6. Do not store uploaded files in database.
7. Keep document records and document files separate.
8. Keep metadata definitions and metadata values separate.
9. Keep approval setup and approval execution separate.
10. Add created_by and updated_by where useful.
11. Use nullable foreign keys when global/fallback configuration is needed.
12. Use safe cascade/restrict behavior.
13. Do not add undocumented columns unless necessary and explained.
```

---

## 3. Recommended Migration Order

```text
1. system_settings
2. companies
3. departments
4. document_categories
5. metadata_fields
6. documents
7. document_files
8. document_metadata_values
9. document_tags
10. document_tag_links
11. user_company_accesses
12. user_department_accesses
13. document_access_rules
14. approval_flows
15. approval_steps
16. document_approvals
17. audit_logs
18. document_ocr_texts
```

Important:

```text
Spatie Permission migrations must be installed before document_access_rules, approval_steps, and role-based approval logic because those may reference roles table.
```

---

# 4. Prompt 01 — System Settings Migration

## Codex Prompt

```text
Read 11-system-settings-configuration-design and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for system_settings table.

Columns:
- id
- key unique string 150
- value nullable text
- value_type string 50 default string
- group nullable string 100
- label nullable string 150
- description nullable text
- is_public boolean default false
- is_editable boolean default true
- timestamps

Add indexes for key, group, is_public, and is_editable.

Also create SystemSetting model under app/Domains/Settings/Models with fillable fields and casts for is_public and is_editable.

Do not build UI yet.
```

---

# 5. Prompt 02 — Companies Migration

## Codex Prompt

```text
Read 04-database-design and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for companies table.

Columns:
- id
- name string 150 required
- code string 50 nullable unique
- address text nullable
- phone string 50 nullable
- email string 100 nullable
- status tinyInteger default 1
- created_by nullable foreignId constrained users nullOnDelete
- updated_by nullable foreignId constrained users nullOnDelete
- timestamps
- softDeletes

Indexes:
- name
- status
- code unique

Create Company model under app/Domains/Company/Models with SoftDeletes, fillable fields, and relationships for departments and documents.
```

---

# 6. Prompt 03 — Departments Migration

## Codex Prompt

```text
Read 04-database-design and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for departments table.

Columns:
- id
- company_id foreignId constrained companies restrictOnDelete
- name string 150 required
- code string 50 nullable
- status tinyInteger default 1
- created_by nullable foreignId constrained users nullOnDelete
- updated_by nullable foreignId constrained users nullOnDelete
- timestamps
- softDeletes

Indexes:
- company_id
- name
- status
- unique company_id + code

Create Department model under app/Domains/Department/Models with SoftDeletes, fillable fields, and relationships to Company and Document.
```

---

# 7. Prompt 04 — Document Categories Migration

## Codex Prompt

```text
Read 04-database-design, 06-approval-workflow-design, and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for document_categories table.

Columns:
- id
- parent_id nullable foreignId references document_categories nullOnDelete
- company_id nullable foreignId constrained companies nullOnDelete
- department_id nullable foreignId constrained departments nullOnDelete
- name string 150 required
- code string 50 nullable
- description nullable text
- requires_approval boolean default false
- status tinyInteger default 1
- created_by nullable foreignId constrained users nullOnDelete
- updated_by nullable foreignId constrained users nullOnDelete
- timestamps
- softDeletes

Indexes:
- parent_id
- company_id
- department_id
- status
- requires_approval
- unique company_id + department_id + code

Create DocumentCategory model under app/Domains/DocumentCategory/Models with SoftDeletes and relationships: parent, children, company, department, metadataFields, documents.
```

---

# 8. Prompt 05 — Metadata Fields Migration

## Codex Prompt

```text
Read 04-database-design and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for metadata_fields table.

Columns:
- id
- category_id foreignId constrained document_categories restrictOnDelete
- label string 150 required
- field_key string 100 required
- field_type string 50 required
- placeholder nullable string 150
- options nullable json
- is_required boolean default false
- is_searchable boolean default true
- sort_order integer default 0
- status tinyInteger default 1
- created_by nullable foreignId constrained users nullOnDelete
- updated_by nullable foreignId constrained users nullOnDelete
- timestamps
- softDeletes

Indexes:
- category_id
- field_key
- field_type
- is_searchable
- status
- unique category_id + field_key

Create MetadataField model under app/Domains/Metadata/Models with SoftDeletes, fillable fields, options cast as array, boolean casts, and relationship to DocumentCategory and DocumentMetadataValue.
```

---

# 9. Prompt 06 — Documents Migration

## Codex Prompt

```text
Read 04-database-design, 05-permission-access-control-design, 06-approval-workflow-design, 08-file-storage-versioning-design, and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for documents table.

Columns:
- id
- uuid char 36 unique required
- company_id foreignId constrained companies restrictOnDelete
- department_id foreignId constrained departments restrictOnDelete
- category_id foreignId constrained document_categories restrictOnDelete
- title string 255 required
- document_no string 100 nullable
- document_date date nullable
- expiry_date date nullable
- confidentiality_level string 50 default internal
- status string 50 default draft
- summary nullable text
- remarks nullable text
- uploaded_by foreignId constrained users restrictOnDelete
- approved_by nullable foreignId constrained users nullOnDelete
- approved_at nullable timestamp
- archived_at nullable timestamp
- timestamps
- softDeletes

Indexes:
- uuid unique
- company_id
- department_id
- category_id
- uploaded_by
- approved_by
- status
- confidentiality_level
- document_no
- document_date
- expiry_date
- created_at
- fulltext title, document_no, summary if supported by database

Create Document model under app/Domains/Document/Models with SoftDeletes, fillable fields, casts for dates, and relationships: company, department, category, uploadedBy, approvedBy, files, latestFile, metadataValues, approvals, accessRules, ocrTexts, tags.
```

---

# 10. Prompt 07 — Document Files Migration

## Codex Prompt

```text
Read 04-database-design, 08-file-storage-versioning-design, and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for document_files table.

Columns:
- id
- document_id foreignId constrained documents cascadeOnDelete
- version_no integer default 1
- original_name string 255 required
- stored_name string 255 required
- file_path string 500 required
- disk string 50 default private
- file_extension string 20 nullable
- mime_type string 100 nullable
- file_size unsignedBigInteger default 0
- checksum string 128 nullable
- uploaded_by foreignId constrained users restrictOnDelete
- is_latest boolean default true
- timestamps
- softDeletes

Indexes:
- document_id
- uploaded_by
- version_no
- is_latest
- mime_type
- checksum
- unique document_id + version_no

Create DocumentFile model under app/Domains/Document/Models with SoftDeletes, boolean casts, and relationships to Document and uploadedBy user.
```

---

# 11. Prompt 08 — Document Metadata Values Migration

## Codex Prompt

```text
Read 04-database-design and 15-implementation-rules-and-coding-standard.

Create a Laravel migration for document_metadata_values table.

Columns:
- id
- document_id foreignId constrained documents cascadeOnDelete
- metadata_field_id foreignId constrained metadata_fields restrictOnDelete
- value nullable text
- value_date nullable date
- value_number nullable decimal 18,4
- timestamps

Indexes:
- document_id
- metadata_field_id
- value_date
- value_number
- unique document_id + metadata_field_id
- fulltext value if supported by database

Create DocumentMetadataValue model under app/Domains/Metadata/Models with relationships to Document and MetadataField.
```

---

# 12. Prompt 09 — Document Tags Migrations

## Codex Prompt

```text
Read 04-database-design and 15-implementation-rules-and-coding-standard.

Create Laravel migrations for document_tags and document_tag_links.

Table document_tags:
- id
- name string 100 required
- slug string 120 unique required
- timestamps
Indexes: name, slug unique

Table document_tag_links:
- document_id foreignId constrained documents cascadeOnDelete
- document_tag_id foreignId constrained document_tags cascadeOnDelete
- timestamps nullable/created_at only if preferred
- primary key document_id + document_tag_id
- index document_tag_id

Create DocumentTag model under app/Domains/Document/Models with relationship belongsToMany Document.
Add tags relationship to Document model.
```

---

# 13. Prompt 10 — User Company & Department Access Migrations

## Codex Prompt

```text
Read 05-permission-access-control-design and 15-implementation-rules-and-coding-standard.

Create Laravel migrations for user_company_accesses and user_department_accesses.

Table user_company_accesses:
- id
- user_id foreignId constrained users cascadeOnDelete
- company_id foreignId constrained companies cascadeOnDelete
- created_by nullable foreignId constrained users nullOnDelete
- timestamps
Indexes: user_id, company_id, unique user_id + company_id

Table user_department_accesses:
- id
- user_id foreignId constrained users cascadeOnDelete
- department_id foreignId constrained departments cascadeOnDelete
- created_by nullable foreignId constrained users nullOnDelete
- timestamps
Indexes: user_id, department_id, unique user_id + department_id

Create models UserCompanyAccess and UserDepartmentAccess under app/Domains/AccessControl/Models with relationships to User, Company, and Department.
```

---

# 14. Prompt 11 — Document Access Rules Migration

## Codex Prompt

```text
Read 05-permission-access-control-design and 15-implementation-rules-and-coding-standard.

Create Laravel migration for document_access_rules table.

Important: Spatie Permission migrations must already exist because this table may reference roles.

Columns:
- id
- document_id foreignId constrained documents cascadeOnDelete
- user_id nullable foreignId constrained users cascadeOnDelete
- role_id nullable foreignId constrained roles cascadeOnDelete
- access_type string 50 required
- can_access boolean default true
- created_by nullable foreignId constrained users nullOnDelete
- timestamps

Indexes:
- document_id
- user_id
- role_id
- access_type
- can_access

Add a validation note in comments or model-level rules: either user_id or role_id should be provided.

Create DocumentAccessRule model under app/Domains/AccessControl/Models with boolean cast and relationships to Document, User, Role, and createdBy.
```

---

# 15. Prompt 12 — Approval Flows Migration

## Codex Prompt

```text
Read 06-approval-workflow-design carefully and 15-implementation-rules-and-coding-standard.

Create Laravel migration for approval_flows table.

Columns:
- id
- company_id nullable foreignId constrained companies nullOnDelete
- department_id nullable foreignId constrained departments nullOnDelete
- category_id foreignId constrained document_categories restrictOnDelete
- name string 150 required
- is_default boolean default false
- auto_approve_if_no_flow boolean default false
- status tinyInteger default 1
- created_by nullable foreignId constrained users nullOnDelete
- updated_by nullable foreignId constrained users nullOnDelete
- timestamps
- softDeletes

Indexes:
- company_id
- department_id
- category_id
- is_default
- auto_approve_if_no_flow
- status

Create ApprovalFlow model under app/Domains/Approval/Models with SoftDeletes, boolean casts, and relationships to company, department, category, steps, createdBy, updatedBy.

Important: approval must be dynamic and DB-driven. Do not hardcode steps.
```

---

# 16. Prompt 13 — Approval Steps Migration

## Codex Prompt

```text
Read 06-approval-workflow-design carefully and 15-implementation-rules-and-coding-standard.

Create Laravel migration for approval_steps table.

Important: Spatie Permission migrations must already exist because this table may reference roles.

Columns:
- id
- approval_flow_id foreignId constrained approval_flows cascadeOnDelete
- step_order integer required
- step_name nullable string 150
- approver_type string 50 required
- approver_user_id nullable foreignId constrained users nullOnDelete
- approver_role_id nullable foreignId constrained roles nullOnDelete
- is_required boolean default true
- can_edit_document boolean default false
- can_download_document boolean default false
- requires_comment_on_reject boolean default true
- timestamps

Indexes:
- approval_flow_id
- step_order
- approver_type
- approver_user_id
- approver_role_id
- is_required
- unique approval_flow_id + step_order

Create ApprovalStep model under app/Domains/Approval/Models with boolean casts and relationships to ApprovalFlow, approverUser, approverRole.

Important: number of steps must be unlimited and controlled by rows in approval_steps.
```

---

# 17. Prompt 14 — Document Approvals Migration

## Codex Prompt

```text
Read 06-approval-workflow-design carefully and 15-implementation-rules-and-coding-standard.

Create Laravel migration for document_approvals table.

Columns:
- id
- document_id foreignId constrained documents cascadeOnDelete
- approval_flow_id foreignId constrained approval_flows restrictOnDelete
- approval_step_id foreignId constrained approval_steps restrictOnDelete
- approval_cycle integer default 1
- step_order integer required
- assigned_to_user_id nullable foreignId constrained users nullOnDelete
- assigned_to_role_id nullable foreignId constrained roles nullOnDelete
- status string 50 default pending
- action_by nullable foreignId constrained users nullOnDelete
- action_at nullable timestamp
- comments nullable text
- timestamps

Indexes:
- document_id
- approval_flow_id
- approval_step_id
- approval_cycle
- step_order
- assigned_to_user_id
- assigned_to_role_id
- status
- action_by

Create DocumentApproval model under app/Domains/Approval/Models with date cast for action_at and relationships to Document, ApprovalFlow, ApprovalStep, assignedToUser, assignedToRole, actionBy.

Important: approval history must be preserved. Re-submission should create a new approval_cycle.
```

---

# 18. Prompt 15 — Audit Logs Migration

## Codex Prompt

```text
Read 09-audit-log-activity-design and 15-implementation-rules-and-coding-standard.

Create Laravel migration for audit_logs table.

Columns:
- id
- user_id nullable foreignId constrained users nullOnDelete
- event string 100 required
- auditable_type nullable string 150
- auditable_id nullable unsignedBigInteger
- old_values nullable json
- new_values nullable json
- metadata nullable json
- ip_address nullable string 45
- user_agent nullable text
- created_at timestamp nullable

Indexes:
- user_id
- event
- auditable_type + auditable_id
- created_at

Create AuditLog model under app/Domains/AuditLog/Models with casts for old_values, new_values, metadata and relationship to User.

Do not add updated_at because audit logs are append-only.
```

---

# 19. Prompt 16 — Document OCR Texts Migration

## Codex Prompt

```text
Read 10-ocr-processing-design and 15-implementation-rules-and-coding-standard.

Create Laravel migration for document_ocr_texts table.

Columns:
- id
- document_id foreignId constrained documents cascadeOnDelete
- document_file_id nullable foreignId constrained document_files cascadeOnDelete
- text nullable longText
- ocr_status string 50 default pending
- error_message nullable text
- processed_at nullable timestamp
- timestamps

Indexes:
- document_id
- document_file_id
- ocr_status
- processed_at
- fulltext text if supported by database

Create DocumentOcrText model under app/Domains/Ocr/Models with date cast for processed_at and relationships to Document and DocumentFile.

Important: OCR must be version-aware through document_file_id.
```

---

# 20. Prompt 17 — Enum Classes

## Codex Prompt

```text
Read 04-database-design, 06-approval-workflow-design, 10-ocr-processing-design, and 15-implementation-rules-and-coding-standard.

Create enum classes under app/Shared/Enums:

1. DocumentStatus
- DRAFT = draft
- PENDING_APPROVAL = pending_approval
- APPROVED = approved
- REJECTED = rejected
- ARCHIVED = archived

2. ConfidentialityLevel
- PUBLIC = public
- INTERNAL = internal
- CONFIDENTIAL = confidential
- RESTRICTED = restricted

3. ApprovalStatus
- PENDING = pending
- APPROVED = approved
- REJECTED = rejected
- SKIPPED = skipped

4. MetadataFieldType
- TEXT = text
- TEXTAREA = textarea
- NUMBER = number
- DATE = date
- SELECT = select
- MULTI_SELECT = multi_select
- CHECKBOX = checkbox
- BOOLEAN = boolean

5. OcrStatus
- PENDING = pending
- PROCESSING = processing
- COMPLETED = completed
- FAILED = failed
- SKIPPED = skipped

6. AccessType
- VIEW = view
- DOWNLOAD = download
- EDIT = edit
- DELETE = delete
- APPROVE = approve
- MANAGE_PERMISSIONS = manage_permissions

Use PHP backed enums. Do not scatter magic strings.
```

---

# 21. Prompt 18 — Migration Review Prompt

Use this after Codex generates migrations.

## Codex Prompt

```text
Review all DMS migrations against these documents:
- 04-database-design
- 05-permission-access-control-design
- 06-approval-workflow-design
- 08-file-storage-versioning-design
- 09-audit-log-activity-design
- 10-ocr-processing-design
- 11-system-settings-configuration-design
- 15-implementation-rules-and-coding-standard

Check:
1. Missing columns
2. Wrong nullable fields
3. Missing indexes
4. Missing soft deletes
5. Wrong foreign key behavior
6. Missing approval dynamic fields
7. Missing OCR document_file_id
8. Missing audit metadata JSON
9. Missing system_settings fields
10. Any hardcoded/non-dynamic design issue

Return a clear checklist and apply fixes if needed.
```

---

# 22. Recommended Execution Plan

Do not run all prompts at once.

Recommended batches:

```text
Batch 1:
- system_settings
- companies
- departments
- document_categories

Batch 2:
- metadata_fields
- documents
- document_files
- document_metadata_values
- document_tags

Batch 3:
- access mapping
- document_access_rules

Batch 4:
- approval_flows
- approval_steps
- document_approvals

Batch 5:
- audit_logs
- document_ocr_texts
- enums
```

After each batch:

```text
php artisan migrate:fresh --seed
php artisan test
```

---

# 23. Final Migration Checklist

Before accepting migrations, confirm:

```text
✔ system_settings exists
✔ companies/departments support group structure
✔ categories support parent-child and approval flag
✔ metadata supports dynamic fields
✔ documents and document_files are separate
✔ document_files support versioning
✔ access mapping tables exist
✔ document access override exists
✔ approval is fully dynamic
✔ approval cycles preserve history
✔ audit_logs are append-only
✔ OCR is version-aware
✔ indexes exist for search/filter columns
✔ soft deletes exist for important business tables
✔ enum classes exist for fixed values
```

---

## 24. Summary

This document gives Codex exact migration prompts. The goal is to generate the database foundation carefully and consistently.

The most important rule is:

```text
Do not let Codex invent the database. Make Codex follow the documented schema step by step.
```
