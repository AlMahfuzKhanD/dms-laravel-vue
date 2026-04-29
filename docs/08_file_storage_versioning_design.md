# Document Management System (DMS) — File Storage & Versioning Design

## 1. Purpose

This document defines how files will be uploaded, stored, protected, versioned, previewed, downloaded, archived, and backed up in the Document Management System.

The DMS will store sensitive scanned and digital documents for a group of companies. Therefore, file storage must be secure, organized, scalable, and version-aware.

---

## 2. Core Principles

```text
1. Do not store files directly in the database.
2. Store files in protected storage.
3. Store only file metadata/path in database.
4. Every document can have one or more file versions.
5. Approved documents should not be overwritten.
6. New file upload after approval should create a new version.
7. File access must always go through backend permission checks.
8. File actions must be audited.
```

---

## 3. File Storage Strategy

## 3.1 Initial Storage

For the first version, use Laravel local storage:

```text
storage/app/private/documents
```

Do not store documents inside:

```text
public/
```

Reason:

```text
Public files can be accessed directly by URL.
DMS files must be protected by authorization.
```

---

## 3.2 Future Storage

Later, the system should support S3-compatible storage:

```text
AWS S3
MinIO
DigitalOcean Spaces
Wasabi
Other S3-compatible object storage
```

Database should store:

```text
disk
file_path
```

So the storage backend can change without changing business logic.

---

## 4. Recommended Folder Structure

Files should be organized by company, department, document, and version.

Recommended path:

```text
storage/app/private/documents/{company_id}/{department_id}/{document_uuid}/v{version_no}/{stored_name}
```

Example:

```text
storage/app/private/documents/1/3/550e8400-e29b-41d4-a716-446655440000/v1/factory-license.pdf
```

---

## 5. File Naming Strategy

Original file name should be preserved in database, but stored file name should be system-generated.

### Why?

Original names can contain:

```text
- spaces
- special characters
- duplicate names
- unsafe characters
```

### Recommended stored name

```text
{document_uuid}_v{version_no}_{timestamp}.{extension}
```

Example:

```text
550e8400-e29b-41d4-a716-446655440000_v1_20260429153000.pdf
```

---

## 6. Supported File Types

Initial supported file types:

```text
PDF
JPG
JPEG
PNG
DOC
DOCX
XLS
XLSX
```

Optional later:

```text
TIFF
WEBP
CSV
ZIP
```

---

## 7. File Size Rules

Initial recommendation:

```text
Max single file size: 25 MB
Max files per document: configurable
```

Later this can be controlled from system settings.

Suggested future table:

```text
system_settings
- key
- value
```

Example:

```text
max_upload_file_size_mb = 25
allowed_file_types = pdf,jpg,jpeg,png,doc,docx,xls,xlsx
```

---

## 8. Database Tables Involved

Main tables:

```text
documents
document_files
document_ocr_texts
audit_logs
```

---

## 9. document_files Table

Each uploaded file/version is stored here.

Recommended fields:

```text
id
document_id
version_no
original_name
stored_name
file_path
disk
file_extension
mime_type
file_size
checksum
uploaded_by
is_latest
created_at
updated_at
deleted_at
```

---

## 10. Versioning Concept

A document can have multiple file versions.

Example:

```text
Document: Factory Trade License

Version 1 → uploaded first
Version 2 → uploaded after correction
Version 3 → uploaded after renewal
```

Only one version should be marked as latest:

```text
is_latest = true
```

---

## 11. Version Number Rule

Version number should increment automatically.

```text
New version_no = max(existing version_no) + 1
```

Example:

```text
Existing latest = v2
New upload = v3
```

---

## 12. Versioning + Approval Rule

Very important rule:

```text
Approved documents should not be overwritten.
```

If an approved document needs correction/update:

```text
Upload new version
        ↓
Mark document status according to approval requirement
        ↓
If category requires approval → pending_approval
        ↓
If no approval required → approved
```

---

## 13. Document Status Behavior with Versioning

### 13.1 Draft Document

```text
status = draft
```

Behavior:

```text
Uploader can replace file before submission if allowed.
```

Recommended:

```text
Even draft replacement should create a new version for safety.
```

---

### 13.2 Pending Approval Document

```text
status = pending_approval
```

Behavior:

```text
File should not be changed during approval unless approval is cancelled/reset.
```

---

### 13.3 Approved Document

```text
status = approved
```

Behavior:

```text
Cannot overwrite file.
New file must create new version.
New version may require approval.
```

---

### 13.4 Rejected Document

```text
status = rejected
```

Behavior:

```text
Uploader can upload corrected version and resubmit.
```

---

### 13.5 Archived Document

```text
status = archived
```

Behavior:

```text
No new version should be uploaded unless restored by authorized user.
```

---

## 14. File Upload Flow

```text
User selects file
        ↓
Validate file type and size
        ↓
Create or update document record
        ↓
Generate document UUID if new
        ↓
Calculate next version number
        ↓
Generate stored file name
        ↓
Store file in private storage
        ↓
Create document_files row
        ↓
Mark previous versions is_latest = false
        ↓
Mark new version is_latest = true
        ↓
Dispatch audit event
        ↓
Dispatch OCR job if supported file type
        ↓
Dispatch search indexing job
```

---

## 15. File Download Flow

```text
User clicks download
        ↓
Backend receives request
        ↓
Load document and latest/specific file
        ↓
Check DocumentPolicy download permission
        ↓
Log download action
        ↓
Return file response
```

