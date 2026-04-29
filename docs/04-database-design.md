# Document Management System (DMS) — Database Design

## 1. Purpose

This document defines the initial database design for the Document Management System. It is written to guide Laravel migration, model, relationship, service, and repository implementation.

The design supports:

- Group company structure
- Multiple concerns/companies
- Departments
- Document categories
- Scanned/digital document storage
- Dynamic metadata
- Access control
- Approval matrix
- Version control
- Search indexing
- Audit logs
- Future OCR support

---

## 2. Database Design Principles

### 2.1 Multi-Company Ready

Every document must belong to a company/concern.

### 2.2 Department Aware

Every document should belong to a department under a company.

### 2.3 Category Driven

Document behavior such as metadata fields, approval requirement, and retention can be category-driven.

### 2.4 Flexible Metadata

Different document types require different fields. Therefore, the system must support dynamic metadata fields per category.

### 2.5 Secure by Design

Access rules must support role-based, company-based, department-based, category-based, and document-level permissions.

### 2.6 Search Optimized

Important searchable fields must be indexed in MySQL and later synced with Meilisearch.

### 2.7 Workflow Ready

Document status must support draft, pending approval, approved, rejected, and archived states.

### 2.8 Audit Friendly

Important actions must be recorded in audit logs.

---

## 3. Core Tables Overview

```text
users
companies
departments
document_categories
metadata_fields
documents
document_files
document_metadata_values
document_tags
document_tag_links
user_company_accesses
user_department_accesses
document_access_rules
approval_flows
approval_steps
document_approvals
audit_logs
document_ocr_texts
```

Spatie Laravel Permission will also create its own permission-related tables:

```text
roles
permissions
model_has_roles
model_has_permissions
role_has_permissions
```

---

# 4. Organization Tables

## 4.1 companies

Stores group concern/company/business unit information.

### Columns

```text
id BIGINT UNSIGNED PK
name VARCHAR(150) NOT NULL
code VARCHAR(50) UNIQUE NULL
address TEXT NULL
phone VARCHAR(50) NULL
email VARCHAR(100) NULL
status TINYINT DEFAULT 1
created_by BIGINT UNSIGNED NULL
updated_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Indexes

```text
INDEX status
INDEX name
UNIQUE code
```

### Relationships

```text
Company hasMany Department
Company hasMany Document
Company hasMany UserCompanyAccess
```

---

## 4.2 departments

Stores departments under each company.

### Columns

```text
id BIGINT UNSIGNED PK
company_id BIGINT UNSIGNED NOT NULL
name VARCHAR(150) NOT NULL
code VARCHAR(50) NULL
status TINYINT DEFAULT 1
created_by BIGINT UNSIGNED NULL
updated_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Indexes

```text
INDEX company_id
INDEX status
INDEX name
UNIQUE company_id, code
```

### Relationships

```text
Department belongsTo Company
Department hasMany Document
Department hasMany UserDepartmentAccess
```

---

# 5. Category & Metadata Tables

## 5.1 document_categories

Stores document classification. Supports parent-child category tree.

### Columns

```text
id BIGINT UNSIGNED PK
parent_id BIGINT UNSIGNED NULL
company_id BIGINT UNSIGNED NULL
department_id BIGINT UNSIGNED NULL
name VARCHAR(150) NOT NULL
code VARCHAR(50) NULL
description TEXT NULL
requires_approval BOOLEAN DEFAULT FALSE
status TINYINT DEFAULT 1
created_by BIGINT UNSIGNED NULL
updated_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Notes

- `company_id` nullable means category can be global.
- `department_id` nullable means category can be shared across departments.
- Parent-child category allows structure like:

```text
Legal Documents
 ├── Land Documents
 ├── License Documents
 └── Agreements
