# Document Management System (DMS) — UI/UX Screen Flow Design

## 1. Purpose

This document defines the main user interface screens and user experience flow for the Document Management System.

The DMS will be used by non-technical internal users, admins, uploaders, approvers, and management users. The UI must be simple, fast, secure, and practical.

This document will guide Blade page creation, Vue component design, route implementation, and Codex task generation.

---

## 2. UI Design Principle

The system should feel like a professional enterprise application, not a simple file upload tool.

Core UI principles:

```text
1. Simple for normal users
2. Powerful for admins
3. Search-first experience
4. Access-aware UI
5. Clear document status
6. Clear approval timeline
7. Minimal clicks for common actions
8. Safe handling of sensitive documents
```

---

## 3. Recommended Frontend Approach

Use:

```text
Blade + Vue 3 hybrid
```

### Blade should handle:

```text
- Layout
- Sidebar
- Main page structure
- Server-rendered pages
- Permission-based menu rendering
```

### Vue should handle:

```text
- Document upload form
- Dynamic metadata fields
- Advanced search/filter
- Approval flow step builder
- File version UI
- Timeline components
```

---

## 4. Main Layout

## 4.1 Admin Layout

Common layout sections:

```text
Topbar
Sidebar
Content area
Breadcrumb
Page title
Action buttons
```

---

## 4.2 Sidebar Menu

Recommended menu:

```text
Dashboard
Documents
  - All Documents
  - Upload Document
  - Pending Approval
  - Archived Documents
Master Setup
  - Companies
  - Departments
  - Document Categories
  - Metadata Fields
Approval Setup
  - Approval Flows
  - Approval Steps
Access Control
  - User Company Access
  - User Department Access
  - Document Permissions
Reports / Logs
  - Audit Logs
  - Expiring Documents
Settings
```

Menus should be shown/hidden based on permission.

---

## 5. Dashboard Screen

## 5.1 Purpose

Show summary of document system status.

## 5.2 Cards

```text
Total Documents
Pending Approval
Approved Documents
Rejected Documents
Expiring Soon
Archived Documents
OCR Pending (future)
```

## 5.3 Quick Actions

```text
Upload Document
Search Documents
View Pending Approvals
View Expiring Documents
```

## 5.4 Charts / Lists

Optional:

```text
Documents by Company
Documents by Department
Documents by Category
Recent Uploads
Recent Activity
```

---

## 6. Company Management Screens

## 6.1 Company List

Columns:

```text
SL
Company Name
Code
Status
Created At
Actions
```

Actions:

```text
View
Edit
Deactivate/Delete
```

## 6.2 Company Create/Edit

Fields:

```text
Company Name
Code
Address
Phone
Email
Status
```

---

## 7. Department Management Screens

## 7.1 Department List

Filters:

```text
Company
Status
```

Columns:

```text
SL
Company
Department Name
Code
Status
Actions
```

## 7.2 Department Create/Edit

Fields:

```text
Company
Department Name
Code
Status
```

---

## 8. Document Category Screens

## 8.1 Category List

Filters:

```text
Company
Department
Parent Category
Status
Requires Approval
```

Columns:

```text
SL
Category Name
Parent
Company
Department
Requires Approval
Status
Actions
```

## 8.2 Category Create/Edit

Fields:

```text
Parent Category
Company optional
Department optional
Category Name
Code
Description
Requires Approval yes/no
Status
```

Important:

```text
Requires Approval controls whether documents under this category go through approval.
```

---

## 9. Metadata Field Screens

## 9.1 Metadata Field List

Shown under selected category.

Columns:

```text
SL
Label
Field Key
Field Type
Required
Searchable
Sort Order
Status
Actions
```

## 9.2 Metadata Field Create/Edit

Fields:

```text
Label
Field Key
Field Type
Placeholder
Options for select/multi_select
Required yes/no
Searchable yes/no
Sort Order
Status
```

## 9.3 UX Rule

When category is selected in document upload form, metadata fields should load dynamically.

---

# 10. Document Upload Screen

## 10.1 Purpose

Allow user to upload scanned/digital documents with proper classification and metadata.

## 10.2 Screen Sections

```text
1. Basic Information
2. Classification
3. Dynamic Metadata
4. File Upload
5. Tags & Remarks
6. Submit Actions
```

---