Important:

```text
Never expose real storage path to frontend.
```

---

## 16. File Preview Flow

Preview should be allowed only if user has view/preview permission.

```text
User clicks preview
        ↓
Backend checks permission
        ↓
Return protected preview response
```

### Supported Preview

Initial:

```text
PDF preview
Image preview
```

Later:

```text
Office document preview/conversion
```

---

## 17. Multiple Files Per Document

Two options exist:

### Option A — One file per version

```text
Each document version has one main file.
```

### Option B — Multiple files per version

```text
Each version can contain multiple attachments.
```

Recommended for first version:

```text
Option A: One main file per version
```

Reason:

```text
Simpler approval, versioning, preview, and audit.
```

Future support:

```text
Add attachment_group or document_file_group if multiple files per version become mandatory.
```

---

## 18. File Checksum

System should generate checksum for uploaded file.

Recommended:

```text
SHA-256
```

Purpose:

```text
- Detect duplicate uploads
- Verify file integrity
- Audit support
```

Duplicate file behavior:

```text
Warn user but do not automatically block in MVP.
```

---

## 19. File Deletion Strategy

Normal users should not hard-delete files.

Use soft delete for `document_files`.

### Delete behavior

```text
Delete document → soft delete document record
Delete version → soft delete document_files row if allowed
```

Actual physical file deletion should be restricted.

Recommended:

```text
Physical file cleanup only by scheduled admin task after retention period.
```

---

## 20. File Archive Strategy

Archiving is safer than deleting.

```text
status = archived
archived_at = timestamp
```

Archived documents:

```text
- Hidden from normal search by default
- Visible to authorized users
- Not editable
- Not versionable unless restored
```

---

## 21. File Security

## 21.1 Private Storage

Files must be stored in private disk.

```text
Do not use public storage link for confidential files.
```

---

## 21.2 Backend Controlled Access

All access should go through Laravel routes/controllers.

Example routes:

```text
GET /documents/{document}/preview
GET /documents/{document}/download
GET /documents/{document}/files/{file}/download
```

Each route must call policy/access service.

---

## 21.3 Filename Exposure

Frontend may show original file name, but should not expose:

```text
- internal file_path
- stored_name if sensitive
- server path
```

---

## 22. OCR Integration

OCR should be optional and asynchronous.

OCR should run only for supported file types:

```text
PDF
JPG
JPEG
PNG
TIFF later
```

OCR flow:

```text
File uploaded
        ↓
OCR status = pending
        ↓
Queue job starts processing
        ↓
OCR text extracted
        ↓
Save in document_ocr_texts
        ↓
Update search index
```

---

## 23. Backup Strategy

Database backup alone is not enough.

Must back up:

```text
1. MySQL database
2. Document storage directory
3. Application .env/config
```

Initial storage backup path:

```text
storage/app/private/documents
```

---

## 24. Storage Migration Strategy

If moving from local storage to S3 later:

```text
1. Copy files from local to S3
2. Update disk and file_path if needed
3. Verify checksum
4. Keep old backup until verification complete
```

Since `document_files` stores `disk`, the system can support mixed storage temporarily.

---

## 25. Audit Events

File-related events to audit:

```text
document_file_uploaded
document_file_version_created
document_file_previewed
document_file_downloaded
document_file_deleted
document_file_restored
document_file_archived
```

Audit log should include:

```text
user_id
document_id
document_file_id
version_no
ip_address
user_agent
created_at
```

---

## 26. Laravel Implementation Strategy

## 26.1 Services

Create:

```text
DocumentFileStorageService
DocumentVersionService
DocumentFileDownloadService
DocumentFilePreviewService
FileChecksumService
```

---

## 26.2 Responsibilities

### DocumentFileStorageService

```text
- Validate storage path
- Store file
- Generate stored name
- Return file metadata
```

### DocumentVersionService

```text
- Calculate next version
- Mark old versions as not latest
- Create new document_files row
- Apply document status rules
```

### DocumentFileDownloadService

```text
- Check permission
- Return secure file download response
- Log download event
```

### DocumentFilePreviewService

```text
- Check permission
- Return inline preview response
- Log preview event
```

### FileChecksumService

```text
- Generate checksum
- Detect duplicate file warning
```

---

## 27. Codex Rules

Codex must follow these rules:

1. Do not store files in database.
2. Do not store documents in public folder.
3. Do not expose storage path to frontend.
4. Always check permission before preview/download.
5. Approved documents must not be overwritten.
6. New upload after approval must create new version.
7. Only one latest version is allowed per document.
8. Use private disk for file storage.
9. Use services for file/version logic.
10. Dispatch audit events for file actions.
11. OCR must run asynchronously.
12. Use checksum for uploaded files.

---

## 28. Recommended Implementation Order

```text
1. Configure private documents disk
2. Create file upload validation rules
3. Implement DocumentFileStorageService
4. Implement DocumentVersionService
5. Implement file upload flow
6. Implement secure preview route
7. Implement secure download route
8. Add audit events
9. Add checksum generation
10. Add OCR job later
```

---

## 29. Summary

File storage and versioning must be secure and reliable. The system should never overwrite approved documents. Every important file change should create a version and be tracked.

The most important rule is:

```text
Files are private, versioned, permission-protected, and audit-tracked.
```

This design ensures that company documents remain safe, traceable, and maintainable over time.

