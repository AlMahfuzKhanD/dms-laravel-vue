# Document Management System (DMS) — Module Breakdown

## 1. Purpose

This document defines the domain-based module structure for the Document Management System. The goal is to keep the application clean, scalable, maintainable, and easy to implement using Codex AI agent.

The system will use a modular monolith architecture. Each module owns a specific business domain and should contain its own controllers, services, requests, repositories, policies, and related logic where appropriate.

---

## 2. Architecture Style

Recommended architecture:

```text
Modular Laravel Monolith
 ├── Domain-based modules
 ├── Service layer for business logic
 ├── Repository layer for complex database queries
 ├── Policy layer for authorization
 ├── Jobs for background tasks
 ├── Events/listeners for audit and notifications
 └── Vue components for complex frontend screens
```

The system should not be built as a flat controller-heavy Laravel project.

---

## 3. Suggested App Structure

```text
app/
 ├── Domains/
 │    ├── Company/
 │    ├── Department/
 │    ├── DocumentCategory/
 │    ├── Document/
 │    ├── Metadata/
 │    ├── AccessControl/
 │    ├── Approval/
 │    ├── Search/
 │    ├── AuditLog/
 │    └── Notification/
 │
 ├── Shared/
 │    ├── Enums/
 │    ├── Helpers/
 │    ├── Traits/
 │    ├── DTOs/
 │    └── Services/
 │
 └── Support/
      ├── FileStorage/
      ├── SearchIndexing/
      └── Ocr/
```

Each domain module may contain:

```text
DomainName/
 ├── Controllers/
 ├── Models/
 ├── Requests/
 ├── Services/
 ├── Repositories/
 ├── Policies/
 ├── Actions/
 ├── DTOs/
 ├── Enums/
 └── Jobs/
```

Not every module needs every folder. Only create folders when needed.

---

## 4. Core Modules

## 4.1 Company Module

### Responsibility

Manages the group company's concerns/business units.

### Main Features

- Create company/concern
- Update company information
- Activate/deactivate company
- Assign users to company
- Filter documents by company

### Main Entities

- Company
- UserCompanyAccess

### Example Fields

```text
companies
- id
- name
- code
- address
- status
```

### Codex Boundary Rule

Company module should only manage company data and company access mapping. It should not contain document upload or approval logic.

---

## 4.2 Department Module

### Responsibility

Manages departments under each company/concern.

### Main Features

- Create department
- Update department
- Assign department to company
- Assign users to department
- Filter documents by department

### Main Entities

- Department
- UserDepartmentAccess

### Example Fields

```text
departments
- id
- company_id
- name
- code
- status
```

### Codex Boundary Rule

Department module should only manage department data and department access mapping. It should not handle document approval or search logic.

---

## 4.3 Document Category Module

### Responsibility

Manages document classification.

### Main Features

- Create document category
- Support parent-child category structure
- Assign category to company/department if needed
- Define whether category requires approval
- Define whether category supports custom metadata

### Main Entities

- DocumentCategory

### Example Categories

```text
- Land Documents
- Company License
- Legal Agreement
- HR Document
- Finance Document
- Audit Report
- Procurement Document
```

### Example Fields

```text
document_categories
- id
- parent_id
- company_id
- department_id
- name
- code
- requires_approval
- status
```

### Codex Boundary Rule

Category module should define classification only. Approval process should be handled by Approval module.

---

## 4.4 Metadata Module

### Responsibility

Manages custom metadata fields for different document categories.

### Why This Module Exists

Different document categories require different fields.

Example: Land document fields:

```text
- Mouza
- Dag No
- Khatian No
- Land Area
- Registration No
```

Example: License document fields:

```text
- License No
- Issuing Authority
- Issue Date
- Expiry Date
```

### Main Features

- Create metadata field definitions per category
- Define field type
- Define required/optional fields
- Store document metadata values
- Make metadata searchable

### Main Entities

- MetadataField
- DocumentMetadataValue

### Example Fields

```text
metadata_fields
- id
- category_id
- label
- field_key
- field_type
- is_required
- is_searchable
- sort_order
```

```text
document_metadata_values
- id
- document_id
- metadata_field_id
- value
```

### Codex Boundary Rule

Metadata module should manage dynamic fields and values only. It should not manage document file upload.

---

## 4.5 Document Module

### Responsibility

This is the central module of the system. It manages document records, file upload, versions, document status, and document lifecycle.

### Main Features

- Create document record
- Upload document file
- Store document metadata
- Update document basic information
- Upload new document version
- Preview document
- Download document
- Archive document
- Delete/restore document if allowed

### Main Entities

- Document
- DocumentFile
- DocumentVersion
- DocumentTag

### Main Document Statuses

```text
DRAFT
PENDING_APPROVAL
APPROVED
REJECTED
ARCHIVED
```

### Common Document Fields

```text
documents
- id
- company_id
- department_id
- category_id
- title
- document_no
- document_date
- expiry_date
- confidentiality_level
- status
- uploaded_by
- approved_at
- archived_at
```

### File Fields

```text
document_files
- id
- document_id
- version_no
- original_name
- stored_name
- file_path
- file_type
- mime_type
- file_size
- checksum
- uploaded_by
- is_latest
```

### Codex Boundary Rule

Document module should orchestrate document creation and updates, but should call other modules/services for approval, search indexing, access checking, and audit logging.

---

## 4.6 Access Control Module

