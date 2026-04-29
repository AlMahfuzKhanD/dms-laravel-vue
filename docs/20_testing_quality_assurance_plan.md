# Document Management System (DMS) — Testing & Quality Assurance Plan

## 1. Purpose

This document defines the testing and quality assurance strategy for the Document Management System.

The DMS will manage sensitive company documents across multiple concerns and departments. Therefore, testing must focus strongly on security, access control, document lifecycle, approval workflow, file protection, search accuracy, and auditability.

---

## 2. Testing Goal

The goal is to ensure that the system is:

```text
Secure
Reliable
Accurate
Searchable
Permission-safe
Workflow-correct
Audit-friendly
Maintainable
```

The system should not only work for happy paths. It must also handle unauthorized access, invalid workflows, incorrect files, rejected approvals, search restrictions, and missing configuration safely.

---

## 3. Main Testing Areas

```text
1. Authentication and authorization
2. Company and department access
3. Document upload and metadata
4. File storage and versioning
5. Search and filtering
6. Dynamic approval workflow
7. Document permission override
8. Audit logs
9. System settings
10. OCR later phase
11. UI/UX behavior
12. Performance and security
```

---

## 4. Testing Types

## 4.1 Unit Tests

Test small business logic methods.

Examples:

```text
SettingsService value casting
DocumentVersionService next version number
ApprovalFlowResolverService flow matching
DocumentAccessService permission decision
MetadataValueService value normalization
```

---

## 4.2 Feature Tests

Test full Laravel request flows.

Examples:

```text
User uploads document
User downloads allowed file
Unauthorized user cannot view document
Approver approves current step
Search returns only allowed documents
```

---

## 4.3 Browser/UI Tests Optional

Can be added later using Laravel Dusk or manual QA.

Focus:

```text
Document upload form
Dynamic metadata loading
Approval step builder
Document search page
Document details page
```

---

## 5. Critical Security Test Cases

## 5.1 Unauthorized Document View

Scenario:

```text
User A belongs to Company 1.
Document belongs to Company 2.
User A tries to view document.
```

Expected:

```text
Access denied.
Document details not shown.
```

---

## 5.2 Unauthorized Search Leakage

Scenario:

```text
User searches keyword matching restricted document from another company.
```

Expected:

```text
Restricted document does not appear in results.
No title/metadata/OCR snippet is leaked.
```

---

## 5.3 Unauthorized Download

Scenario:

```text
User can view document but does not have download permission.
User tries direct download URL.
```

Expected:

```text
Download blocked.
File not returned.
Audit/security event optionally logged.
```

---

## 5.4 Private File Path Protection

Scenario:

```text
User inspects frontend response/source.
```

Expected:

```text
No storage/app/private file path exposed.
Only secure preview/download routes are visible.
```

---

## 6. Access Control Test Cases

## 6.1 Super Admin Access

Expected:

```text
Super admin can access all companies, departments, documents, settings, and audit logs.
```

---

## 6.2 Company Access

Expected:

```text
User can only see documents from assigned companies.
```

---

## 6.3 Department Access

Expected:

```text
User can only see documents from assigned departments unless higher role allows broader access.
```

---

## 6.4 Confidentiality Level

Test levels:

```text
public
internal
confidential
restricted
```

Expected:

```text
Restricted/confidential documents require additional permission or explicit access.
```

---

## 6.5 Explicit Deny Priority

Scenario:

```text
User has role permission to view document.
Document has explicit deny rule for that user.
```

Expected:

```text
Access denied.
Explicit deny wins.
```

---

## 6.6 Explicit Allow

Scenario:

```text
User normally cannot access restricted document.
Admin adds explicit allow for view.
```

Expected:

```text
User can view only the allowed document/action.
```

---

## 7. Document Upload Test Cases

## 7.1 Valid Upload

Expected:

```text
Document record created.
File stored privately.
Metadata saved.
Tags saved.
Audit log created.
```

---

## 7.2 Invalid File Type

Expected:

```text
Upload rejected with validation error.
No document/file row created.
```

---

## 7.3 File Size Exceeded

