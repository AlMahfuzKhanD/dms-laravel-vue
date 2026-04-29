# Document Management System (DMS) — Search Architecture Design

## 1. Purpose

This document defines the search architecture for the Document Management System.

Search is one of the most important features of this system because the company will scan and store many physical documents across multiple concerns, departments, and categories.

The system must allow users to quickly find documents by common fields, custom metadata, tags, date ranges, and later OCR-extracted text from scanned documents.

---

## 2. Search Goals

The search system must support:

```text
- Fast document search
- Secure access-controlled search results
- Company/department/category filtering
- Metadata-based filtering
- Tag-based filtering
- Date range filtering
- OCR text search in future
- Scalable search architecture
```

Most important rule:

```text
Search must never return unauthorized documents.
```

---

## 3. Search Phases

Search should be implemented in phases.

```text
Phase 1: MySQL-based search and filters
Phase 2: Meilisearch full-text search
Phase 3: OCR text extraction and OCR search
Phase 4: Advanced ranking and saved search
```

---

# 4. Phase 1 — MySQL Search MVP

## 4.1 Purpose

Phase 1 provides strong search and filtering without introducing extra infrastructure.

This is recommended for the first working version.

---

## 4.2 Searchable Fields

Search should support these document fields:

```text
- title
- document_no
- summary
- remarks
- company_id
- department_id
- category_id
- status
- confidentiality_level
- document_date
- expiry_date
- uploaded_by
- created_at
```

---

## 4.3 Metadata Search

Search should also support custom metadata fields from:

```text
document_metadata_values
```

Example metadata fields:

```text
Land Document:
- mouza
- dag_no
- khatian_no
- registration_no

License Document:
- license_no
- issuing_authority
- issue_date
- expiry_date
```

---

## 4.4 Tag Search

Documents can be searched by tags.

Example:

```text
land
legal
license
urgent
renewal
factory
```

---

## 4.5 Basic Keyword Search

Keyword search should search in:

```text
- documents.title
- documents.document_no
- documents.summary
- documents.remarks
- document_metadata_values.value
- document_tags.name
```

Example user search:

```text
"factory license"
"land dag 1234"
"ABC Foods agreement"
```

---

## 4.6 Advanced Filters

Search page should support filters:

```text
company_id
department_id
category_id
status
confidentiality_level
document_date_from
document_date_to
expiry_date_from
expiry_date_to
uploaded_by
tags
metadata fields
```

---

## 4.7 Access Control in MySQL Search

Every search query must apply access control.

Search query flow:

```text
DocumentSearchService
        ↓
DocumentRepository builds query
        ↓
DocumentAccessService applies visible scope
        ↓
Filters are applied
        ↓
Results are returned
```

Important:

```text
Do not search all documents first and filter in frontend.
Access filtering must happen in backend query.
```

---

## 4.8 Recommended MySQL Indexes

From `documents` table:

```text
INDEX company_id
INDEX department_id
INDEX category_id
INDEX status
INDEX confidentiality_level
INDEX document_no
INDEX document_date
INDEX expiry_date
INDEX uploaded_by
INDEX created_at
FULLTEXT title, document_no, summary
```

From `document_metadata_values`:

```text
INDEX document_id
INDEX metadata_field_id
INDEX value_date
INDEX value_number
FULLTEXT value
```

From `document_tags`:

```text
INDEX name
UNIQUE slug
```

---

# 5. Phase 2 — Meilisearch Integration

## 5.1 Why Meilisearch

MySQL filters are good for structured queries, but scanned document search and fast keyword search across many documents will need a search engine.

Meilisearch provides:

```text
- Fast full-text search
- Typo tolerance
- Ranking
- Filtering
- Simple Laravel Scout integration
- Good developer experience
```

---

## 5.2 Laravel Integration

Use:

```text
Laravel Scout + Meilisearch
```

Documents should be indexed when:

```text
- Document created
- Document updated
- Metadata updated
- Tags updated
- Approval status changed
- OCR text extracted
```

---

## 5.3 Search Index Structure

Each document should be indexed in a flattened structure.

Example:

```json
{
  "id": 1,
  "uuid": "document-uuid",
  "title": "Factory Trade License",
  "document_no": "LIC-2026-001",
  "summary": "Trade license for factory operation",
  "company_id": 1,
  "company_name": "ABC Foods Ltd",
  "department_id": 2,
  "department_name": "Legal Department",
  "category_id": 5,
  "category_name": "License Document",
  "status": "approved",
  "confidentiality_level": "confidential",
  "document_date": "2026-01-10",
  "expiry_date": "2027-01-10",
  "uploaded_by": 3,
  "tags": ["license", "factory", "legal"],
  "metadata": {
    "license_no": "123456",
    "issuing_authority": "City Corporation"
  },
  "ocr_text": "Extracted searchable text from scanned document"
}
```

---

## 5.4 Filterable Attributes

Configure Meilisearch filterable attributes:

```text
company_id
department_id
category_id
status
confidentiality_level
uploaded_by
document_date
expiry_date
tags
```

---

## 5.5 Searchable Attributes

Configure searchable attributes:

```text
title
document_no
summary
company_name
department_name
category_name
tags
metadata
ocr_text
```

---

## 5.6 Meilisearch Access Control Rule

Meilisearch should not be the final security layer.

Recommended safe flow:

```text
1. Search Meilisearch
2. Get matching document IDs
3. Query MySQL documents by those IDs
4. Apply DocumentAccessService visible scope
5. Return final allowed results
```

This prevents unauthorized document leakage.

---

## 5.7 Why Not Directly Return Meilisearch Results

Do not directly return Meilisearch results because:

```text
- Search index may contain sensitive metadata
- Access rules can change anytime
- Meilisearch may still have stale indexed data
- Document-level deny/allow rules must be checked in database
```

Final result must always be permission-checked using database access logic.

---

# 6. Phase 3 — OCR Search

## 6.1 Why OCR is Needed

The company will scan many physical documents. Scanned documents are often images or scanned PDFs. Without OCR, users can only search by manually entered metadata.

OCR allows users to search inside scanned document content.

Example:

```text
Search: "dag no 1234"
System finds scanned land document even if user did not manually enter dag no in metadata.
```

---

## 6.2 OCR Processing Flow

OCR must run in background jobs.

```text
Document uploaded
        ↓
OCR job dispatched
        ↓
OCR extracts text
        ↓
Text saved in document_ocr_texts
        ↓
Document search index updated
```

---

## 6.3 OCR Status

Use `document_ocr_texts.ocr_status`:

```text
pending
processing
completed
failed
```

---

## 6.4 OCR Tool

Suggested tool:

```text
Tesseract OCR
```

For scanned PDFs, system may need PDF-to-image conversion before OCR.

Possible tools later:

```text
Tesseract
Poppler
ImageMagick
```

---

## 6.5 OCR Performance Rule

OCR can be slow. Therefore:

```text
Upload should not wait for OCR.
OCR must be processed by queue.
```

---

# 7. Search Module Design

## 7.1 Services

Create:

```text
DocumentSearchService
DocumentIndexingService
DocumentOcrIndexService
```

---

## 7.2 DocumentSearchService Responsibilities

```text
- Receive search request
- Normalize filters
- Decide MySQL or Meilisearch search
- Apply access control
- Return paginated results
```

---

## 7.3 DocumentIndexingService Responsibilities

```text
- Prepare document searchable payload
- Sync document to Meilisearch
- Remove document from index if deleted/archived if needed
- Re-index document after metadata/tag/status changes
```

---

## 7.4 DocumentOcrIndexService Responsibilities

```text
- Store OCR text
- Update document search index
- Mark OCR status
```

---

# 8. Search Request Structure

Suggested request payload:

```json
{
  "keyword": "factory license",
  "company_id": 1,
  "department_id": 2,
  "category_id": 5,
  "status": "approved",
  "confidentiality_level": "confidential",
  "document_date_from": "2026-01-01",
  "document_date_to": "2026-12-31",
  "expiry_date_from": "2026-01-01",
  "expiry_date_to": "2027-12-31",
  "uploaded_by": 3,
  "tags": ["license", "factory"],
  "metadata": {
    "license_no": "123456",
    "issuing_authority": "City Corporation"
  }
}
```