### Responsibility

Controls who can view, upload, edit, approve, download, delete, or manage documents.

### Main Features

- Role-based permissions
- Company-level access
- Department-level access
- Category-level access
- Document-level access override
- Confidentiality level control

### Permission Actions

```text
- document.view
- document.upload
- document.edit
- document.delete
- document.download
- document.approve
- document.archive
- document.manage_permissions
```

### Main Entities

- Role
- Permission
- UserCompanyAccess
- UserDepartmentAccess
- DocumentAccessRule

### Suggested Package

Use Spatie Laravel Permission for roles and permissions, then build custom access rules for company, department, category, and document-level control.

### Codex Boundary Rule

Access Control module should not fetch all documents directly. It should provide query scopes or services that other modules can use to limit visible records.

---

## 4.7 Approval Module

### Responsibility

Manages approval matrix and document approval process.

### Main Features

- Define approval flows
- Define approval steps
- Assign approver roles/users
- Submit document for approval
- Approve document
- Reject document
- Track approval history

### Approval Flow Example

```text
Category: Land Document
Step 1: Legal Officer
Step 2: Legal Manager
Step 3: Director
```

### Main Entities

- ApprovalFlow
- ApprovalStep
- DocumentApproval

### Approval Statuses

```text
PENDING
APPROVED
REJECTED
SKIPPED
```

### Codex Boundary Rule

Approval module should manage workflow only. It should not directly handle file upload or metadata field definitions.

---

## 4.8 Search Module

### Responsibility

Provides fast search and filtering for documents.

### Main Features

- Basic database search
- Advanced filters
- Search indexing
- Meilisearch integration
- Metadata search
- OCR text search in future

### Searchable Fields

```text
- title
- document_no
- company
- department
- category
- tags
- metadata values
- OCR text
```

### Main Services

- DocumentSearchService
- DocumentIndexingService

### Codex Boundary Rule

Search module should not decide access control by itself. It must accept the current user and apply access filtering using Access Control services.

---

## 4.9 Audit Log Module

### Responsibility

Tracks important user actions for accountability and compliance.

### Events to Track

```text
- document_uploaded
- document_updated
- document_viewed
- document_downloaded
- document_deleted
- document_restored
- document_submitted_for_approval
- document_approved
- document_rejected
- permission_changed
```

### Main Entity

- AuditLog

### Example Fields

```text
audit_logs
- id
- user_id
- event
- auditable_type
- auditable_id
- old_values
- new_values
- ip_address
- user_agent
- created_at
```

### Codex Boundary Rule

Audit Log module should be event-driven where possible. Other modules should dispatch events instead of writing logs manually everywhere.

---

## 4.10 Notification Module

### Responsibility

Sends system notifications related to approval, expiry, and document activity.

### Main Features

- Notify approvers when approval is required
- Notify uploader when document is approved/rejected
- Notify responsible users before document expiry

### Notification Channels

Initial:

```text
- Database notifications
```

Future:

```text
- Email
- SMS
- WhatsApp
```

### Codex Boundary Rule

Notification module should react to events. It should not own approval or document business logic.

---

## 4.11 OCR Module — Future Phase

### Responsibility

Extracts text from scanned files so users can search inside scanned documents.

### Main Features

- Queue OCR processing job
- Extract text from image/PDF
- Store extracted text
- Index extracted text in search engine

### Suggested Tool

Tesseract OCR.

### Main Entities

- DocumentOcrText

### Codex Boundary Rule

OCR should be processed in background jobs. Upload should not wait for OCR to complete.

---

## 5. Module Interaction Flow

### 5.1 Upload Flow

```text
User uploads document
        ↓
Document Module validates and stores file
        ↓
Metadata Module stores custom metadata values
        ↓
Approval Module checks if approval is required
        ↓
Document status is set
        ↓
Audit Log event is dispatched
        ↓
Search Module indexes document
```

---

### 5.2 Search Flow

```text
User searches document
        ↓
Search Module receives filters/search keyword
        ↓
Access Control Module applies visibility rules
        ↓
Search results are returned
```

---

### 5.3 Approval Flow

```text
Document submitted for approval
        ↓
Approval Module finds approval flow
        ↓
First approval step is created
        ↓
Approver approves/rejects
        ↓
Next step is activated or document is finalized
        ↓
Audit Log event is dispatched
        ↓
Notification is sent
```

---

## 6. Development Priority

Recommended development order:

```text
1. Company Module
2. Department Module
3. Document Category Module
4. Access Control Foundation
5. Document Module
6. Metadata Module
7. Audit Log Module
8. Search Module
9. Approval Module
10. Notification Module
11. OCR Module
```

Approval is very important, but the document foundation must exist first.

---

## 7. Codex Rules

Codex must follow these rules during implementation:

1. Do not put business logic directly inside controllers.
2. Use services for business workflows.
3. Use repositories for complex queries.
4. Use policies for authorization.
5. Keep modules separated by domain.
6. Do not introduce unrelated features.
7. Follow documented database design.
8. Dispatch events for audit and notification.
9. Keep file upload, approval, search, and access control separated.
10. Write code that is easy to test and extend.

---

## 8. Summary

This DMS should be built as a domain-based modular monolith. The main system complexity is not file upload; the real complexity is document classification, access control, search, metadata, approval workflows, and audit history.

A clean module boundary will allow the project to grow safely without becoming a controller-heavy or database-query-heavy application.