Expected:

```text
Upload rejected according to system setting max_upload_file_size_mb.
```

---

## 7.4 Required Metadata Missing

Expected:

```text
Upload rejected.
User sees missing required metadata field error.
```

---

## 7.5 Save as Draft

Expected:

```text
Document status = draft.
Approval not started.
```

---

## 7.6 Submit Without Approval Required

Scenario:

```text
Category requires_approval = false.
```

Expected:

```text
Document becomes approved or saved according to service rule.
No approval steps created.
```

---

## 7.7 Submit With Approval Required

Scenario:

```text
Category requires_approval = true.
```

Expected:

```text
Document status = pending_approval.
Approval cycle created.
First step pending/active.
```

---

## 8. File Storage & Versioning Test Cases

## 8.1 Private Storage

Expected:

```text
File stored under private disk/path.
File is not accessible directly through public URL.
```

---

## 8.2 Initial Version

Expected:

```text
document_files version_no = 1.
is_latest = true.
```

---

## 8.3 New Version

Expected:

```text
Previous file is_latest = false.
New file version_no = previous max + 1.
New file is_latest = true.
```

---

## 8.4 Approved Document Cannot Be Overwritten

Scenario:

```text
Document status = approved.
User uploads updated file.
```

Expected:

```text
New version created.
Old version preserved.
No overwrite happens.
```

---

## 8.5 Version + Approval

Scenario:

```text
Approved document category requires approval.
New version uploaded.
```

Expected:

```text
New approval required.
Document status changes according to workflow rule.
```

---

## 8.6 Checksum

Expected:

```text
Checksum generated for uploaded file.
Duplicate file can be detected/warned.
```

---

## 9. Dynamic Approval Workflow Test Cases

Approval must follow the latest fully dynamic approval design.

## 9.1 Category Approval Disabled

Scenario:

```text
requires_approval = false
```

Expected:

```text
No approval workflow starts.
```

---

## 9.2 One-Step Approval

Scenario:

```text
Approval flow has 1 step.
```

Expected:

```text
After step approval, document becomes approved.
```

---

## 9.3 Multi-Step Approval

Scenario:

```text
Approval flow has 3 steps.
```

Expected:

```text
Only Step 1 active initially.
After Step 1 approval → Step 2 active.
After Step 2 approval → Step 3 active.
After Step 3 approval → document approved.
```

---

## 9.4 Dynamic Step Increase

Scenario:

```text
Admin adds Step 4 to approval flow.
```

Expected:

```text
New submissions follow 4-step flow without code change.
```

---

## 9.5 Dynamic Step Reduction

Scenario:

```text
Admin reduces approval flow from 5 steps to 2 steps.
```

Expected:

```text
New submissions follow 2-step flow without code change.
Existing approval history remains safe.
```

---

## 9.6 Role-Based Approver

Scenario:

```text
Step approver_type = role.
```

Expected:

```text
Any user with assigned role can approve current step.
After one approves, step is completed.
```

---

## 9.7 User-Based Approver

Scenario:

```text
Step approver_type = user.
```

Expected:

```text
Only assigned user can approve.
Other users cannot approve.
```

---

## 9.8 Rejection Comment Required

Scenario:

```text
requires_comment_on_reject = true.
Approver rejects without comment.
```

Expected:

```text
Reject action blocked with validation/business error.
```

---

## 9.9 Rejection and Resubmission

Expected:

```text
Document status = rejected.
Uploader can edit.
Resubmission creates new approval_cycle.
Old history remains.
```

---

## 9.10 No Approval Flow Found

Scenario A:

```text
auto_approve_if_no_flow = true
```

Expected:

```text
Document auto-approved.
```

Scenario B:

```text
auto_approve_if_no_flow = false
```

Expected:

```text
Submission blocked with approval flow not configured message.
```

---

## 10. Search Test Cases

## 10.1 Keyword Search

Expected:

```text
Search finds documents by title, document_no, summary, remarks, tags, and metadata.
```

---

## 10.2 Metadata Filter

Expected:

```text
Filtering by custom metadata returns matching documents only.
```

