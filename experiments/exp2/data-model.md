# Proposal Analyser — Data Model Design

**Feature:** Source Selection 2.7 — Proposal Analyser
**Date:** February 2026

---

## 1. Existing Tables Referenced

These are the existing tables that the new data model connects to. No structural changes to these tables (except the field-length update noted in Section 5).

| Existing Table | Key Columns Used | Role in Proposal Analyser |
|---|---|---|
| `AS_GSS_Evaluation` | `id`, `status`, `evaluationMethod` | Parent evaluation; analysis is triggered when evaluation starts |
| `AS_GSS_EvaluationVendor` | `id`, `evaluationId`, `legalName` | Vendor being analysed |
| `AS_GSS_Criteria` | `id`, `evaluationId`, `criteriaName`, `description`, `instructions`, `weight`, `parentCriteriaId` | Factor/subfactor definitions; description + instructions feed the AI prompt |
| `AS_GSS_EvaluationDocument` | `id`, `evaluationId`, `documentType` | Source documents uploaded per vendor |
| `AS_GSS_FactorDocumentMapping` | `id`, `documentId`, `criteriaId` | Maps which documents are relevant to which factors |

---

## 2. New Tables

### 2.1 `AS_GSS_VendorAnalysisExtraction`

Tracks each AI extraction run. One record per vendor-factor extraction request. This is the "extraction job" that captures metadata about the AI call itself.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | Integer (PK) | No | Auto-generated primary key |
| `evaluationId` | Integer (FK → `AS_GSS_Evaluation.id`) | No | Parent evaluation |
| `vendorId` | Integer (FK → `AS_GSS_EvaluationVendor.id`) | No | Vendor being analysed |
| `criteriaId` | Integer (FK → `AS_GSS_Criteria.id`) | No | Factor/subfactor being analysed |
| `extractionType` | Integer (FK → `AS_GSS_R_DATA.id`) | No | Reference data: `VENDOR_FACTOR_ANALYSIS`, `VENDOR_OVERALL_SUMMARY` |
| `promptIdentifier` | Text (255) | Yes | Identifier/version of the AI prompt template used |
| `status` | Integer (FK → `AS_GSS_R_DATA.id`) | No | Reference data: `QUEUED`, `IN_PROGRESS`, `COMPLETED`, `FAILED` |
| `requestPayload` | Text (16000) | Yes | Serialised prompt sent to AI (for debugging/audit) |
| `responsePayload` | Text (16000) | Yes | Raw AI response (for debugging/audit) |
| `aiProvider` | Integer (FK → `AS_GSS_R_DATA.id`) | Yes | Reference data: `AZURE_OPENAI`, `APPIAN_NATIVE_AI` |
| `tokenCount` | Integer | Yes | Tokens consumed by this extraction |
| `durationMs` | Integer | Yes | Execution time in milliseconds |
| `errorMessage` | Text (4000) | Yes | Error details if status = FAILED |
| `createdBy` | Text (255) | No | System user or process that triggered extraction |
| `createdOn` | Timestamp | No | When the extraction was initiated |
| `modifiedBy` | Text (255) | Yes | Last modifier |
| `modifiedOn` | Timestamp | Yes | Last modification timestamp |

**Purpose:** Maps to the "Document Extraction Data" story — stores extraction identifier, type, prompt identifier, status, and metadata for each AI call.

---

### 2.2 `AS_GSS_VendorAnalysis`

