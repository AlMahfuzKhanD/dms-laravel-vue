# Document Management System (DMS) — System Settings & Configuration Design

## 1. Purpose

This document defines how system-wide and module-specific settings will be managed in the Document Management System.

The DMS should not require code changes for every operational rule. Many behaviors should be configurable from database or admin UI.

Examples:

```text
- Maximum upload file size
- Allowed file types
- OCR enable/disable
- Approval fallback behavior
- Default confidentiality level
- Retention policy
- Expiry reminder days
- Search engine mode
```

---

## 2. Core Principle

```text
Operational rules should be configurable.
Business logic should read settings from a centralized settings service.
```

Do not scatter configuration values across controllers, services, requests, and views.

---

## 3. Settings Storage Strategy

Use a database table for dynamic settings.

Static environment-specific values should remain in `.env`.

### Database settings examples

```text
max_upload_file_size_mb
allowed_file_types
ocr_enabled
expiry_reminder_days
default_confidentiality_level
```

### .env examples

```text
DB connection
Redis connection
Mail credentials
Storage disk credentials
Meilisearch host/key
```

---

## 4. system_settings Table

### Columns

```text
id BIGINT UNSIGNED PK
key VARCHAR(150) UNIQUE NOT NULL
value TEXT NULL
value_type VARCHAR(50) DEFAULT 'string'
group VARCHAR(100) NULL
label VARCHAR(150) NULL
description TEXT NULL
is_public BOOLEAN DEFAULT FALSE
is_editable BOOLEAN DEFAULT TRUE
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

---

## 5. value_type Options

```text
string
integer
boolean
json
array
decimal
```

The system should cast setting values based on `value_type`.

---

## 6. Settings Groups

Recommended groups:

```text
general
file_upload
security
approval
ocr
search
notification
retention
backup
```

---

## 7. General Settings

```text
app_name = Document Management System
company_group_name = Example Group
timezone = Asia/Dhaka
default_date_format = Y-m-d
records_per_page = 20
```

---

## 8. File Upload Settings

```text
max_upload_file_size_mb = 25
allowed_file_types = pdf,jpg,jpeg,png,doc,docx,xls,xlsx
default_storage_disk = private
allow_multiple_files_per_document = false
generate_file_checksum = true
```

These settings will be used by:

```text
Document upload request validation
DocumentFileStorageService
DocumentVersionService
```

---

## 9. Security Settings

```text
default_confidentiality_level = internal
allow_download_for_viewers = false
log_document_view = true
log_document_download = true
session_timeout_minutes = 120
```

Important:

```text
Security settings should never weaken backend authorization checks.
```

---

## 10. Approval Settings

Approval flow is mainly controlled by:

```text
document_categories
approval_flows
approval_steps
```

But global approval behavior can be configured here:

```text
auto_approve_if_no_flow = false
allow_uploader_to_approve_own_document = false
require_comment_on_reject = true
reset_approval_on_new_version = true
```

Important:

```text
If approval_flows.auto_approve_if_no_flow exists, flow-level setting should override global setting.
```

Priority:

```text
1. Flow-specific setting
2. Global system setting
3. Hardcoded safe default
```

Safe default:

```text
auto_approve_if_no_flow = false
```

---

## 11. OCR Settings

```text
ocr_enabled = false
ocr_supported_file_types = pdf,jpg,jpeg,png
ocr_max_file_size_mb = 25
ocr_max_pdf_pages = 50
ocr_languages = eng
ocr_retry_count = 3
ocr_timeout_seconds = 300
```

OCR settings will be used by:

```text
ProcessDocumentOcrJob
DocumentOcrService
PdfToImageService
```

---

## 12. Search Settings

```text
search_driver = mysql
meilisearch_enabled = false
search_results_per_page = 20
search_include_ocr_text = true
search_include_archived_by_default = false
```

Supported `search_driver` values:

```text
mysql
meilisearch
```

Initial default:

```text
search_driver = mysql
```

---

## 13. Notification Settings

```text
enable_database_notifications = true
enable_email_notifications = false
notify_approver_on_submit = true
notify_uploader_on_approval = true
notify_uploader_on_rejection = true
notify_before_expiry_days = 30
```

---

## 14. Retention Settings

```text
soft_delete_enabled = true
physical_file_delete_after_days = 365
archive_old_versions = false
retain_audit_logs_forever = true
audit_log_retention_days = null
```

Important:

```text
For MVP, do not physically delete uploaded files automatically.
```

---

## 15. Backup Settings

```text
backup_enabled = false
backup_database = true
backup_files = true
backup_retention_days = 30
backup_storage_disk = local
```

Backup implementation can be added later.

---

## 16. Public vs Private Settings

`is_public` controls whether a setting can be exposed to frontend.

### Public examples

```text
app_name
records_per_page
allowed_file_types
max_upload_file_size_mb
```

### Private examples

```text
backup settings
security settings
ocr command paths
storage disk details
```

Frontend should never receive sensitive/private settings.

---

## 17. Settings Service

Create:

```text
app/Domains/Settings/Services/SettingsService.php
```

Recommended methods:

```text
get(string $key, mixed $default = null): mixed
set(string $key, mixed $value, ?string $type = null): void
getGroup(string $group): array
forgetCache(?string $key = null): void
isEnabled(string $key): bool
```

---

## 18. Caching Strategy

Settings should be cached for performance.

Recommended:

```text
Cache all settings forever or for long duration.
Clear cache when setting is updated.
```

Example cache key:

```text
dms_settings_all
```

---

## 19. Settings Seeder

Create initial settings seeder:

```text
SystemSettingsSeeder
```

Seed important default settings:

```text
max_upload_file_size_mb
allowed_file_types
default_confidentiality_level
auto_approve_if_no_flow
ocr_enabled
search_driver
records_per_page
```

---

## 20. Settings Admin UI

Admin UI should group settings by category.

Example tabs:

```text
General
File Upload
Security
Approval
OCR
Search
Notification
Retention
Backup
```

Only super admin should manage system settings initially.

---

## 21. Validation Rules

Settings update must be validated.

Examples:

```text
max_upload_file_size_mb must be integer and greater than 0
allowed_file_types must be array/string of allowed extensions
ocr_enabled must be boolean
search_driver must be mysql or meilisearch
```

Do not allow invalid settings that can break the system.

---

## 22. Configuration Priority

When system behavior depends on settings, use this priority:

```text
1. Specific module/entity setting
2. System setting from database
3. Config file setting
4. Safe default
```

Example approval fallback:

```text
1. approval_flows.auto_approve_if_no_flow
2. system_settings.auto_approve_if_no_flow
3. false
```

---

## 23. Environment Config vs Database Settings

Use `.env` for technical infrastructure:

```text
DB_HOST
REDIS_HOST
FILESYSTEM_DISK
MEILISEARCH_HOST
MAIL_HOST
```

Use database settings for business/operational rules:

```text
max_upload_file_size_mb
ocr_enabled
approval fallback
allowed file types
notification behavior
```

---

## 24. Audit Requirements

Changing settings is sensitive.

Log these events:

```text
system_setting_created
system_setting_updated
system_setting_deleted
```

Audit log should store:

```text
key
old_value
new_value
updated_by
ip_address
created_at
```

---

## 25. Security Considerations

```text
1. Only super admin can manage settings initially.
2. Private settings must not be exposed to frontend.
3. Validate all values before saving.
4. Do not store passwords/secrets in system_settings.
5. Use .env for secrets.
6. Audit every setting change.
```

---

## 26. Codex Rules

Codex must follow these rules:

1. Create system_settings table.
2. Create SettingsService.
3. Cache settings for performance.
4. Clear cache after update.
5. Seed default settings.
6. Do not use hardcoded operational values inside services.
7. Do not store secrets in database settings.
8. Validate setting values before update.
9. Audit setting changes.
10. Expose only public settings to frontend.

---

## 27. Recommended Implementation Order

```text
1. Create system_settings migration
2. Create SystemSetting model
3. Create SettingsService
4. Create SystemSettingsSeeder
5. Add helper function if needed
6. Use settings in file upload validation
7. Use settings in OCR job
8. Use settings in search service
9. Use settings in approval fallback
10. Build settings admin UI later
```

---

## 28. Summary

System settings make the DMS flexible and maintainable. Operational rules should be changed from admin configuration instead of code whenever practical.

The most important rule is:

```text
Centralize configurable behavior in SettingsService and avoid scattered hardcoded values.
```

This will make the system easier to maintain, safer to operate, and more suitable for enterprise use.