---

## 10.3 Date Range Filter

Expected:

```text
Document date and expiry date filters work correctly.
```

---

## 10.4 Access-Controlled Search

Expected:

```text
Search returns only documents user can access.
```

---

## 10.5 Archived Documents

Expected:

```text
Archived documents hidden by default unless filter/permission allows.
```

---

## 11. Audit Log Test Cases

## 11.1 Document Created

Expected:

```text
Audit log created with user, event, auditable document, IP, user agent.
```

---

## 11.2 File Downloaded

Expected:

```text
Audit log created for successful download.
```

---

## 11.3 Approval Action

Expected:

```text
Approve/reject action logged with step info and comment where applicable.
```

---

## 11.4 Permission Change

Expected:

```text
Permission/access rule changes logged with old/new values.
```

---

## 11.5 Audit Logs Are Append-Only

Expected:

```text
Normal users cannot edit/delete audit logs.
```

---

## 12. System Settings Test Cases

## 12.1 Settings Read

Expected:

```text
SettingsService returns correct typed values.
```

---

## 12.2 Settings Cache

Expected:

```text
Settings are cached.
Cache clears after update.
```

---

## 12.3 Upload Limit from Settings

Expected:

```text
File upload validation uses configured max_upload_file_size_mb.
```

---

## 12.4 Approval Fallback from Settings

Expected:

```text
Approval fallback respects flow-specific setting first, then system setting, then safe default.
```

---

## 13. OCR Test Cases — Later Phase

## 13.1 OCR Job Created

Expected:

```text
Supported file upload creates pending OCR record and dispatches job.
```

---

## 13.2 Upload Does Not Wait

Expected:

```text
Document upload response returns without waiting for OCR completion.
```

---

## 13.3 OCR Success

Expected:

```text
OCR text saved.
OCR status = completed.
Search index updated.
```

---

## 13.4 OCR Failure

Expected:

```text
OCR status = failed.
Error message saved.
Document remains usable.
```

---

## 13.5 OCR Access Control

Expected:

```text
Unauthorized users cannot view OCR text or OCR snippets.
```

---

## 14. UI Manual QA Checklist

## 14.1 Document Upload UI

```text
✔ Company → department dropdown works
✔ Department → category dropdown works
✔ Category → metadata fields load
✔ Required metadata shows required indicator
✔ File validation errors are visible
✔ Draft/submit buttons behave based on approval requirement
```

---

## 14.2 Document Search UI

```text
✔ Keyword search works
✔ Filters work
✔ Reset filters works
✔ Pagination works
✔ Empty state appears
✔ Unauthorized actions hidden
```

---

## 14.3 Document Details UI

```text
✔ Status badge visible
✔ Metadata shown correctly
✔ File preview works if allowed
✔ Download button hidden if not allowed
✔ Version history visible
✔ Approval timeline visible
✔ Activity timeline visible
```

---

## 14.4 Approval UI

```text
✔ Pending approval list shows assigned tasks only
✔ Approve works for current approver
✔ Reject requires comment if configured
✔ Step builder supports add/remove/reorder
✔ Step builder supports user/role approver
```

---

## 15. Performance QA

## 15.1 Document List Performance

Expected:

```text
Paginated document list loads quickly with indexes.
```

---

## 15.2 Search Performance

MVP target:

```text
MySQL search should be acceptable for initial document volume.
```

Later target:

```text
Meilisearch should return fast keyword results with final DB permission filtering.
```

---

## 15.3 Large File Upload

Expected:

```text
System validates file size and handles failure gracefully.
```

---

## 16. Regression Testing Checklist

Run these after major changes:

```text
✔ Login works
✔ Super admin can access all modules
✔ Normal user cannot access unauthorized documents
✔ Upload document works
✔ Download protected file works only with permission
✔ Search is access-controlled
✔ Approval flow still dynamic
✔ Audit logs are written
✔ Settings still load from cache
```

---

## 17. Suggested Automated Test Structure