Stores the overall AI-computed metrics and summary for each vendor within an evaluation. One record per vendor per evaluation.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | Integer (PK) | No | Auto-generated primary key |
| `evaluationId` | Integer (FK → `AS_GSS_Evaluation.id`) | No | Parent evaluation |
| `vendorId` | Integer (FK → `AS_GSS_EvaluationVendor.id`) | No | Vendor being analysed |
| `overallSummary` | Text (16000) | Yes | AI-generated narrative summary across all factors |
| `overallScore` | Decimal (5,4) | Yes | Weighted sum of per-factor scores (0–1 range) |
| `fitment` | Integer (FK → `AS_GSS_R_DATA.id`) | Yes | Reference data: `STRONG`, `MODERATE`, `WEAK` |
| `confidenceScore` | Decimal (5,4) | Yes | Weighted average of per-factor AI confidence scores |
| `documentCount` | Integer | Yes | Number of documents analysed for this vendor (displayed in UI header) |
| `analysisStatus` | Integer (FK → `AS_GSS_R_DATA.id`) | No | Reference data: `PENDING`, `IN_PROGRESS`, `COMPLETED`, `FAILED`, `REGENERATING` |
| `lastGeneratedOn` | Timestamp | Yes | When the analysis was last completed |
| `createdBy` | Text (255) | No | Creator |
| `createdOn` | Timestamp | No | Creation timestamp |
| `modifiedBy` | Text (255) | Yes | Last modifier |
| `modifiedOn` | Timestamp | Yes | Last modification timestamp |

**Purpose:** Maps to the "AI Vendor Metrics" story — stores Strong/Moderate/Weak categorisation, vendor confidence, vendor summary, and confidence breakdown.

---

### 2.3 `AS_GSS_VendorFactorAnalysis`

Stores the AI analysis results per vendor-factor combination. One record per vendor-factor pair.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | Integer (PK) | No | Auto-generated primary key |
| `vendorAnalysisId` | Integer (FK → `AS_GSS_VendorAnalysis.id`) | No | Parent vendor analysis |
| `evaluationId` | Integer (FK → `AS_GSS_Evaluation.id`) | No | Parent evaluation (denormalised for query performance) |
| `vendorId` | Integer (FK → `AS_GSS_EvaluationVendor.id`) | No | Vendor (denormalised for query performance) |
| `criteriaId` | Integer (FK → `AS_GSS_Criteria.id`) | No | Factor/subfactor being analysed |
| `factorSummary` | Text (16000) | Yes | AI-generated summary for this vendor-factor combination |
| `factorScore` | Decimal (5,4) | Yes | Score: 1 (Strong), 0.6 (Moderate), 0.25 (Weak) |
| `factorFitment` | Integer (FK → `AS_GSS_R_DATA.id`) | Yes | Reference data: `STRONG`, `MODERATE`, `WEAK` |
| `factorConfidence` | Decimal (5,4) | Yes | Average AI confidence for findings in this factor (0–1) |

| `extractionId` | Integer (FK → `AS_GSS_VendorAnalysisExtraction.id`) | Yes | Link to the extraction job that produced this analysis |
| `createdBy` | Text (255) | No | Creator |
| `createdOn` | Timestamp | No | Creation timestamp |
| `modifiedBy` | Text (255) | Yes | Last modifier |
| `modifiedOn` | Timestamp | Yes | Last modification timestamp |

**Purpose:** Maps to the "Extracted Information Storage" story — stores factor-based summaries and category-level metrics.

---

### 2.4 `AS_GSS_VendorAnalysisFinding`

Stores individual findings extracted by AI. Multiple findings per vendor-factor combination.