---

# 9. Search Result Structure

Suggested response:

```json
{
  "data": [
    {
      "id": 1,
      "uuid": "document-uuid",
      "title": "Factory Trade License",
      "document_no": "LIC-2026-001",
      "company": "ABC Foods Ltd",
      "department": "Legal Department",
      "category": "License Document",
      "status": "approved",
      "confidentiality_level": "confidential",
      "document_date": "2026-01-10",
      "expiry_date": "2027-01-10",
      "uploaded_by": "User Name",
      "can_view": true,
      "can_download": false,
      "can_edit": false
    }
  ],
  "meta": {
    "total": 100,
    "per_page": 20,
    "current_page": 1
  }
}
```

---

# 10. Frontend Search UX

The search UI should include:

```text
- Global search box
- Company filter
- Department filter
- Category filter
- Status filter
- Date range filters
- Tag filter
- Advanced metadata filters
- Reset filter button
- Search result table/grid
- Preview/download actions based on permission
```

---

## 10.1 Search UX Principle

Most users are non-technical. Search should feel simple.

Recommended UI:

```text
Basic Search:
- Single search box
- Main filters

Advanced Search:
- Expandable section
- Metadata-specific fields
```

---

# 11. Ranking Strategy

When using Meilisearch, ranking should prioritize:

```text
1. Exact document_no match
2. Exact title match
3. Title partial match
4. Tag match
5. Metadata match
6. OCR text match
```

Reason:

OCR text can be noisy, so it should not always outrank clean metadata/title matches.

---

# 12. Saved Search — Future Phase

Later, users may save common filters.

Example:

```text
My Expiring Licenses
Legal Documents Pending Approval
Land Documents of Company A
```

Suggested table later:

```text
saved_searches
- id
- user_id
- name
- filters JSON
- created_at
- updated_at
```

---

# 13. Expiry Search / Reminder

Documents with expiry date should be searchable and reportable.

Example filters:

```text
- Expiring in next 30 days
- Expired documents
- Expiring this month
```

This is important for:

```text
- licenses
- agreements
- certificates
- renewals
```

---

# 14. Security Risks

## 14.1 Risk: Unauthorized Search Result Leakage

Bad example:

```text
Meilisearch returns document title and metadata directly to unauthorized user.
```

Solution:

```text
Always apply database access control before final response.
```

---

## 14.2 Risk: OCR Text Contains Sensitive Data

OCR text may contain confidential legal or financial information.

Solution:

```text
Do not expose OCR snippets unless user has view permission.
```

---

## 14.3 Risk: Stale Search Index

Document permission or status may change but index may not be updated immediately.

Solution:

```text
Final MySQL permission check is mandatory.
```

---

# 15. Codex Implementation Rules

Codex must follow these rules:

1. Start with MySQL search first.
2. Do not add Meilisearch before core document flow works.
3. Centralize search in DocumentSearchService.
4. Use DocumentAccessService in every search/list query.
5. Never return unauthorized documents.
6. Keep OCR processing asynchronous.
7. Store OCR text separately in document_ocr_texts.
8. Index only prepared searchable payload.
9. Re-index when document, metadata, tags, status, or OCR text changes.
10. Do not put search logic directly in controllers.

---

# 16. Recommended Implementation Order

```text
1. Build MySQL document listing filters
2. Add keyword search across document fields
3. Add metadata filters
4. Add tag filters
5. Apply access-controlled query scope
6. Add Meilisearch integration
7. Add OCR text table and queue job
8. Add OCR text indexing
9. Add saved searches and advanced ranking
```

---

# 17. Summary

Search must be designed as a secure, scalable, and phased system.

The first version should use MySQL filters and access-controlled queries. After the core document flow is stable, Meilisearch should be added for fast full-text search. OCR should be added later to make scanned documents searchable.

The most important rule is:

```text
Search result security is more important than search speed.
```

Therefore, every search result must pass through the DMS access control layer before being returned to the user.

