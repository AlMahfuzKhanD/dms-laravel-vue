# Document Management System (DMS) — Requirements Specification

## 1. Introduction

This document defines the functional and non-functional requirements for the Document Management System (DMS). It is intended to guide development (including AI-assisted coding via Codex) and ensure clarity of system behavior.

---

## 2. Functional Requirements

### 2.1 User Management

- System must support user registration and authentication
- System must support role-based access control
- Users can be assigned to:
  - Company (Concern)
  - Department
- Roles include:
  - Super Admin
  - Company Admin
  - Department Admin
  - Uploader
  - Approver
  - Viewer

---

### 2.2 Company (Concern) Management

- Create, update, delete company
- Each document must belong to a company
- Users can be restricted to specific companies

---

### 2.3 Department Management

- Create, update, delete departments
- Departments belong to a company
- Documents must be assigned to a department

---

### 2.4 Document Category Management

- Create, update, delete categories
- Categories may be hierarchical (parent-child)
- Categories belong to a department or company

---

### 2.5 Document Upload

- Upload documents (PDF, image, Word, Excel)
- Support multiple file uploads
- Validate file size and type
- Store file securely
- Store metadata with document

---

### 2.6 Document Metadata

Each document must have:

- Title
- Document number (optional but recommended)
- Company
- Department
- Category
- Upload date
- Document date
- Expiry date (optional)
- Tags (optional)
- Remarks

System should support custom metadata fields per category.

---

### 2.7 Document Listing

- List documents in tabular format
- Support pagination
- Show:
  - Title
  - Company
  - Department
  - Category
  - Uploaded by
  - Status

---

### 2.8 Document Search & Filtering

Users must be able to filter/search by:

- Title
- Document number
- Company
- Department
- Category
- Date range
- Tags
- Uploaded by

System must support fast search performance.

---

### 2.9 Document View & Download

- View document (PDF/image preview)
- Download document
- Restrict access based on permissions

---

### 2.10 Access Control

- Role-based permissions
- Additional document-level access control
- Actions:
  - View
  - Upload
  - Edit
  - Delete
  - Approve
  - Download

---

### 2.11 Approval Workflow

- Configurable approval flow per category
- Multi-step approval
- Document status:
  - Draft
  - Pending
  - Approved
  - Rejected
- Track approval history

---

### 2.12 Document Versioning

- Upload new version of existing document
- Keep previous versions
- Mark latest version

---

### 2.13 Activity Log

Track all major actions:

- Upload
- Update
- Delete
- View
- Download
- Approval

---

### 2.14 Notifications (Phase 2)

- Notify approvers
- Notify uploaders on approval/rejection

---

## 3. Non-Functional Requirements

### 3.1 Performance

- Search results should be fast (<1 second typical)
- Support thousands of documents

### 3.2 Security

- Secure file storage
- Role-based and document-level access
- Prevent unauthorized access

### 3.3 Scalability

- Support future growth (more companies, users, documents)
- Should allow integration with search engines and OCR

### 3.4 Reliability

- No data loss
- Backup mechanism required

### 3.5 Usability

- Simple UI for non-technical users
- Easy upload and search

### 3.6 Maintainability

- Clean architecture
- Modular design

---

## 4. Future Requirements (Not MVP)

- OCR-based search
- AI document classification
- Digital signature
- Mobile app

---

## 5. Assumptions

- Users are internal employees
- Documents are confidential
- Initial deployment is on a single server

---

## 6. Constraints

- Limited initial requirement clarity
- Must be generic and extensible
- Built using Laravel ecosystem

---

## 7. Codex Implementation Notes

- Follow this document strictly
- Do not add undocumented features
- Implement modules step-by-step
- Keep code modular and reusable

