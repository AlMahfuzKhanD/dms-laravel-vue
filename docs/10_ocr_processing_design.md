# Document Management System (DMS) — OCR Processing Design

## 1. Purpose

This document defines how OCR (Optical Character Recognition) will be designed and implemented in the Document Management System.

The company will scan many physical documents such as land papers, legal documents, licenses, agreements, HR papers, finance documents, and administrative records. Many scanned documents will be PDF/image files. Without OCR, users can only search by manually entered metadata. OCR will allow users to search inside scanned document content.

OCR is important, but it should not be implemented before the core document storage, metadata, access control, approval, and search foundation are stable.

---

## 2. OCR Goal

OCR should allow the system to extract readable text from scanned files and make that text searchable.

Example:

```text
User uploads scanned land document.
OCR extracts text from scanned image/PDF.
User later searches "dag no 1234".
System finds the scanned document even if dag no was not manually entered in metadata.
```

---

## 3. Core OCR Principles

```text
1. OCR must be asynchronous.
2. File upload must not wait for OCR processing.
3. OCR text must be stored separately from document metadata.
4. OCR result must be indexed for search.
5. OCR text may contain sensitive data, so access control is mandatory.
6. OCR can fail; failure should not break document upload.
7. OCR should be retryable.
8. OCR should be processed only for supported file types.
```

---

## 4. OCR Implementation Phase

OCR should be implemented in Phase 3, after:

```text
- Document upload works
- File versioning works
- Metadata works
- Access control works
- Basic search works
- Approval workflow works
```

Do not implement OCR in the first MVP unless specifically required.

---

## 5. Supported File Types for OCR

Initial OCR-supported types:

```text
PDF
JPG
JPEG
PNG
```

Future support:

```text
TIFF
WEBP
Multi-page scanned PDF optimization
```

Non-OCR types:

```text
DOC
DOCX
XLS
XLSX
ZIP
```

Office files may be searchable through text extraction later, but that is separate from OCR.

---

## 6. OCR Tool Recommendation

Recommended open-source OCR tool:

```text
Tesseract OCR
```

For PDF-to-image conversion, possible tools:

```text
Poppler
ImageMagick
Ghostscript
```

Recommended later setup:

```text
Tesseract + Poppler
```

Tesseract extracts text from images. Poppler can convert scanned PDF pages into images.

---

## 7. OCR Architecture

OCR should run through queued jobs.

```text
Document/File Uploaded
        ↓
Create OCR record as pending
        ↓
Dispatch OCR job
        ↓
Job processes file
        ↓
Extract text
        ↓
Save text in document_ocr_texts
        ↓
Update OCR status
        ↓
Update search index
```

---

## 8. OCR Table

## 8.1 document_ocr_texts

Stores extracted OCR text for a document.

### Columns

```text
id BIGINT UNSIGNED PK
document_id BIGINT UNSIGNED NOT NULL
document_file_id BIGINT UNSIGNED NULL
text LONGTEXT NULL
ocr_status VARCHAR(50) DEFAULT 'pending'
error_message TEXT NULL
processed_at TIMESTAMP NULL
created_at TIMESTAMP NULL
updated_at TIMESTAMP NULL
```

### Status Values

```text
pending
processing
completed
failed
skipped
```

### Notes

```text
document_file_id is useful because OCR result should ideally belong to a specific file/version.
If document has a new version, OCR should run again for the latest file.
```

---

## 9. Important Database Design Update

The earlier database design included:

```text
document_ocr_texts.document_id
```

Recommended improvement:

```text
Add document_file_id nullable foreign key
```

Reason:

```text
OCR should be version-aware.
Each file version may have different text.
```

Recommended relationship:

```text
Document hasMany DocumentOcrText
DocumentFile hasOne DocumentOcrText
DocumentOcrText belongsTo Document
DocumentOcrText belongsTo DocumentFile
```

---

## 10. OCR Status Behavior

### 10.1 pending

OCR record created but not processed yet.

### 10.2 processing

OCR job is currently running.

### 10.3 completed

OCR text extracted successfully.

### 10.4 failed

OCR job failed.

Store reason in:

```text
error_message
```

### 10.5 skipped

File type is not supported or OCR was disabled.

---

## 11. OCR Processing Flow Details

## 11.1 On File Upload

```text
1. Validate file
2. Store file
3. Create document_files row
4. Check if file type supports OCR
5. If supported, create document_ocr_texts row with pending status
6. Dispatch ProcessDocumentOcrJob
7. Continue normal upload response
```

Important:

```text
User should not wait for OCR to finish.
```

---

## 11.2 OCR Job Flow

```text
1. Load document file
2. Mark OCR status = processing
3. Detect file type
4. If image → run Tesseract
5. If PDF → convert pages to images, then run Tesseract
6. Combine extracted text
7. Save text
8. Mark status = completed
9. Dispatch search re-index job
```

---

## 11.3 OCR Failure Flow

If OCR fails:

```text
1. Mark status = failed
2. Save error message
3. Log audit/system event
4. Do not affect document upload/status
```

---

## 12. OCR Reprocessing

System should allow authorized users to re-run OCR.

Use cases:

```text
- OCR failed
- Better OCR engine added later
- Document was scanned in poor quality
- New version uploaded
```

Reprocess behavior:

```text
Set status = pending
Clear old error message
Dispatch OCR job again
```