## 10.3 Basic Information Fields

```text
Title
Document No
Document Date
Expiry Date
Confidentiality Level
Summary
```

---

## 10.4 Classification Fields

```text
Company
Department
Category
```

Dependency:

```text
Select Company → Load Departments
Select Department → Load Categories
Select Category → Load Metadata Fields
```

---

## 10.5 Dynamic Metadata Section

Fields depend on selected category.

Example for Land Document:

```text
Mouza
Dag No
Khatian No
Land Area
Registration No
```

Example for License:

```text
License No
Issuing Authority
Issue Date
Expiry Date
```

---

## 10.6 File Upload Section

Should show:

```text
Allowed file types
Maximum file size
Selected file name
File preview if possible
```

Initial support:

```text
One main file per document version
```

---

## 10.7 Submit Buttons

Buttons depend on category approval rule.

If category does not require approval:

```text
Save Document
```

If category requires approval:

```text
Save as Draft
Submit for Approval
```

---

## 10.8 Upload UX Validation

Show clear errors:

```text
Company is required
Department is required
Category is required
File is required
Required metadata missing
File size exceeds limit
File type not allowed
```

---

# 11. Document List / Search Screen

## 11.1 Purpose

This is one of the most important screens.

Users should be able to quickly find documents.

---

## 11.2 Basic Search

Top search bar:

```text
Search by title, document no, tag, metadata, OCR text later
```

---

## 11.3 Main Filters

```text
Company
Department
Category
Status
Document Date Range
Expiry Date Range
Uploaded By
Confidentiality Level
Tags
```

---

## 11.4 Advanced Filters

Expandable section:

```text
Category-specific metadata filters
OCR status future
Approval status
Archived yes/no
```

---

## 11.5 Search Result Table

Columns:

```text
SL
Title
Document No
Company
Department
Category
Status
Confidentiality
Document Date
Expiry Date
Uploaded By
Actions
```

---

## 11.6 Row Actions

Actions depend on permission:

```text
View
Preview
Download
Edit
Upload New Version
Submit for Approval
Archive
Delete
```

Do not show actions user cannot perform.

Backend must still enforce permission.

---

## 11.7 Status Badges

Use clear badges:

```text
Draft
Pending Approval
Approved
Rejected
Archived
Expired
Expiring Soon
```

---

# 12. Document Details Screen

## 12.1 Purpose

Show complete document details, file information, approval timeline, version history, and activity.

## 12.2 Sections

```text
1. Header Summary
2. Document Information
3. Metadata
4. File Preview
5. Version History
6. Approval Timeline
7. Activity Timeline
8. Permission Panel if authorized
```

---

## 12.3 Header Summary

Show:

```text
Title
Document No
Status Badge
Confidentiality Badge
Company
Department
Category
```

Action buttons:

```text
Preview
Download
Edit
Upload New Version
Submit for Approval
Approve
Reject
Archive
```

Buttons depend on permission and status.

---

## 12.4 Metadata Display

Show common fields first:

```text
Document Date
Expiry Date
Uploaded By
Created At
Approved At
```

Then custom metadata.

---

## 12.5 File Preview

Initial preview support:

```text
PDF
Image
```

Unsupported file types:

```text
Show file icon + download button if allowed
```

---

## 12.6 Version History

Columns:

```text
Version
Original File Name
Uploaded By
Uploaded At
File Size
Is Latest
Actions
```

Actions:

```text
Preview
Download
```

---

## 12.7 Approval Timeline

Example:

```text
Step 1: Legal Officer — Approved by Rahim — 10:30 AM
Step 2: Legal Manager — Pending
Step 3: Director — Waiting
```

Rejected example:

```text
Step 2: Legal Manager — Rejected — Comment: Missing registration page
```

---

## 12.8 Activity Timeline

Example:

```text
Created
File uploaded
Submitted for approval
Approved
Downloaded
New version uploaded
```

---

# 13. Upload New Version Screen / Modal

## 13.1 Purpose

Allow authorized users to upload corrected/renewed file version.

Fields:

```text
New File
Version Note / Remarks
```

Rules:

```text
Approved document cannot be overwritten.
New upload creates new version.
If category requires approval, new version goes to pending approval.
```

---

# 14. Approval Screens

## 14.1 Pending Approval List

Shows documents assigned to current user or user's role.

