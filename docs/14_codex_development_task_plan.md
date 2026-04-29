# Document Management System (DMS) — Codex Development Task Plan

## 1. Purpose

This document defines a structured development task plan for building the Document Management System using Codex AI agent.

The goal is to make Codex work step-by-step from documentation, instead of generating random or unplanned code.

Codex must follow the architecture and documentation already prepared:

```text
01-project-overview
02-requirements-specification
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
13-ui-ux-screen-flow-design
```

---

## 2. Development Strategy

The system must be developed as a **domain-based modular Laravel monolith**.

Recommended strategy:

```text
1. Build foundation first
2. Build core document flow
3. Add access control
4. Add search
5. Add approval workflow
6. Add audit logs
7. Add advanced features later
```

Do not start with OCR, Meilisearch, dashboard charts, or complex UI before the core document flow is stable.

---

## 3. Global Codex Rules

Codex must follow these rules in every task:

```text
1. Read the relevant documentation before coding.
2. Do not create undocumented features.
3. Keep controllers thin.
4. Put business logic in services.
5. Put complex queries in repositories.
6. Use policies for authorization.
7. Use Form Request classes for validation.
8. Use enums/constants for status values.
9. Use events/listeners for audit and notification where practical.
10. Never expose private file paths.
11. Never return unscoped document queries.
12. Always apply access control on document list/search/view/download.
13. Use transactions for multi-table writes.
14. Keep code clean, testable, and modular.
```

---

## 4. Recommended Laravel Structure

Codex should follow this structure:

```text
app/
 ├── Domains/
 │    ├── Company/
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

Each domain may contain:

```text
Controllers
Models
Requests
Services
Repositories
Policies
Actions
DTOs
Enums
Jobs
Events
Listeners
```

Only create folders when needed.

---

# 5. Development Phases

## Phase 0 — Project Setup

### Goal

Prepare Laravel project foundation.

### Tasks

```text
1. Install Laravel 12
2. Configure database
3. Configure auth system
4. Install Spatie Laravel Permission
5. Configure private file disk
6. Configure Redis queue later if available
7. Prepare admin layout
8. Create route group /admin/dms
```

### Acceptance Criteria

```text
- App runs successfully
- User can log in
- Admin route group exists
- Spatie permission tables exist
- Private document disk configured
```

---

## Phase 1 — Settings Foundation

### Goal

Create system settings so operational rules are configurable.

### Tasks

```text
1. Create system_settings migration
2. Create SystemSetting model
3. Create SettingsService
4. Create SystemSettingsSeeder
5. Add default settings
6. Add cache clear logic after update
```

### Important Settings

```text
max_upload_file_size_mb
allowed_file_types
default_confidentiality_level
auto_approve_if_no_flow
ocr_enabled
search_driver
records_per_page
```

### Acceptance Criteria

```text
- Settings can be read from SettingsService
- Settings are cached
- Default settings are seeded
- No hardcoded upload limit in services
```

### Codex Prompt

```text
Read 11-system-settings-configuration-design. Implement the system settings foundation in Laravel using migration, model, seeder, and SettingsService. Use caching and provide methods get, set, getGroup, forgetCache, and isEnabled. Do not build UI yet.
```

---

## Phase 2 — Role & Permission Foundation

### Goal

Set up base roles and permissions.

### Tasks

```text
1. Install/configure Spatie Laravel Permission
2. Create RolePermissionSeeder
3. Seed base roles
4. Seed base permissions
5. Assign super_admin role to default admin user
```

### Base Roles

```text
super_admin
company_admin
department_admin
document_uploader
document_approver
document_viewer
```

### Acceptance Criteria

```text
- Roles exist
- Permissions exist
- Super admin can be assigned
- Permission middleware works
```

### Codex Prompt

```text
Read 05-permission-access-control-design. Implement Spatie Laravel Permission foundation. Create seeders for documented roles and permissions. Assign super_admin role to the first admin user. Do not implement document access logic yet.
```

---

## Phase 3 — Company Module

### Goal

Build company/concern management.

### Tasks

```text
1. Create companies migration
2. Create Company model
3. Create CompanyRepository
4. Create CompanyService
5. Create StoreCompanyRequest and UpdateCompanyRequest
6. Create CompanyController
7. Create CompanyPolicy
8. Create Blade list/create/edit pages
```

### Acceptance Criteria

```text
- Super admin can create company
- Company list works
- Company edit works
- Soft delete works
- Validation works
```

### Codex Prompt

```text
Read 03-module-breakdown, 04-database-design, and 12-api-route-design. Implement the Company module with migration, model, repository, service, requests, policy, controller, and Blade CRUD pages. Keep controller thin and put business logic in service.
```

---

## Phase 4 — Department Module

### Goal

Build department management under companies.

### Tasks

```text
1. Create departments migration
2. Create Department model
3. Create DepartmentRepository
4. Create DepartmentService
5. Create requests
6. Create DepartmentController
7. Create DepartmentPolicy
8. Create Blade pages
9. Add company filter
```

### Acceptance Criteria

```text
- Department belongs to company
- Department CRUD works
- Company filter works
- Validation works
```

### Codex Prompt

```text
Read 03-module-breakdown, 04-database-design, and 12-api-route-design. Implement the Department module under Company. Include migration, model relationship, repository, service, requests, policy, controller, and Blade CRUD UI.
```

---

## Phase 5 — Document Category Module

### Goal

Build dynamic document category structure.

### Tasks

```text
1. Create document_categories migration
2. Create DocumentCategory model
3. Add parent-child relationship
4. Create repository/service
5. Create requests/controller/policy
6. Create list/create/edit pages
7. Add requires_approval option
```

### Acceptance Criteria

```text
- Category can be global/company/department-specific
- Parent-child category works
- requires_approval can be enabled/disabled
- Category CRUD works
```

### Codex Prompt

```text
Read 03-module-breakdown, 04-database-design, 06-approval-workflow-design, and 13-ui-ux-screen-flow-design. Implement the Document Category module with parent-child category support and requires_approval flag.
```

---

## Phase 6 — Metadata Module

### Goal

Allow category-specific dynamic metadata fields.

### Tasks

```text
1. Create metadata_fields migration
2. Create document_metadata_values migration
3. Create models and relationships
4. Create MetadataFieldService
5. Create MetadataValueService
6. Create metadata field CRUD
7. Create API endpoint to fetch metadata fields by category
```

### Acceptance Criteria

```text
- Admin can create metadata fields for category
- Field types work
- Required/searchable flags exist
- API returns fields by category
```

### Codex Prompt

```text
Read 03-module-breakdown, 04-database-design, 12-api-route-design, and 13-ui-ux-screen-flow-design. Implement Metadata module with metadata fields per category and API endpoint to load fields dynamically by category.
```

---

## Phase 7 — Access Mapping Foundation

### Goal

Create company/department access assignment.

### Tasks

```text
1. Create user_company_accesses migration
2. Create user_department_accesses migration
3. Create models
4. Create UserCompanyAccessService
5. Create UserDepartmentAccessService
6. Create assignment screens/APIs
```

### Acceptance Criteria

```text
- User can be assigned to one or many companies
- User can be assigned to one or many departments
- Duplicate assignment prevented
```

### Codex Prompt

```text
Read 05-permission-access-control-design. Implement user company and department access mapping with migrations, models, services, controllers, and basic assignment UI. Do not implement document-level override yet.
```

---

## Phase 8 — Document Core Module

### Goal

Build document creation, metadata save, and basic document list.

### Tasks

```text
1. Create documents migration
2. Create Document model
3. Create document_tags and pivot migrations
4. Create DocumentRepository
5. Create DocumentCreateService
6. Create DocumentUpdateService
7. Create StoreDocumentRequest
8. Create UpdateDocumentRequest
9. Create DocumentController
10. Create document upload page shell
11. Save common metadata
12. Save custom metadata values
13. Save tags
```

### Acceptance Criteria

```text
- User can create document record
- Common fields saved
- Company/department/category saved
- Dynamic metadata values saved
- Tags saved
- Document status works
```

### Codex Prompt

```text
Read 04-database-design, 08-file-storage-versioning-design, 12-api-route-design, and 13-ui-ux-screen-flow-design. Implement the Document core module to create/update document records with common fields, custom metadata values, and tags. Do not implement approval yet. Keep file handling in a separate service.
```

---

## Phase 9 — File Storage & Versioning

### Goal

Add private file upload, secure storage, versioning, preview, and download.

### Tasks

```text
1. Create document_files migration
2. Create DocumentFile model
3. Configure private disk
4. Create DocumentFileStorageService
5. Create DocumentVersionService
6. Create FileChecksumService
7. Store uploaded file privately
8. Create latest version logic
9. Create preview route
10. Create download route
```

### Acceptance Criteria

```text
- File stored in private storage
- File path not exposed
- document_files row created
- Version increments properly
- Only one latest version exists
- Download route checks policy
- Preview route checks policy
```

### Codex Prompt

```text
Read 08-file-storage-versioning-design and 12-api-route-design. Implement private file storage and versioning for documents. Approved documents must never be overwritten. New upload creates a new version. Create secure preview and download routes with policy checks.
```

---

## Phase 10 — Document Access Control Service

### Goal

Implement reusable document visibility and permission checks.

### Tasks

```text
1. Create DocumentAccessService
2. Create DocumentPolicy
3. Implement canView
4. Implement canDownload
5. Implement canEdit
6. Implement canDelete/archive
7. Implement applyVisibleDocumentsScope
8. Use policy in document routes
```

### Acceptance Criteria

```text
- Unauthorized users cannot view documents
- Unauthorized users cannot download files
- Document list is scoped by user access
- Super admin can access all
- Company/department access works
```

### Codex Prompt

```text
Read 05-permission-access-control-design. Implement DocumentAccessService and DocumentPolicy. Apply access checks to document view, edit, preview, download, and listing. Every document query must be scoped by authenticated user's access rights.
```

---

## Phase 11 — Document Search MVP

### Goal

Build MySQL-based document search and filters.

### Tasks

```text
1. Create DocumentSearchService
2. Create DocumentSearchRequest
3. Create DocumentSearchController
4. Search by title/document_no/summary/remarks
5. Filter by company/department/category/status/date
6. Filter by tags
7. Filter by metadata values
8. Apply DocumentAccessService scope
9. Return paginated JSON or Blade list
```

### Acceptance Criteria

```text
- Search works by keyword
- Filters work
- Metadata filtering works
- Unauthorized documents never appear
- Pagination works
```

### Codex Prompt

```text
Read 07-search-architecture-design and 05-permission-access-control-design. Implement MySQL-based document search first. Support keyword, company, department, category, status, date range, tags, and metadata filters. Apply DocumentAccessService visible scope before returning results.
```

---

## Phase 12 — Audit Log Foundation

### Goal

Track important document actions.

### Tasks

```text
1. Create audit_logs migration
2. Create AuditLog model
3. Create AuditLogService
4. Create events/listeners where useful
5. Log document created/updated
6. Log file uploaded/downloaded/previewed
7. Log permission changes later
```

### Acceptance Criteria

```text
- Audit log created for major actions
- Audit log stores user/IP/user agent
- Document timeline can be shown
```

### Codex Prompt

```text
Read 09-audit-log-activity-design. Implement audit log foundation with migration, model, AuditLogService, and basic events/listeners. Log document creation, updates, file upload, preview, and download. Do not allow normal users to edit/delete audit logs.
```

---

## Phase 13 — Dynamic Approval Workflow

### Goal

Implement fully dynamic approval matrix.

### Tasks

```text
1. Create approval_flows migration
2. Create approval_steps migration
3. Create document_approvals migration
4. Create models and relationships
5. Create ApprovalFlowResolverService
6. Create DocumentApprovalService
7. Create ApprovalActionService
8. Create approval flow CRUD
9. Create approval step builder
10. Implement submit for approval
11. Implement approve/reject actions
12. Implement pending approval list
13. Add approval timeline
```

### Acceptance Criteria

```text
- Category can require approval or skip approval
- Flow can have 1 to N steps
- Step can be user-based or role-based
- Only current step can approve
- Rejection works with comment rule
- Old approval history preserved
- New version triggers new approval if required
```

### Codex Prompt

```text
Read 06-approval-workflow-design carefully. Implement fully dynamic approval workflow. No approval logic should be hardcoded. Use approval_flows and approval_steps. Support category-level approval requirement, 1 to N steps, user/role approvers, dynamic flow resolution, approve/reject, and approval timeline. Use transactions and preserve approval history.
```

---

## Phase 14 — Document Permission Override

### Goal

Add document-specific allow/deny rules.

### Tasks

```text
1. Create document_access_rules migration
2. Create DocumentAccessRule model
3. Create DocumentAccessRuleService
4. Add manage permissions UI
5. Update DocumentAccessService to check explicit allow/deny
6. Audit permission changes
```

### Acceptance Criteria

```text
- Explicit deny takes priority
- Explicit allow works
- Rules can be user-based or role-based
- Permission changes are audited
```

### Codex Prompt

```text
Read 05-permission-access-control-design. Implement document-level access override using document_access_rules. Support user/role allow/deny rules for view, download, edit, delete, approve, and manage_permissions. Explicit deny must take priority.
```

---

## Phase 15 — UI Polish & Professional UX

### Goal

Make the system usable and professional.

### Tasks

```text
1. Improve sidebar/menu permissions
2. Improve document upload UI
3. Improve document search UI
4. Add status badges
5. Add document details layout
6. Add version history section
7. Add approval timeline section
8. Add activity timeline section
9. Add confirmation modals
10. Add user-friendly toast messages
```

### Acceptance Criteria

```text
- UI looks professional
- Buttons are permission-aware
- Document detail page is complete
- Users can understand document status easily
```

### Codex Prompt

```text
Read 13-ui-ux-screen-flow-design. Improve the DMS UI using Blade and Vue 3 hybrid approach. Add clean layout, status badges, document details, version history, approval timeline, activity timeline, and permission-aware action buttons. Backend permission checks must remain mandatory.
```

---

## Phase 16 — Meilisearch Integration Later

### Goal

Add high-performance search after MySQL search is stable.

### Tasks

```text
1. Install Laravel Scout
2. Configure Meilisearch
3. Add searchable payload to Document model/service
4. Create DocumentIndexingService
5. Re-index document on update
6. Use Meilisearch for keyword search
7. Apply DB access filtering before response
```

### Acceptance Criteria

```text
- Meilisearch search works
- Search is fast
- Unauthorized documents are still blocked
- Index updates after document changes
```

### Codex Prompt

```text
Read 07-search-architecture-design. Add Laravel Scout + Meilisearch integration. Build DocumentIndexingService and index flattened document payload. Search Meilisearch first, then apply MySQL DocumentAccessService filter before returning results. Do not return raw Meilisearch results directly.
```

---

## Phase 17 — OCR Later

### Goal

Extract searchable text from scanned documents.

### Tasks

```text
1. Create/update document_ocr_texts migration with document_file_id
2. Create DocumentOcrText model
3. Create ProcessDocumentOcrJob
4. Create DocumentOcrService
5. Add OCR status creation after file upload
6. Process image/PDF files asynchronously
7. Store extracted text
8. Re-index search
9. Add OCR status UI
```

### Acceptance Criteria

```text
- Upload does not wait for OCR
- OCR runs in queue
- OCR text is stored securely
- OCR failure does not break document upload
- OCR text is searchable only by authorized users
```

### Codex Prompt

```text
Read 10-ocr-processing-design. Implement OCR as a queued background process. Link OCR result to document_file_id. Store OCR text in document_ocr_texts. Do not block upload. Handle failures gracefully. Re-index search after OCR completion.
```

---

# 6. Testing Strategy

## 6.1 Must-Test Areas

```text
Access control
Document upload
File preview/download
Versioning
Search scope
Approval workflow
Audit logging
Metadata save/search
```

---

## 6.2 Critical Test Cases

```text
User cannot see documents from unassigned company
User cannot download without permission
Uploader cannot approve own document unless configured
Approved document cannot be overwritten
New version creates version_no + 1
Rejected document can be resubmitted
Search does not return unauthorized documents
Explicit deny overrides allow
```

---

# 7. Pull Request / Commit Rules

Each Codex task should be small.

Recommended:

```text
One module or sub-feature per PR/commit
```

Bad:

```text
Implement full DMS in one task
```

Good:

```text
Implement Company module CRUD
Implement DocumentFileStorageService
Implement DocumentAccessService
```

---

# 8. Common Codex Mistakes to Avoid

```text
1. Putting all logic in controllers
2. Skipping policies
3. Returning all documents without access scope
4. Storing files in public folder
5. Overwriting approved files
6. Hardcoding approval steps
7. Ignoring dynamic metadata
8. Mixing OCR with upload response
9. Returning raw Meilisearch results
10. Forgetting audit logs for sensitive actions
```

---

# 9. Recommended First Codex Task

Start with Phase 0 and Phase 1 only.

First prompt:

```text
We are building a Document Management System using Laravel 12. Read the project documentation. Start with project setup foundation only: configure domain-based folder structure, admin DMS route group, private documents disk, Spatie Laravel Permission installation, and system_settings foundation with SettingsService and seeder. Do not implement document upload yet. Keep code modular and documented.
```

---

# 10. Final Development Order Summary

```text
0. Project Setup
1. Settings Foundation
2. Role & Permission Foundation
3. Company Module
4. Department Module
5. Document Category Module
6. Metadata Module
7. Access Mapping Foundation
8. Document Core Module
9. File Storage & Versioning
10. Document Access Control Service
11. Document Search MVP
12. Audit Log Foundation
13. Dynamic Approval Workflow
14. Document Permission Override
15. UI Polish
16. Meilisearch Integration
17. OCR Processing
```

---

# 11. Summary

This development task plan gives Codex a safe and structured execution path.

The most important rule is:

```text
Build the DMS step-by-step from foundation to advanced features, never randomly.
```

If Codex follows this plan, the project will stay clean, scalable, secure, and suitable for enterprise use.