```

### Indexes

```text
INDEX parent_id
INDEX company_id
INDEX department_id
INDEX status
INDEX requires_approval
UNIQUE company_id, department_id, code
```

### Relationships

```text
DocumentCategory belongsTo parent DocumentCategory
DocumentCategory hasMany child DocumentCategory
DocumentCategory hasMany MetadataField
DocumentCategory hasMany Document
```

---

## 5.2 metadata_fields

Defines custom metadata fields for each document category.

### Columns

```text
id BIGINT UNSIGNED PK
category_id BIGINT UNSIGNED NOT NULL
label VARCHAR(150) NOT NULL
field_key VARCHAR(100) NOT NULL
field_type VARCHAR(50) NOT NULL
placeholder VARCHAR(150) NULL
options JSON NULL
is_required BOOLEAN DEFAULT FALSE
is_searchable BOOLEAN DEFAULT TRUE
sort_order INT DEFAULT 0
status TINYINT DEFAULT 1
created_by BIGINT UNSIGNED NULL
updated_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Supported field_type values

```text
text
textarea
number
date
select
multi_select
checkbox
boolean
```

### Indexes

```text
INDEX category_id
INDEX field_key
INDEX is_searchable
INDEX status
UNIQUE category_id, field_key
```

### Relationships

```text
MetadataField belongsTo DocumentCategory
MetadataField hasMany DocumentMetadataValue
```

---

# 6. Document Tables

## 6.1 documents

Stores the main document record and common metadata.

### Columns

```text
id BIGINT UNSIGNED PK
uuid CHAR(36) UNIQUE NOT NULL
company_id BIGINT UNSIGNED NOT NULL
department_id BIGINT UNSIGNED NOT NULL
category_id BIGINT UNSIGNED NOT NULL
title VARCHAR(255) NOT NULL
document_no VARCHAR(100) NULL
document_date DATE NULL
expiry_date DATE NULL
confidentiality_level VARCHAR(50) DEFAULT 'internal'
status VARCHAR(50) DEFAULT 'draft'
summary TEXT NULL
remarks TEXT NULL
uploaded_by BIGINT UNSIGNED NOT NULL
approved_by BIGINT UNSIGNED NULL
approved_at TIMESTAMP NULL
archived_at TIMESTAMP NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Status values

```text
draft
pending_approval
approved
rejected
archived
```

### Confidentiality levels

```text
public
internal
confidential
restricted
```

### Indexes

```text
UNIQUE uuid
INDEX company_id
INDEX department_id
INDEX category_id
INDEX uploaded_by
INDEX status
INDEX confidentiality_level
INDEX document_no
INDEX document_date
INDEX expiry_date
INDEX created_at
FULLTEXT title, document_no, summary
```

### Relationships

```text
Document belongsTo Company
Document belongsTo Department
Document belongsTo DocumentCategory
Document belongsTo User as uploadedBy
Document hasMany DocumentFile
Document hasMany DocumentMetadataValue
Document hasMany DocumentApproval
Document hasMany DocumentAccessRule
Document hasMany AuditLog morphMany
Document hasOne DocumentOcrText
Document belongsToMany DocumentTag
```

---

## 6.2 document_files

Stores uploaded document files and versions.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
version_no INT DEFAULT 1
original_name VARCHAR(255) NOT NULL
stored_name VARCHAR(255) NOT NULL
file_path VARCHAR(500) NOT NULL
disk VARCHAR(50) DEFAULT 'local'
file_extension VARCHAR(20) NULL
mime_type VARCHAR(100) NULL
file_size BIGINT UNSIGNED DEFAULT 0
checksum VARCHAR(128) NULL
uploaded_by BIGINT UNSIGNED NOT NULL
is_latest BOOLEAN DEFAULT TRUE
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Notes

- Every document must have at least one file.
- When a new version is uploaded, previous `is_latest` should become false.
- `checksum` helps detect duplicate or corrupted files.

### Indexes

```text
INDEX document_id
INDEX uploaded_by
INDEX version_no
INDEX is_latest
INDEX mime_type
INDEX checksum
UNIQUE document_id, version_no
```

### Relationships

```text
DocumentFile belongsTo Document
DocumentFile belongsTo User as uploadedBy
```

---

## 6.3 document_metadata_values

Stores custom metadata values for each document.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
metadata_field_id BIGINT UNSIGNED NOT NULL
value TEXT NULL
value_date DATE NULL
value_number DECIMAL(18,4) NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### Notes

- `value` stores general text/string/JSON values.
- `value_date` is used for date filtering.
- `value_number` is used for numeric filtering.
- This avoids inefficient casting during search/filter.

### Indexes

```text
INDEX document_id
INDEX metadata_field_id
INDEX value_date
INDEX value_number
FULLTEXT value
UNIQUE document_id, metadata_field_id
```

### Relationships

```text
DocumentMetadataValue belongsTo Document
DocumentMetadataValue belongsTo MetadataField
```

---

## 6.4 document_tags

Stores reusable tags.

### Columns

```text
id BIGINT UNSIGNED PK
name VARCHAR(100) NOT NULL
slug VARCHAR(120) UNIQUE NOT NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### Indexes

