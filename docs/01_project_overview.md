# Document Management System (DMS) — Project Overview

## 1. Project Purpose

The goal of this project is to build an enterprise-grade Document Management System (DMS) for a group of companies. The organization has multiple concerns and departments, and currently many important physical documents need to be scanned, stored, categorized, searched, accessed securely, and approved through a proper workflow.

This system will allow the company to digitally manage all types of documents such as company licenses, legal papers, land documents, agreements, HR documents, finance documents, administrative papers, and other scanned or digital files.

The system must be designed as a scalable, secure, searchable, and workflow-driven document platform.

---

## 2. Business Context

The company is a group of companies with approximately 22 concerns. Each concern may have multiple departments. Each department may manage different types of documents.

Examples of document-owning departments may include:

- Legal Department
- HR Department
- Accounts & Finance Department
- Administration Department
- Land Department
- Audit Department
- Procurement Department
- Factory / Operation Department
- Management Office

The system should not be built for one fixed department only. It must support a flexible structure so that new companies, departments, document categories, and approval workflows can be added later.

---

## 3. High-Level Goal

The system should support the following flow:

```text
Physical Document / Digital File
        ↓
Scan or Upload
        ↓
Add Metadata
        ↓
Assign Company / Department / Category
        ↓
Store Securely
        ↓
Search / Filter / Preview / Download
        ↓
Approval Workflow if Required
        ↓
Final Approved Document Archive
```

---

## 4. Main Objectives

### 4.1 Centralized Document Storage

All company documents should be stored in one centralized system instead of being scattered across physical files, local computers, email attachments, or shared folders.

### 4.2 Multi-Company Support

The system must support a group-company structure where documents can belong to a specific concern/company.

### 4.3 Department-Based Organization

Documents must be organized by department so that users can easily manage and access department-specific files.

### 4.4 Category-Based Classification

Documents should be grouped into categories such as land documents, licenses, agreements, HR files, finance documents, legal documents, and others.

### 4.5 Powerful Search

The system must provide fast and effective searching. Users should be able to search documents by title, document number, company, department, category, date range, tags, metadata fields, and later OCR-extracted text.

### 4.6 Access Control

The system must ensure that users can only access documents they are permitted to view, download, edit, approve, or delete.

### 4.7 Approval Matrix

Some document types may require approval. The system must support configurable approval workflows based on company, department, document category, or document type.

### 4.8 Audit Trail

Every important action must be tracked, including upload, update, view, download, approval, rejection, delete, restore, and permission changes.

### 4.9 Future OCR Support

Since many documents will be scanned physical files, the system should be designed to support OCR-based searchable content in the future.

---

## 5. Target Users

The system may include the following types of users:

### 5.1 Super Admin

Responsible for full system configuration, company setup, department setup, roles, permissions, and global settings.

### 5.2 Company Admin

Responsible for managing documents, users, and workflows for a specific concern/company.

### 5.3 Department Admin

Responsible for managing documents inside a specific department.

### 5.4 Document Uploader

Responsible for scanning/uploading documents and entering metadata.

### 5.5 Reviewer / Approver

Responsible for reviewing and approving/rejecting submitted documents.

### 5.6 Viewer

Can search and view/download documents based on permission.

---

## 6. Recommended Technology Stack

The project will be built as a modular Laravel-based web application.

```text
Backend: Laravel 12
Frontend: Blade + Vue 3 Hybrid
Database: MySQL 8
Search Engine: Meilisearch
Queue: Redis
File Storage: Local storage initially, S3-compatible storage later
OCR: Tesseract OCR in later phase
Permission Package: Spatie Laravel Permission + custom document access rules
UI: Tailwind CSS + reusable Vue components
```

---

## 7. Recommended Architecture Style

The system should be built as a clean modular monolith.

A microservice architecture is not recommended for the first version because the project needs fast delivery, strong business logic, and maintainable implementation. A well-structured Laravel monolith is easier to build, test, deploy, and maintain.

Suggested architecture:

```text
Laravel Application
 ├── Modules / Domains
 │    ├── Company Management
 │    ├── Department Management
 │    ├── Document Management
 │    ├── Metadata Management
 │    ├── Approval Workflow
 │    ├── Search
 │    ├── Access Control
 │    └── Audit Log
 │
 ├── Services
 ├── Repositories
 ├── Policies
 ├── Jobs
 ├── Events
 ├── Listeners
 └── Vue Components for complex screens
```

---

## 8. Initial Scope

The first version should focus on a strong foundation, not every advanced feature.

### Phase 1 — Core Foundation

- Login and user management
- Company/concern management
- Department management
- Document category management
- Document upload
- Document metadata entry
- Document listing
- Advanced filters
- Basic document preview/download
- Role-based access control
- Activity log

### Phase 2 — Enterprise Workflow

- Approval matrix setup
- Document approval/rejection flow
- Document status lifecycle
- Version control
- Document expiry reminders
- Notification system

### Phase 3 — Advanced Search and Automation

- Meilisearch integration
- OCR text extraction
- OCR-based search
- Full-text indexing
- Bulk upload support
- Retention/archive policy
- Dashboard and reports

---

## 9. Out of Scope for Initial Version

The following features should not be built in the first phase unless absolutely required:

- AI-based document classification
- Complex OCR automation
- Digital signature
- Mobile app
- Public document sharing portal
- Microservices
- Multi-server distributed storage
- Advanced analytics

These can be considered after the core DMS is stable.

---

## 10. Key Design Principles

### 10.1 Security First

Documents may contain sensitive legal, financial, HR, and land-related information. Access control must be a core part of the design, not an afterthought.

### 10.2 Search First

The system must be designed with search in mind from the beginning. Metadata, tags, and OCR text should be structured properly so search can scale later.

### 10.3 Workflow Ready

Even if approval is implemented in Phase 2, the document status and database design should be workflow-ready from Phase 1.

### 10.4 Audit Everything

Every important user action should be logged for accountability.

### 10.5 Flexible Metadata

Different document types need different fields. For example, land documents need dag/khatian/mouza information, while licenses need license number, issue date, expiry date, and authority.

The system should support both common document fields and category-specific custom metadata.

### 10.6 Scalable but Simple

The first version should be easy to build and maintain, but the structure must allow future expansion.

---

## 11. Success Criteria

The project will be considered successful if:

- Users can store scanned and digital documents properly.
- Documents are categorized by company, department, and category.
- Users can search documents quickly and accurately.
- Access control prevents unauthorized users from viewing sensitive documents.
- Approval workflow can be configured and executed.
- Important actions are logged.
- The system can later support OCR and advanced search without major redesign.

---

## 12. Codex Development Guideline

When using Codex AI agent, implementation should be done step by step from this documentation.

Recommended order:

```text
1. Read project overview
2. Read requirements document
3. Read database design
4. Implement migrations
5. Implement models and relationships
6. Implement services and repositories
7. Implement policies and permissions
8. Implement controllers and requests
9. Implement Blade/Vue UI
10. Implement tests
```

Codex should not generate random features outside the documented scope unless specifically instructed.

---

## 13. Project Status

Current status: Planning and documentation phase.

Coding should not start until the following documents are prepared:

- Project Overview
- Requirements Specification
- Module Breakdown
- Database Design
- Permission & Access Control Design
- Approval Workflow Design
- Search Architecture
- Codex Task Prompts