---

## 13. OCR and Search Integration

OCR text should be searchable only after processing is completed.

### MySQL Phase

Use FULLTEXT index on:

```text
document_ocr_texts.text
```

### Meilisearch Phase

Include OCR text in searchable payload:

```json
{
  "ocr_text": "Extracted OCR text here..."
}
```

---

## 14. OCR Search Security

OCR text can contain highly sensitive data.

Important rule:

```text
Never expose OCR text or OCR snippets to unauthorized users.
```

Search results must still pass through:

```text
DocumentAccessService
```

---

## 15. OCR Text Display

OCR text should not be shown by default.

Recommended UI:

```text
- Show OCR status on document details page
- Allow authorized users to view extracted OCR text if needed
- Show warning: OCR text may contain errors
```

---

## 16. OCR Quality Considerations

OCR accuracy depends on:

```text
- Scan quality
- Image resolution
- Lighting
- Text clarity
- Document language
- Page orientation
- Handwritten vs printed text
```

OCR works better on printed text than handwritten documents.

---

## 17. Language Support

Initial OCR language:

```text
English
```

Possible future language support:

```text
Bengali
English + Bengali mixed documents
```

Tesseract language data must be installed for each language.

---

## 18. Performance Considerations

OCR is CPU-heavy and slow for large scanned PDFs.

Rules:

```text
1. Process OCR in queue.
2. Limit file size for OCR.
3. Limit page count if needed.
4. Add retry limit.
5. Run OCR during off-peak time if system grows.
```

Recommended queue:

```text
Redis queue
```

---

## 19. Queue Job Design

Create job:

```text
ProcessDocumentOcrJob
```

Recommended properties:

```text
public int $documentFileId
public int $tries = 3
public int $timeout = 300
```

Recommended behavior:

```text
- Load DocumentFile
- Validate file exists
- Process OCR
- Save result
- Dispatch indexing job
```

---

## 20. Services

Create:

```text
DocumentOcrService
PdfToImageService
OcrTextCleanupService
DocumentOcrIndexService
```

### DocumentOcrService

```text
- Coordinates OCR processing
- Detects file type
- Calls PDF/image processors
- Saves OCR text
```

### PdfToImageService

```text
- Converts PDF pages into images
- Returns temporary image paths
```

### OcrTextCleanupService

```text
- Cleans extra spaces
- Normalizes line breaks
- Removes invalid characters
```

### DocumentOcrIndexService

```text
- Updates search index after OCR completed
```

---

## 21. Temporary File Handling

PDF conversion may create temporary image files.

Rules:

```text
1. Store temporary files outside public directory.
2. Delete temporary files after OCR.
3. Clean failed job temp files with scheduled cleanup.
```

Recommended temp path:

```text
storage/app/private/tmp/ocr
```

---

## 22. OCR Settings

Future configurable settings:

```text
ocr_enabled = true
ocr_max_file_size_mb = 25
ocr_max_pdf_pages = 50
ocr_languages = eng
ocr_retry_count = 3
```

Can be stored in:

```text
system_settings
```

---

## 23. Audit Events

Log OCR-related events:

```text
ocr_job_created
ocr_processing_started
ocr_processing_completed
ocr_processing_failed
ocr_reprocess_requested
```

Do not store huge OCR text inside audit logs.

---

## 24. UI Requirements

Document details page should show:

```text
OCR Status: Pending / Processing / Completed / Failed / Skipped
Processed At
Reprocess OCR button if authorized
Error message if failed
```

Search UI should not force users to know OCR exists. Search should work naturally.

---

## 25. Error Handling

Common errors:

```text
File missing
Unsupported file type
Tesseract not installed
PDF conversion failed
Timeout
Unreadable scanned image
Language data missing
```

System should:

```text
- Mark OCR failed
- Store error_message
- Keep document usable
- Allow retry
```

---

## 26. Security Considerations

```text
1. Do not process files outside allowed storage.
2. Do not expose temp files publicly.
3. Validate file path before OCR.
4. Do not allow arbitrary shell command input.
5. Escape paths when calling system commands.
6. Keep OCR text protected by document permissions.
```

---

## 27. Codex Rules

Codex must follow these rules:

1. Do not implement OCR before core document flow unless instructed.
2. OCR must be queue-based.
3. Upload must not wait for OCR completion.
4. Store OCR text in document_ocr_texts.
5. Link OCR result to document_file_id for version awareness.
6. Do not expose OCR text to unauthorized users.
7. Do not store OCR text in audit logs.
8. Handle OCR failures gracefully.
9. Clean temporary files.
10. Re-index search after OCR completes.
11. Avoid unsafe shell command construction.

---

## 28. Recommended Implementation Order

```text
1. Update document_ocr_texts table with document_file_id
2. Create DocumentOcrText model
3. Create ProcessDocumentOcrJob
4. Create DocumentOcrService
5. Add OCR status creation after file upload
6. Add queue dispatch after upload
7. Add OCR result storage
8. Add OCR status display in UI
9. Add OCR reprocess button
10. Add Meilisearch re-index after OCR
```

---

## 29. Summary

OCR will make scanned physical documents searchable, but it should be implemented carefully as a background process.

The most important rule is:

```text
OCR must improve search without slowing upload or weakening document security.
```

This design keeps OCR scalable, secure, retryable, and ready for future advanced search.