```text
UNIQUE slug
INDEX name
```

---

## 6.5 document_tag_links

Pivot table between documents and tags.

### Columns

```text
document_id BIGINT UNSIGNED NOT NULL
document_tag_id BIGINT UNSIGNED NOT NULL
created_at TIMESTAMP NULL
```

### Indexes

```text
PRIMARY document_id, document_tag_id
INDEX document_tag_id
```

---

# 7. Access Control Tables

## 7.1 user_company_accesses

Defines which companies a user can access.

### Columns

```text
id BIGINT UNSIGNED PK
user_id BIGINT UNSIGNED NOT NULL
company_id BIGINT UNSIGNED NOT NULL
created_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### Indexes

```text
INDEX user_id
INDEX company_id
UNIQUE user_id, company_id
```

---

## 7.2 user_department_accesses

Defines which departments a user can access.

### Columns

```text
id BIGINT UNSIGNED PK
user_id BIGINT UNSIGNED NOT NULL
department_id BIGINT UNSIGNED NOT NULL
created_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### Indexes

```text
INDEX user_id
INDEX department_id
UNIQUE user_id, department_id
```

---

## 7.3 document_access_rules

Defines document-specific access overrides.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
user_id BIGINT UNSIGNED NULL
role_id BIGINT UNSIGNED NULL
access_type VARCHAR(50) NOT NULL
can_access BOOLEAN DEFAULT TRUE
created_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### access_type values

```text
view
download
edit
delete
approve
manage_permissions
```

### Notes

- Either `user_id` or `role_id` can be used.
- This table is for exceptions/special rules.
- General role permission should be handled by Spatie Permission.

### Indexes

```text
INDEX document_id
INDEX user_id
INDEX role_id
INDEX access_type
INDEX can_access
```

---

# 8. Approval Tables

## 8.1 approval_flows

Defines approval matrix for category/company/department.

### Columns

```text
id BIGINT UNSIGNED PK
company_id BIGINT UNSIGNED NULL
department_id BIGINT UNSIGNED NULL
category_id BIGINT UNSIGNED NOT NULL
name VARCHAR(150) NOT NULL
status TINYINT DEFAULT 1
created_by BIGINT UNSIGNED NULL
updated_by BIGINT UNSIGNED NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
deleted_at TIMESTAMP NULL
```

### Notes

- Category-specific flow is required.
- Company/department nullable means global flow for category.

### Indexes

```text
INDEX company_id
INDEX department_id
INDEX category_id
INDEX status
```

---

## 8.2 approval_steps

Defines steps inside an approval flow.

### Columns