| Column | Type | Nullable | Description |
|---|---|---|---|
| `id` | Integer (PK) | No | Auto-generated primary key |
| `vendorFactorAnalysisId` | Integer (FK → `AS_GSS_VendorFactorAnalysis.id`) | No | Parent factor analysis |
| `evaluationId` | Integer (FK → `AS_GSS_Evaluation.id`) | No | Parent evaluation (denormalised) |
| `vendorId` | Integer (FK → `AS_GSS_EvaluationVendor.id`) | No | Vendor (denormalised) |
| `criteriaId` | Integer (FK → `AS_GSS_Criteria.id`) | No | Factor (denormalised) |
| `category` | Text (500) | Yes | High-level category of the finding (e.g., "Deployment", "Scalability", "Architecture") — the "What" field |
| `findingText` | Text (4000) | No | AI-generated one-line factual summary of the finding |
| `actualText` | Text (16000) | Yes | Verbatim excerpt from the source document |
| `findingType` | Integer (FK → `AS_GSS_R_DATA.id`) | No | Reference data: `HIGHLIGHT`, `RISK`, `OBSERVATION` |
| `aiConfidence` | Integer (FK → `AS_GSS_R_DATA.id`) | No | Reference data: `HIGH`, `MEDIUM`, `LOW` |
| `sourceDocumentId` | Integer (FK → `AS_GSS_EvaluationDocument.id`) | Yes | Reference to the source document |
| `sourceDocumentName` | Text (500) | Yes | Document name (denormalised for display) |
| `sourceSection` | Text (500) | Yes | Section within the document |
| `sourcePage` | Text (100) | Yes | Page number or range (e.g., "12–14") |
| `sortOrder` | Integer | Yes | Display ordering within the factor |
| `createdBy` | Text (255) | No | Creator |
| `createdOn` | Timestamp | No | Creation timestamp |
| `modifiedBy` | Text (255) | Yes | Last modifier |
| `modifiedOn` | Timestamp | Yes | Last modification timestamp |

**Purpose:** Maps to the "Extracted Information Storage" story — stores individual findings with their sources, types, and confidence levels.

---

## 3. Reference Data Entries (AS_GSS_R_DATA)

New reference data values to be added to the existing `AS_GSS_R_DATA` reference table.

### 3.1 Extraction Type (`AS_GSS_REF_ID_EXTRACTION_TYPE_*`)

| Label | Constant Name | Description |
|---|---|---|
| Vendor Factor Analysis | `AS_GSS_REF_ID_EXTRACTION_TYPE_VENDOR_FACTOR_ANALYSIS` | AI extraction for a single vendor-factor combination |
| Vendor Overall Summary | `AS_GSS_REF_ID_EXTRACTION_TYPE_VENDOR_OVERALL_SUMMARY` | AI extraction for vendor-level overall summary and fitment |

### 3.2 Extraction / Analysis Status (`AS_GSS_REF_ID_ANALYSIS_STATUS_*`)

| Label | Constant Name | Description |
|---|---|---|
| Pending | `AS_GSS_REF_ID_ANALYSIS_STATUS_PENDING` | Analysis queued but not started |
| In Progress | `AS_GSS_REF_ID_ANALYSIS_STATUS_IN_PROGRESS` | AI extraction currently running |
| Completed | `AS_GSS_REF_ID_ANALYSIS_STATUS_COMPLETED` | Analysis finished successfully |
| Failed | `AS_GSS_REF_ID_ANALYSIS_STATUS_FAILED` | Analysis encountered an error |
| Regenerating | `AS_GSS_REF_ID_ANALYSIS_STATUS_REGENERATING` | Re-running analysis (e.g., after document update) |

### 3.3 Vendor Fitment (`AS_GSS_REF_ID_VENDOR_FITMENT_*`)

| Label | Constant Name | Score Threshold |
|---|---|---|
| Strong | `AS_GSS_REF_ID_VENDOR_FITMENT_STRONG` | Overall score > 0.75 |
| Moderate | `AS_GSS_REF_ID_VENDOR_FITMENT_MODERATE` | Overall score 0.45 – 0.74 |
| Weak | `AS_GSS_REF_ID_VENDOR_FITMENT_WEAK` | Overall score < 0.45 |

### 3.4 Finding Type (`AS_GSS_REF_ID_FINDING_TYPE_*`)

| Label | Constant Name | Description |
|---|---|---|
| Highlight | `AS_GSS_REF_ID_FINDING_TYPE_HIGHLIGHT` | Positive finding — vendor strength or differentiator |
| Risk | `AS_GSS_REF_ID_FINDING_TYPE_RISK` | Negative finding — gap, concern, or shortfall |
| Observation | `AS_GSS_REF_ID_FINDING_TYPE_OBSERVATION` | Neutral finding — factual note, neither positive nor negative |