Columns:

```text
SL
Title
Document No
Company
Department
Category
Current Step
Submitted By
Submitted At
Actions
```

Actions:

```text
View
Approve
Reject
```

---

## 14.2 Approval Action Modal

Approve modal:

```text
Comment optional
Confirm Approve
```

Reject modal:

```text
Comment required if configured
Confirm Reject
```

---

## 14.3 Approval Flow Setup Screen

Admin screen for configuring dynamic approval flow.

Fields:

```text
Flow Name
Company optional
Department optional
Category required
Is Default
Auto Approve If No Flow
Status
```

---

## 14.4 Approval Step Builder

Vue component recommended.

Each step row:

```text
Step Order
Step Name
Approver Type user/role
Approver User or Role
Required yes/no
Can Edit Document yes/no
Can Download Document yes/no
Requires Comment on Reject yes/no
Actions remove/reorder
```

Buttons:

```text
Add Step
Save Flow
Reorder Steps
```

---

# 15. Access Control Screens

## 15.1 User Company Access

Fields:

```text
User
Companies multi-select
```

---

## 15.2 User Department Access

Fields:

```text
User
Company filter
Departments multi-select
```

---

## 15.3 Document Permission Override

Available from document details if authorized.

Fields:

```text
User or Role
Access Type
Allow/Deny
```

Access types:

```text
view
download
edit
delete
approve
manage_permissions
```

---

# 16. Audit Log Screens

## 16.1 Global Audit Log

Filters:

```text
Event
User
Date Range
Auditable Type
Company optional
Department optional
```

Columns:

```text
Date/Time
User
Event
Target
IP Address
Details
```

---

## 16.2 Document Timeline

Shown inside document details.

Human-readable timeline preferred.

---

# 17. OCR UI — Future Phase

## 17.1 OCR Status Display

Inside document details:

```text
OCR Status: Pending / Processing / Completed / Failed / Skipped
Processed At
Reprocess OCR button if authorized
```

## 17.2 OCR Text View

Only authorized users can view extracted OCR text.

Show warning:

```text
OCR text may contain recognition errors.
```

---

# 18. Settings Screen

Settings grouped by tabs:

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

Only super admin should manage initially.

---

# 19. Responsive Design

The system is mainly desktop-focused, but should still work on tablet/mobile for viewing/searching.

Priority:

```text
Desktop first
Tablet usable
Mobile basic usable
```

---

# 20. Empty States

Use helpful empty states.

Examples:

```text
No documents found. Try changing your filters.
No approval tasks assigned to you.
No metadata fields configured for this category.
No document versions found.
```

---

# 21. Confirmation Dialogs

Require confirmation for sensitive actions:

```text
Delete document
Archive document
Download restricted document optional
Approve document
Reject document
Change permission
Change system setting
```

---

# 22. Notification / Toast Messages

Use clear messages:

```text
Document uploaded successfully.
Document submitted for approval.
Document approved successfully.
Document rejected successfully.
You are not authorized to download this document.
Approval flow not configured for this category.
```

---

# 23. Codex Rules

Codex must follow these UI rules:

1. Use Blade for layout/page shell.
2. Use Vue 3 for dynamic complex screens.
3. Do not show buttons user cannot use.
4. Backend must still check permission.
5. Keep upload form clean and step-based if needed.
6. Use status badges consistently.
7. Make search page fast and simple.
8. Use reusable components for filters, metadata fields, timelines, and approval steps.
9. Do not expose file paths in UI.
10. Keep UI desktop-first but responsive.

---

# 24. Recommended UI Implementation Order

```text
1. Admin layout and sidebar
2. Company CRUD screens
3. Department CRUD screens
4. Category CRUD screens
5. Metadata field screens
6. Document upload screen
7. Document list/search screen
8. Document details screen
9. File preview/download UI
10. Version history UI
11. Approval flow setup screen
12. Pending approval screen
13. Approval timeline
14. Audit log screen
15. Settings screen
16. OCR UI later
```

---

# 25. Summary

The DMS UI should be clean, professional, and search-focused. Normal users should be able to upload, search, preview, and download documents easily. Admin users should be able to configure companies, departments, categories, metadata, approval flows, and access control without code changes.

The most important UX rule is:

```text
Users should find documents quickly and safely, while the system silently enforces security and workflow rules.
```