```text
id BIGINT UNSIGNED PK
approval_flow_id BIGINT UNSIGNED NOT NULL
step_order INT NOT NULL
step_name VARCHAR(150) NULL
approver_type VARCHAR(50) NOT NULL
approver_user_id BIGINT UNSIGNED NULL
approver_role_id BIGINT UNSIGNED NULL
is_required BOOLEAN DEFAULT TRUE
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### approver_type values

```text
user
role
```

### Indexes

```text
INDEX approval_flow_id
INDEX step_order
INDEX approver_type
INDEX approver_user_id
INDEX approver_role_id
UNIQUE approval_flow_id, step_order
```

---

## 8.3 document_approvals

Stores approval execution history for a document.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
approval_flow_id BIGINT UNSIGNED NOT NULL
approval_step_id BIGINT UNSIGNED NOT NULL
step_order INT NOT NULL
assigned_to_user_id BIGINT UNSIGNED NULL
assigned_to_role_id BIGINT UNSIGNED NULL
status VARCHAR(50) DEFAULT 'pending'
action_by BIGINT UNSIGNED NULL
action_at TIMESTAMP NULL
comments TEXT NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### status values

```text
pending
approved
rejected
skipped
```

### Indexes

```text
INDEX document_id
INDEX approval_flow_id
INDEX approval_step_id
INDEX step_order
INDEX assigned_to_user_id
INDEX assigned_to_role_id
INDEX status
INDEX action_by
```

---

# 9. Audit Log Table

## 9.1 audit_logs

Stores important system activity.

### Columns

```text
id BIGINT UNSIGNED PK
user_id BIGINT UNSIGNED NULL
event VARCHAR(100) NOT NULL
auditable_type VARCHAR(150) NULL
auditable_id BIGINT UNSIGNED NULL
old_values JSON NULL
new_values JSON NULL
ip_address VARCHAR(45) NULL
user_agent TEXT NULL
created_at TIMESTAMP NULL
```

### Example events

```text
document_uploaded
document_updated
document_viewed
document_downloaded
document_deleted
document_restored
document_submitted_for_approval
document_approved
document_rejected
permission_changed
```

### Indexes

```text
INDEX user_id
INDEX event
INDEX auditable_type, auditable_id
INDEX created_at
```

---

# 10. OCR Table — Future Phase

## 10.1 document_ocr_texts

Stores OCR extracted text from scanned documents.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
text LONGTEXT NULL
ocr_status VARCHAR(50) DEFAULT 'pending'
processed_at TIMESTAMP NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### ocr_status values

```text
pending
processing
completed
failed
```

### Indexes

```text
UNIQUE document_id
INDEX ocr_status
FULLTEXT text
```

### Notes

- OCR should be processed asynchronously using queue jobs.
- OCR text should later be synced with Meilisearch.

---

# 11. Important Foreign Key Relationships

```text
departments.company_id → companies.id

document_categories.parent_id → document_categories.id
document_categories.company_id → companies.id
document_categories.department_id → departments.id

metadata_fields.category_id → document_categories.id

documents.company_id → companies.id
documents.department_id → departments.id
documents.category_id → document_categories.id
documents.uploaded_by → users.id
documents.approved_by → users.id

document_files.document_id → documents.id
document_files.uploaded_by → users.id

document_metadata_values.document_id → documents.id
document_metadata_values.metadata_field_id → metadata_fields.id

document_tag_links.document_id → documents.id
document_tag_links.document_tag_id → document_tags.id

user_company_accesses.user_id → users.id
user_company_accesses.company_id → companies.id

user_department_accesses.user_id → users.id
user_department_accesses.department_id → departments.id

document_access_rules.document_id → documents.id
document_access_rules.user_id → users.id
document_access_rules.role_id → roles.id

approval_flows.company_id → companies.id
approval_flows.department_id → departments.id
approval_flows.category_id → document_categories.id

approval_steps.approval_flow_id → approval_flows.id
approval_steps.approver_user_id → users.id
approval_steps.approver_role_id → roles.id

document_approvals.document_id → documents.id
document_approvals.approval_flow_id → approval_flows.id
document_approvals.approval_step_id → approval_steps.id
document_approvals.assigned_to_user_id → users.id
document_approvals.assigned_to_role_id → roles.id
document_approvals.action_by → users.id

audit_logs.user_id → users.id