### 3.5 AI Confidence (`AS_GSS_REF_ID_AI_CONFIDENCE_*`)

| Label | Constant Name | Description |
|---|---|---|
| High | `AS_GSS_REF_ID_AI_CONFIDENCE_HIGH` | Clear, explicit statements in source documents |
| Medium | `AS_GSS_REF_ID_AI_CONFIDENCE_MEDIUM` | Implied or partially supported |
| Low | `AS_GSS_REF_ID_AI_CONFIDENCE_LOW` | Inferred with limited supporting evidence |

### 3.6 AI Provider (`AS_GSS_REF_ID_AI_PROVIDER_*`)

| Label | Constant Name | Description |
|---|---|---|
| Azure OpenAI | `AS_GSS_REF_ID_AI_PROVIDER_AZURE_OPENAI` | Azure OpenAI Chat Completion |
| Appian Native AI | `AS_GSS_REF_ID_AI_PROVIDER_APPIAN_NATIVE` | Appian Generative AI Skill |

---

## 4. Audit Tables

Following the existing `AS_GSS_A_R_{Name}` / `AS_GSS_A_R_{Name}_Field` pattern, the following audit CDTs are required:

| Audit CDT | Audits |
|---|---|
| `AS_GSS_A_R_VendorAnalysisExtraction` | `AS_GSS_VendorAnalysisExtraction` |
| `AS_GSS_A_R_VendorAnalysisExtraction_Field` | Field-level audit for `AS_GSS_VendorAnalysisExtraction` |
| `AS_GSS_A_R_VendorAnalysis` | `AS_GSS_VendorAnalysis` |
| `AS_GSS_A_R_VendorAnalysis_Field` | Field-level audit for `AS_GSS_VendorAnalysis` |
| `AS_GSS_A_R_VendorFactorAnalysis` | `AS_GSS_VendorFactorAnalysis` |
| `AS_GSS_A_R_VendorFactorAnalysis_Field` | Field-level audit for `AS_GSS_VendorFactorAnalysis` |
| `AS_GSS_A_R_VendorAnalysisFinding` | `AS_GSS_VendorAnalysisFinding` |
| `AS_GSS_A_R_VendorAnalysisFinding_Field` | Field-level audit for `AS_GSS_VendorAnalysisFinding` |

---

## 5. Modifications to Existing Tables

### 5.1 `AS_GSS_Criteria` — Field Length Updates

| Column | Current | New | Reason |
|---|---|---|---|
| `description` | Text (255) | Text (4000) | Rich text support for solicitation requirements; feeds AI context |
| `instructions` | Text (255) | Text (4000) | Rich text support for evaluation guidance; feeds AI context |

**Related constant update:** `AS_GSS_ENUM_TEXT_LIMIT_FOR_EVALUATION_DESCRIPTION` from `255` → `4000`

**Note:** The synced record `AS_GSS_Criteria_SYNCEDRECORD` description mentions that Solicitation Description and Evaluation Description are 5000 chars but trimmed in the synced record. The MariaDB text field max is 16,000 characters (`AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS`), so 4000 is well within limits.

---

## 6. Entity Relationships Diagram

