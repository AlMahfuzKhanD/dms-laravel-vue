# Document Management System (DMS) — Audit Log & Activity Design

## 1. Purpose

This document defines how audit logs and activity tracking should work in the Document Management System.

The DMS will store sensitive legal, land, HR, finance, license, agreement, and company documents. Therefore, every important action must be traceable.

Audit logging helps answer questions like:

```text
Who uploaded this document?
Who viewed this document?
Who downloaded this file?
Who changed metadata?
Who approved or rejected it?
Who changed permission?
When was this document deleted or archived?
```

---

## 2. Audit Design Goal

The system must maintain a reliable history of important actions.

Audit log must support:

```text
- Security accountability
- User activity tracking
- Document lifecycle tracking
- Approval history tracking
- Permission change tracking
- Investigation support
- Compliance support
```

---

## 3. Core Principles

```text
1. Audit logs must be append-only.
2. Normal users must not edit or delete audit logs.
3. Important actions must be logged automatically.
4. Audit logging should be event-driven where possible.
5. Audit logs should store old and new values for important updates.
6. Audit logs should include user, IP address, and user agent.
7. Audit logs must respect access control when displayed.
```

---

## 4. Main Table

## 4.1 audit_logs

Stores system activity and auditable actions.

### Columns

```text
id BIGINT UNSIGNED PK
user_id BIGINT UNSIGNED NULL
event VARCHAR(100) NOT NULL
auditable_type VARCHAR(150) NULL
auditable_id BIGINT UNSIGNED NULL
old_values JSON NULL
new_values JSON NULL
metadata JSON NULL
ip_address VARCHAR(45) NULL
user_agent TEXT NULL
created_at TIMESTAMP NULL
```

### Notes

```text
user_id NULL means system-generated event.
auditable_type + auditable_id supports polymorphic logging.
old_values and new_values support change tracking.
metadata stores extra contextual information.
```

---

## 5. Event Naming Convention

Use clear snake_case event names.

Examples:

```text
document_created
document_updated
document_deleted
document_restored
document_archived
document_submitted_for_approval
document_approved
document_rejected
document_viewed
document_downloaded
document_file_uploaded
document_file_version_created
permission_changed
approval_flow_created
approval_flow_updated
metadata_field_created
metadata_value_updated
user_company_access_changed
user_department_access_changed
login_success
login_failed
```

---

## 6. Events That Must Be Logged

## 6.1 Document Events

```text
document_created
document_updated
document_deleted
document_restored
document_archived
document_status_changed
```

---

## 6.2 File Events

```text
document_file_uploaded
document_file_version_created
document_file_previewed
document_file_downloaded
document_file_deleted
document_file_restored
```

---

## 6.3 Approval Events

```text
document_submitted_for_approval
approval_step_approved
approval_step_rejected
document_approved
document_rejected
approval_flow_triggered
```

---

## 6.4 Permission Events

```text
role_assigned
role_removed
permission_assigned
permission_removed
document_permission_changed
user_company_access_changed
user_department_access_changed
```

---

## 6.5 Metadata Events

```text
metadata_field_created
metadata_field_updated
metadata_field_deleted
document_metadata_created
document_metadata_updated
```

---

## 6.6 Search Events — Optional

Search logging can become heavy.

Initial recommendation:

```text
Do not log every search in MVP.
```

Later optional events:

```text
document_searched
advanced_search_used
```

Use only if business requires search analytics.

---

## 7. What Should Be Stored

## 7.1 Basic Info

Every audit log should store:

```text
user_id
event
auditable_type
auditable_id
ip_address
user_agent
created_at
```

---

## 7.2 Change Tracking

For update events, store:

```text
old_values
new_values
```

Example:

```json
{
  "old_values": {
    "title": "Old License Name",
    "expiry_date": "2026-01-01"
  },
  "new_values": {
    "title": "Updated License Name",
    "expiry_date": "2027-01-01"
  }
}
```

---

## 7.3 Extra Metadata

Use `metadata` JSON for contextual details.

Example for file download:

```json
{
  "document_id": 10,
  "document_file_id": 15,
  "version_no": 2,
  "file_name": "factory-license.pdf"
}
```

Example for approval:

```json
{
  "document_id": 10,
  "approval_step_id": 3,
  "step_order": 2,
  "comment": "Approved after verification"
}
```

---

## 8. Audit Log Display

Audit logs should be visible only to authorized users.

Possible views:

```text
1. Global Audit Log
2. Company Audit Log
3. Department Audit Log
4. Document Activity Timeline
5. User Activity Log
```

---

## 9. Document Activity Timeline

Each document should have an activity timeline.

Example:

```text
2026-04-29 10:15 AM — Created by Rahim
2026-04-29 10:20 AM — File uploaded v1
2026-04-29 10:25 AM — Submitted for approval
2026-04-29 11:00 AM — Approved by Legal Manager
2026-04-29 11:10 AM — Downloaded by Director
```

This is very useful for document traceability.

---

## 10. Access Control for Audit Logs

Audit logs themselves may contain sensitive information.

### Super Admin

```text
Can view all audit logs.
```

### Company Admin

```text
Can view audit logs for assigned companies only.
```

### Department Admin

```text
Can view audit logs for assigned departments only.
```

### Normal User

```text
Can view timeline of documents they are allowed to view if permitted.
```

---

## 11. Event-Driven Logging Strategy

Preferred approach:

```text
Business action happens
        ↓
Event is dispatched
        ↓
Audit listener writes audit log
```

Example:

```text
DocumentUploaded event
        ↓
WriteDocumentAuditLog listener
```

This keeps controllers and services clean.

---

## 12. When Direct Logging Is Acceptable

Direct logging is acceptable for simple or security-sensitive cases.

Examples:

```text
- login_failed
- unauthorized_access_attempt
- file_downloaded
```

But business workflows should prefer events.

---

## 13. Recommended Events and Listeners

## 13.1 Events

```text
DocumentCreated
DocumentUpdated
DocumentDeleted
DocumentArchived
DocumentSubmittedForApproval
DocumentApproved
DocumentRejected
DocumentFileUploaded
DocumentFileDownloaded
DocumentPermissionChanged
UserCompanyAccessChanged
UserDepartmentAccessChanged
```

---

## 13.2 Listener

```text
WriteAuditLog
```

Optionally create specialized listeners later:

```text
WriteDocumentAuditLog
WriteApprovalAuditLog
WritePermissionAuditLog
```

---

## 14. AuditLogService

Create:

```text
app/Domains/AuditLog/Services/AuditLogService.php
```

Recommended methods:

```text
log(string $event, ?Model $auditable, array $oldValues = [], array $newValues = [], array $metadata = []): AuditLog
logDocumentAction(string $event, Document $document, array $metadata = []): AuditLog
logFileAction(string $event, DocumentFile $file, array $metadata = []): AuditLog
logPermissionChange(Model $target, array $oldValues, array $newValues): AuditLog
```

---

## 15. Audit Log Query Filters

Audit log page should support filters:

```text
event
user_id
auditable_type
auditable_id
company_id if available via metadata or relation
department_id if available via metadata or relation
date_from
date_to
```

---

## 16. Performance Considerations

Audit logs can grow quickly.

Use indexes:

```text
INDEX user_id
INDEX event
INDEX auditable_type, auditable_id
INDEX created_at
```

For large systems, later consider:

```text
- monthly partitioning
- archive old logs
- separate audit database
```

Not needed for MVP.

---

## 17. Retention Policy

Audit logs should not be deleted casually.

Recommended:

```text
Keep audit logs permanently in MVP.
```

Future:

```text
Keep 3–7 years depending on company policy.
Archive old logs instead of deleting.
```

---

## 18. Security Events

Optional but recommended security logs:

```text
unauthorized_access_attempt
invalid_download_attempt
permission_denied
login_failed
password_changed
role_changed
```

These events can help investigate misuse.

---

## 19. UI Recommendations

## 19.1 Global Audit Page

Columns:

```text
Date/Time
User
Event
Target Type
Target Name/ID
IP Address
Action Details
```

---

## 19.2 Document Timeline

Should show clean human-readable text.

Example:

```text
Rahim uploaded version 2 of Factory License.
Karim approved Step 1: Legal Review.
Director downloaded approved document.
```

---

## 20. Codex Rules

Codex must follow these rules:

1. Create audit_logs table with JSON fields.
2. Do not allow normal edit/delete of audit logs.
3. Use AuditLogService for writing logs.
4. Prefer event-driven audit logging.
5. Log view/download actions carefully.
6. Store old/new values for important updates.
7. Include IP address and user agent.
8. Apply access control when showing audit logs.
9. Do not log every search in MVP.
10. Keep audit code separate from business logic.

---

## 21. Recommended Implementation Order

```text
1. Create audit_logs migration
2. Create AuditLog model
3. Create AuditLogService
4. Create basic WriteAuditLog listener
5. Dispatch events from document actions
6. Dispatch events from file actions
7. Dispatch events from approval actions
8. Dispatch events from permission changes
9. Build document timeline UI
10. Build global audit log UI
```

---

## 22. Summary

Audit logging is mandatory for an enterprise DMS. The system must track sensitive actions without mixing logging code everywhere.

The most important rule is:

```text
Every important document, file, approval, and permission action must be traceable.
```

A strong audit log will make the DMS trustworthy, secure, and suitable for company-wide u