document_ocr_texts.document_id → documents.id
```

---

# 12. Recommended Laravel Model Names

```text
Company
Department
DocumentCategory
MetadataField
Document
DocumentFile
DocumentMetadataValue
DocumentTag
UserCompanyAccess
UserDepartmentAccess
DocumentAccessRule
ApprovalFlow
ApprovalStep
DocumentApproval
AuditLog
DocumentOcrText
```

---

# 13. Recommended Enums

## 13.1 DocumentStatus

```text
DRAFT = draft
PENDING_APPROVAL = pending_approval
APPROVED = approved
REJECTED = rejected
ARCHIVED = archived
```

## 13.2 ConfidentialityLevel

```text
PUBLIC = public
INTERNAL = internal
CONFIDENTIAL = confidential
RESTRICTED = restricted
```

## 13.3 ApprovalStatus

```text
PENDING = pending
APPROVED = approved
REJECTED = rejected
SKIPPED = skipped
```

## 13.4 MetadataFieldType

```text
TEXT = text
TEXTAREA = textarea
NUMBER = number
DATE = date
SELECT = select
MULTI_SELECT = multi_select
CHECKBOX = checkbox
BOOLEAN = boolean
```

## 13.5 OcrStatus

```text
PENDING = pending
PROCESSING = processing
COMPLETED = completed
FAILED = failed
```

---

# 14. Search Indexing Design

## 14.1 MySQL Searchable Fields

The following should be indexed in MySQL:

```text
documents.title
documents.document_no
documents.company_id
documents.department_id
documents.category_id
documents.status
documents.document_date
documents.expiry_date
document_metadata_values.value
document_metadata_values.value_date
document_metadata_values.value_number
document_ocr_texts.text
```

## 14.2 Meilisearch Document Index

Later, each document should be indexed with a flattened structure:

```json
{
  "id": 1,
  "uuid": "...",
  "title": "Land Registration Document",
  "document_no": "LAND-2026-001",
  "company_id": 1,
  "company_name": "ABC Foods Ltd",
  "department_id": 2,
  "department_name": "Legal Department",
  "category_id": 3,
  "category_name": "Land Documents",
  "status": "approved",
  "confidentiality_level": "restricted",
  "tags": ["land", "legal"],
  "metadata": {
    "mouza": "Example Mouza",
    "dag_no": "1234",
    "khatian_no": "5678"
  },
  "ocr_text": "Extracted text from scanned document..."
}
```

Access control should still be applied before returning search results.

---

# 15. Performance Notes

## 15.1 Important Query Filters

Most document listing/search queries will filter by:

```text
company_id
department_id
category_id
status
document_date
expiry_date
uploaded_by
```

These must be indexed.

## 15.2 Metadata Search

Metadata search can become expensive. For frequently filtered metadata fields:

- Use `is_searchable = true`
- Store typed values in `value_date` and `value_number`
- Consider syncing metadata to Meilisearch

## 15.3 File Storage

Do not store actual files in database.

Store files on disk/cloud storage and store only path + metadata in database.

---

# 16. Deletion Strategy

Use soft deletes for important business tables:

```text
companies
departments
document_categories
metadata_fields
documents
document_files
approval_flows
```

Do not hard delete documents by default. Deleted documents should be recoverable by authorized admins.

---

# 17. Backup Considerations

The database and file storage must be backed up together.

A database backup without the uploaded files is not useful.

Recommended backup scope:

```text
- MySQL database
- storage/app/documents
- .env/config backup
- search index can be rebuilt, so it is not mandatory to back up initially
```

---

# 18. Initial Migration Development Order

Codex should create migrations in this order:

```text
1. companies
2. departments
3. document_categories
4. metadata_fields
5. documents
6. document_files
7. document_metadata_values
8. document_tags
9. document_tag_links
10. user_company_accesses
11. user_department_accesses
12. document_access_rules
13. approval_flows
14. approval_steps
15. document_approvals
16. audit_logs
17. document_ocr_texts
```

Spatie Permission migrations should be installed before role/access-related migrations.

---

# 19. Codex Rules for Database Implementation

1. Use Laravel migrations.
2. Use foreign keys where safe and practical.
3. Use soft deletes for important business records.
4. Use indexes for searchable/filterable fields.
5. Use enums/constants for status values.
6. Do not store files in database.
7. Keep `documents` and `document_files` separate.
8. Keep common fields in `documents` and custom fields in `document_metadata_values`.
9. Keep approval setup and approval execution separate.
10. Keep access override rules separate from Spatie role permissions.

---

# 20. Summary

This database design provides a strong enterprise-ready foundation for a Document Management System. It supports group-company hierarchy, document classification, flexible metadata, access control, approval workflow, audit logging, search optimization, file versioning, and future OCR.

The first implementation should focus on the core tables and relationships. Advanced OCR and Meilisearch integration can be added after the basic document management flow is stable.