```
┌─────────────────────────┐
│   AS_GSS_Evaluation     │
│   (existing)            │
└──────────┬──────────────┘
           │ 1
           │
           │ N
┌──────────▼──────────────┐        ┌──────────────────────────┐
│ AS_GSS_EvaluationVendor │        │    AS_GSS_Criteria       │
│ (existing)              │        │    (existing)            │
└──────────┬──────────────┘        └──────────┬───────────────┘
           │ 1                                │
           │                                  │
           │ N                                │
┌──────────▼──────────────┐                   │
│  AS_GSS_VendorAnalysis  │                   │
│  (NEW)                  │                   │
│  - overallSummary       │                   │
│  - overallScore         │                   │
│  - fitment              │                   │
│  - confidenceScore      │                   │
│  - documentCount        │                   │
│  - analysisStatus       │                   │
└──────────┬──────────────┘                   │
           │ 1                                │
           │                                  │
           │ N                                │
┌──────────▼──────────────────────────────────▼───┐
│         AS_GSS_VendorFactorAnalysis             │
│         (NEW)                                   │
│  - factorSummary                                │
│  - factorScore / factorFitment                  │
│  - factorConfidence                             │
└──────────┬──────────────────────────────────────┘
           │ 1
           │
           │ N
┌──────────▼──────────────────────────────────────┐
│         AS_GSS_VendorAnalysisFinding            │
│         (NEW)                                   │
│  - category (What)                              │
│  - findingText                                  │
│  - actualText                                   │
│  - findingType (Highlight/Risk/Observation)     │
│  - aiConfidence (High/Medium/Low)               │
│  - sourceDocumentId / sourceSection / sourcePage│
└─────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────┐
│      AS_GSS_VendorAnalysisExtraction            │
│      (NEW)                                      │
│  - evaluationId / vendorId / criteriaId         │
│  - extractionType                               │
│  - promptIdentifier                             │
│  - status                                       │
│  - requestPayload / responsePayload             │
│  - aiProvider / tokenCount / durationMs         │
└─────────────────────────────────────────────────┘
  Links to: Evaluation, Vendor, Criteria
  Referenced by: VendorFactorAnalysis.extractionId
```

---

## 7. Relationship Summary

| Relationship | Cardinality | Description |
|---|---|---|
| `Evaluation` → `VendorAnalysis` | 1 : N | One evaluation has analysis for multiple vendors |
| `EvaluationVendor` → `VendorAnalysis` | 1 : 1 | One analysis record per vendor per evaluation |
| `VendorAnalysis` → `VendorFactorAnalysis` | 1 : N | One vendor analysis has multiple factor-level analyses |
| `Criteria` → `VendorFactorAnalysis` | 1 : N | One factor can appear in analyses for multiple vendors |
| `VendorFactorAnalysis` → `VendorAnalysisFinding` | 1 : N | One factor analysis contains multiple findings |
| `EvaluationDocument` → `VendorAnalysisFinding` | 1 : N | One document can be the source for multiple findings |
| `VendorAnalysisExtraction` → `VendorFactorAnalysis` | 1 : 1 | One extraction job produces one factor analysis result |
| `Evaluation` → `VendorAnalysisExtraction` | 1 : N | One evaluation triggers multiple extraction jobs |

---

## 8. Appian Object Naming (Following Existing Conventions)

### CDTs (Data Types)

| CDT Name | Table |
|---|---|
| `AS_GSS_VendorAnalysisExtraction` | `AS_GSS_VENDOR_ANALYSIS_EXTRACTION` |
| `AS_GSS_VendorAnalysis` | `AS_GSS_VENDOR_ANALYSIS` |
| `AS_GSS_VendorFactorAnalysis` | `AS_GSS_VENDOR_FACTOR_ANALYSIS` |
| `AS_GSS_VendorAnalysisFinding` | `AS_GSS_VENDOR_ANALYSIS_FINDING` |

### Synced Record Types

| Record Type Name | Source CDT |
|---|---|
| `AS_GSS_VendorAnalysisExtraction_SYNCEDRECORD` | `AS_GSS_VendorAnalysisExtraction` |
| `AS_GSS_VendorAnalysis_SYNCEDRECORD` | `AS_GSS_VendorAnalysis` |
| `AS_GSS_VendorFactorAnalysis_SYNCEDRECORD` | `AS_GSS_VendorFactorAnalysis` |
| `AS_GSS_VendorAnalysisFinding_SYNCEDRECORD` | `AS_GSS_VendorAnalysisFinding` |