```text
tests/Feature/Dms/
 ├── AccessControlTest.php
 ├── DocumentUploadTest.php
 ├── DocumentVersioningTest.php
 ├── DocumentSearchTest.php
 ├── ApprovalWorkflowTest.php
 ├── AuditLogTest.php
 ├── SettingsTest.php
 └── OcrProcessingTest.php later

 tests/Unit/Dms/
 ├── SettingsServiceTest.php
 ├── DocumentVersionServiceTest.php
 ├── ApprovalFlowResolverServiceTest.php
 ├── DocumentAccessServiceTest.php
 └── MetadataValueServiceTest.php
```

---

## 18. Codex Testing Prompts

## 18.1 Access Control Test Prompt

```text
Read 05-permission-access-control-design and 20-testing-quality-assurance-plan.

Create Laravel feature tests for DMS document access control.

Test:
- Super admin can view all documents.
- User cannot view document from unassigned company.
- User cannot view document from unassigned department.
- User cannot download without document.download permission.
- Explicit deny overrides allow.
- Explicit allow grants view for specific document.

Use factories where possible. Do not bypass policies in tests.
```

---

## 18.2 Versioning Test Prompt

```text
Read 08-file-storage-versioning-design and 20-testing-quality-assurance-plan.

Create tests for document file versioning.

Test:
- Initial file creates version 1 and is_latest true.
- New upload creates version 2.
- Previous version becomes is_latest false.
- Approved document is not overwritten.
- File is stored in private disk.
- File path is not exposed in response.
```

---

## 18.3 Dynamic Approval Test Prompt

```text
Read the latest 06-approval-workflow-design and 20-testing-quality-assurance-plan.

Create feature/unit tests for fully dynamic approval workflow.

Test:
- Category with requires_approval false skips approval.
- One-step flow approves document after one approval.
- Three-step flow activates steps sequentially.
- Role-based approver can approve.
- Non-assigned user cannot approve.
- Reject requires comment when configured.
- Rejected document can be resubmitted and creates new approval_cycle.
- No flow found respects auto_approve_if_no_flow true/false.

Approval steps must come from database rows. Do not hardcode steps in tests except as test data.
```

---

## 18.4 Search Security Test Prompt

```text
Read 07-search-architecture-design, 05-permission-access-control-design, and 20-testing-quality-assurance-plan.

Create feature tests for document search.

Test:
- Keyword search finds allowed documents.
- Metadata filter works.
- Tag filter works.
- Search does not return documents from unassigned company.
- Search does not return restricted documents without permission.
- Archived documents are hidden by default.
```

---

## 18.5 Audit Log Test Prompt

```text
Read 09-audit-log-activity-design and 20-testing-quality-assurance-plan.

Create tests for audit logging.

Test:
- Document create writes audit log.
- File download writes audit log.
- Approval action writes audit log.
- Permission change writes audit log.
- Normal users cannot delete audit logs.
```

---

## 19. Release Readiness Checklist

Before first internal release:

```text
✔ Migrations run successfully
✔ Seeders run successfully
✔ Super admin login works
✔ Company/department/category setup works
✔ Metadata field setup works
✔ Document upload works
✔ File stored privately
✔ Search/list access control works
✔ Approval flow works dynamically
✔ Download/preview permission works
✔ Audit logs are created
✔ Basic UI usable by non-technical users
✔ No direct file paths exposed
✔ Backup plan documented
```

---

## 20. Common Bugs to Watch

```text
1. User sees documents from wrong company.
2. Search leaks unauthorized document titles.
3. Download route bypasses policy.
4. Approved file gets overwritten.
5. Approval flow assumes fixed number of steps.
6. Rejection loses old approval history.
7. Metadata values stored only as text and date/number filters fail.
8. Settings cache not cleared after update.
9. Audit logs missing for sensitive actions.
10. Frontend hides button but backend still allows action.
```

---

## 21. Summary

Testing is critical for this DMS because the system handles sensitive company documents and workflow approvals.

The most important QA rule is:

```text
Security and access-control bugs are more dangerous than normal UI bugs.
```

Every major feature must be tested against unauthorized access, workflow correctness, data integrity, and audit traceability before production use.