### Entity Constants

| Constant Name | Description |
|---|---|
| `AS_GSS_ENT_VENDOR_ANALYSIS_EXTRACTION` | Data store entity for `AS_GSS_VendorAnalysisExtraction` |
| `AS_GSS_ENT_VENDOR_ANALYSIS` | Data store entity for `AS_GSS_VendorAnalysis` |
| `AS_GSS_ENT_VENDOR_FACTOR_ANALYSIS` | Data store entity for `AS_GSS_VendorFactorAnalysis` |
| `AS_GSS_ENT_VENDOR_ANALYSIS_FINDING` | Data store entity for `AS_GSS_VendorAnalysisFinding` |

### View CDT (for UI queries)

| CDT Name | Description |
|---|---|
| `AS_GSS_V_vendorAnalysisWithFindings` | Joins VendorAnalysis + VendorFactorAnalysis + Findings with vendor name, criteria name — used by the Vendor Analysis tab and Evaluator pane |

---

## 9. Scoring Logic Reference

Included here for implementers building the expression rules that compute scores after extraction.

### Per-Factor Scoring (`AS_GSS_BL_calculateFactorFitment`)

```
IF highlightCount >= 2
   AND highlightCount >= (riskCount * 2)
   AND riskCount <= 1
THEN factorScore = 1.0, fitment = STRONG

ELSE IF riskCount > highlightCount
   OR (highConfidenceRiskCount >= 2)
   OR (highlightCount = 0 AND riskCount = 0)
THEN factorScore = 0.25, fitment = WEAK

ELSE factorScore = 0.6, fitment = MODERATE
```

### Overall Vendor Scoring (`AS_GSS_BL_calculateVendorFitment`)

```
overallScore = SUM(factorScore[i] * factorWeight[i]) / SUM(factorWeight[i])

IF overallScore > 0.75 THEN fitment = STRONG
ELSE IF overallScore >= 0.45 THEN fitment = MODERATE
ELSE fitment = WEAK
```

### Confidence Rollup (`AS_GSS_BL_calculateVendorConfidence`)

```
Map aiConfidence to numeric: HIGH = 1.0, MEDIUM = 0.6, LOW = 0.3
factorConfidence = AVG(finding.aiConfidenceNumeric) for findings in factor
vendorConfidence = SUM(factorConfidence[i] * factorWeight[i]) / SUM(factorWeight[i])
```

---

## 10. Data Flow Summary

```
1. Evaluation Started (AS GSS Start Evaluation)
       │
       ▼
2. For each Vendor × Factor combination:
       │
       ├─ Create AS_GSS_VendorAnalysisExtraction (status = QUEUED)
       ├─ Gather inputs: Criteria.description, Criteria.instructions,
       │                 Vendor documents (via FactorDocumentMapping)
       ├─ Call AI (Azure OpenAI or Appian Native — same router pattern)
       ├─ Update Extraction (status = COMPLETED, store payload/metrics)
       ├─ Parse AI response → Write AS_GSS_VendorFactorAnalysis
       └─ Parse findings → Write AS_GSS_VendorAnalysisFinding[]
       │
       ▼
3. For each Vendor (after all factors complete):
       │
       ├─ Calculate overallScore (weighted sum of factorScores)
       ├─ Calculate confidenceScore (weighted avg of factorConfidence)
       ├─ Determine fitment (Strong/Moderate/Weak)
       ├─ Call AI for overall vendor summary
       └─ Write AS_GSS_VendorAnalysis
       │
       ▼
4. Display in UI:
       ├─ Vendor Analysis Tab (CO/Factor Chair) — reads VendorAnalysis + VendorFactorAnalysis + Findings
       └─ Evaluation Form Pane (Evaluator) — reads VendorFactorAnalysis + Findings for specific vendor-factor
```
