# Design Document: Factor/Subfactor Description & Instruction Rich Text Enhancement

## 1. Overview

This document details the design for enhancing the Factor and Subfactor Description and Instruction fields in the GAM Source Selection (GSS) application. The change replaces existing plain-text paragraph fields (`a!paragraphField`) with Appian Rich Text Editor components (`a!richTextEditorField`) for editable contexts, and `a!richTextDisplayField` for read-only contexts. The maximum character limit for these fields will be increased to 16,000 characters.

## 2. Current State Analysis

### 2.1 Current Component Usage

The Description and Instruction fields for Factors and Subfactors are currently rendered using:

- `AS_CO_INP_paragraphField` — a standard wrapper for `a!paragraphField()` used in editable mode
- `AS_CO_DSP_standardReadOnlyField` — a standard read-only text field for display mode
- `AS_GSS_CPS_richTextWithShowMoreLess` — a generic rich text display component with show more/less toggle (already exists and is widely used across 54 bundles)

Character limits are currently governed by:
- `AS_CO_ENUM_PARAGRAPH_LENGTH_LARGE` (value: 2000)
- `AS_CO_ENUM_PARAGRAPH_LENGTH_SHORT` (value: 255)

### 2.2 Impacted Appian Objects

The primary interface that renders the Factor/Subfactor fields in editable mode:

| Object | Type | Role |
|--------|------|------|
| `AS_GSS_CPS_evaluationFactorFields` | Interface | Contains all editable fields for factor and subfactor (Description, Instruction, Title, Due Date, etc.). Called by both `AS_GSS_CPS_evaluationFactorDetails` and `AS_GSS_CPS_evaluationSubFactorDetails`. |

Parent interfaces that embed the factor fields:

| Object | Type | Role |
|--------|------|------|
| `AS_GSS_CPS_evaluationFactorDetails` | Interface | Displays factor details with edit/delete options in Continue Setup wizard |
| `AS_GSS_CPS_evaluationSubFactorDetails` | Interface | Displays subfactor details with add/edit/delete options in Continue Setup wizard |
| `AS_GSS_CPS_evaluationCriteriaSetup` | Interface | Criteria setup step that hosts factor and subfactor detail components |

Read-only display interfaces impacted:

| Object | Type | Role |
|--------|------|------|
| `AS_GSS_CPS_viewFactors` | Interface | Read-only view of factors and subfactors on the Factors tab of the Evaluation Record |
| `AS_GSS_GRD_ViewFactorsAndSubfactors` | Interface | Grid displaying factors with expandable subfactors on the Factors tab |
| `AS_GSS_CPS_additionalEvaluationFactorDetailsForSummary` | Interface | Factor details on the Evaluation Summary tab (left panel) |
| `AS_GSS_CPS_additionalEvaluationSubfactorDetailsForSummary` | Interface | Subfactor details on the Evaluation Summary tab |
| `AS_GSS_FM_completeEvaluation` | Interface | Complete Evaluation task form |
| `AS_GSS_CPS_completeEvaluationHeader` | Interface | Header layout for Complete Evaluation task showing factor/subfactor info |
| `AS_GSS_FM_evaluationTaskForLpta` | Interface | LPTA Evaluation task form |
| `AS_GSS_CPS_lptaEvaluationTaskContentsPerVendor` | Interface | LPTA evaluation task contents per vendor |
| `AS_GSS_GRD_evaluationSubFactors` | Interface | Grid showing evaluation subfactors |

### 2.3 Data Layer

The Description and Instruction fields are stored in the `AS_GSS_Criteria` CDT, which maps to the `AS_GSS_CRITERIA` database table. The relevant fields are:
- `criteriaDescription` — Factor/Subfactor description
- `criteriaInstruction` — Factor/Subfactor instruction

The database column type for these fields must support up to 16,000 characters. The existing constant `AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS` (value: 16,000) already defines this limit, confirming the database `TEXT` column type can accommodate this.

## 3. Detailed Design

### 3.1 Editable Mode — Continue Setup Add/Edit Screen

#### 3.1.1 Interface: `AS_GSS_CPS_evaluationFactorFields`

This is the central interface that must be modified. The current Description and Instruction fields use `AS_CO_INP_paragraphField` with character count instructions via `AS_CO_UT_instructionForCharactersEntered`.

**Change:** Replace the `a!paragraphField` (via `AS_CO_INP_paragraphField`) calls for Description and Instruction with `a!richTextEditorField`.

**New component configuration for Description:**
```
a!richTextEditorField(
  label: /* existing label from bundle */,
  labelPosition: "ABOVE",
  value: ri!criteria.criteriaDescription,
  saveInto: ri!criteria.criteriaDescription,
  height: "MEDIUM",
  maxLength: 16000,
  placeholder: /* existing placeholder */,
  required: /* existing required logic */,
  readOnly: ri!readOnly,
  validations: /* updated validation for 16000 chars */
)
```

**New component configuration for Instruction:**
```
a!richTextEditorField(
  label: /* existing label from bundle */,
  labelPosition: "ABOVE",
  value: ri!criteria.criteriaInstruction,
  saveInto: ri!criteria.criteriaInstruction,
  height: "MEDIUM",
  maxLength: 16000,
  placeholder: /* existing placeholder */,
  readOnly: ri!readOnly,
  validations: /* updated validation for 16000 chars */
)
```

#### 3.1.2 Validation Updates

- Create a new constant `AS_GSS_ENUM_RICHTEXT_LENGTH_LARGE` with value 16000, or reuse the existing `AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS` constant (already set to 16,000).
- Update the character validation in `AS_GSS_CPS_evaluationFactorFields` to reference the 16,000 character limit instead of `AS_CO_ENUM_PARAGRAPH_LENGTH_LARGE` (2000).
- Remove or update the `AS_CO_UT_instructionForCharactersEntered` call for these fields, as the Rich Text Editor has built-in character count display via `maxLength`.

#### 3.1.3 Process Model: `AS GSS Continue Evaluation Setup`

The process model flow does not require changes. The user task "Evaluation wizard" calls `AS_GSS_FM_createOrUpdateEvaluation`, which in turn renders the criteria setup step. The data written to `AS_GSS_Criteria` DSE will now contain HTML-formatted rich text content instead of plain text. No process node changes are needed since the field types remain Text in the CDT.

### 3.2 Read-Only Mode — Multiple Screens

#### 3.2.1 Factors Tab (Evaluation Record Detail View)

**View:** `"Factors"` tab (url_stub: `_n_87YA`) on `AS_GSS_Evaluation_RECORD`

**Interfaces to update:**
- `AS_GSS_CPS_viewFactors` — Replace any `AS_CO_DSP_standardReadOnlyField` usage for Description/Instruction with `a!richTextDisplayField` that renders the stored HTML content.
- `AS_GSS_GRD_ViewFactorsAndSubfactors` — If Description is shown in the grid columns, update the column to use a rich text display component.

**Approach:** Use `a!richTextDisplayField` with the stored HTML value:
```
a!richTextDisplayField(
  value: /* criteria.criteriaDescription */
)
```

Since the existing `AS_GSS_CPS_richTextWithShowMoreLess` component is already used extensively (19 callers), leverage it for read-only display with show more/less truncation behavior. This component already calls `AS_CO_UT_left` for truncation and uses `AS_GSS_TOGGLE_AUTOMATION_TESTING_ENABLED` for test support.

#### 3.2.2 Evaluation Summary Tab

**Interfaces to update:**
- `AS_GSS_CPS_additionalEvaluationFactorDetailsForSummary` — Already calls `AS_GSS_CPS_richTextWithShowMoreLess` for description display. Verify it correctly renders HTML content. The text limit constant `AS_GSS_ENUM_TEXT_LIMIT_FOR_EVALUATION_DESCRIPTION` (value: 255) used for truncation in the show more/less component should be reviewed — the truncation logic in `AS_CO_UT_left` must handle HTML tags properly to avoid breaking markup mid-tag.
- `AS_GSS_CPS_additionalEvaluationSubfactorDetailsForSummary` — Same pattern as above. Also calls `AS_GSS_CPS_richTextWithShowMoreLess`.

**Key consideration:** The `AS_GSS_CPS_richTextWithShowMoreLess` component truncates text using `AS_CO_UT_left`. When the content is HTML (from the rich text editor), truncating by character count can break HTML tags. The truncation logic must be updated to either:
1. Strip HTML tags before calculating truncation length, then display full rich text when expanded
2. Use `a!richTextDisplayField` with a CSS-based truncation approach
3. Convert rich text to plain text for the truncated preview using `AS_GSS_UT_convertRichTextToPlainText` (this expression rule already exists in the application)

**Recommended approach:** Use `AS_GSS_UT_convertRichTextToPlainText` for the collapsed/truncated preview, and render the full rich text HTML when the user clicks "Show More."

#### 3.2.3 Complete Evaluation Task

**Process:** `AS GSS Complete Evaluation` → User Task "Complete Evaluation task" → Interface `AS_GSS_FM_completeEvaluation`

**Interfaces to update:**
- `AS_GSS_FM_completeEvaluation` — The task form displays factor/subfactor information in read-only mode. Update Description and Instruction display to use `a!richTextDisplayField`.
- `AS_GSS_CPS_completeEvaluationHeader` — Already calls `AS_GSS_CPS_richTextWithShowMoreLess`. Verify HTML rendering.

**LPTA variant:**
- `AS_GSS_FM_evaluationTaskForLpta` — Same treatment for LPTA evaluation task form.
- `AS_GSS_CPS_lptaEvaluationTaskContentsPerVendor` — Update factor description display.

#### 3.2.4 Subfactor Grid

- `AS_GSS_GRD_evaluationSubFactors` — Already calls `AS_GSS_CPS_richTextWithShowMoreLess`. Verify HTML rendering in grid context.

### 3.3 Shared Component Updates

#### 3.3.1 `AS_GSS_CPS_richTextWithShowMoreLess`

This is the most widely used display component (54 bundles, 19 direct callers). It must be updated to properly handle HTML content from the rich text editor:

**Current behavior:** Displays text with truncation using `AS_CO_UT_left` and a show more/less toggle.

**Required changes:**
1. When content is HTML, use `AS_GSS_UT_convertRichTextToPlainText` for the truncated preview
2. When expanded, render the full HTML content using `a!richTextDisplayField`
3. Maintain backward compatibility for callers that still pass plain text

#### 3.3.2 New Constant (if needed)

| Constant | Value | Description |
|----------|-------|-------------|
| `AS_GSS_ENUM_RICHTEXT_FIELD_MAX_CHARACTERS` | 16000 | Maximum character limit for rich text fields (Description, Instruction). Alternatively, reuse `AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS`. |

### 3.4 Database Changes

The `AS_GSS_CRITERIA` table columns `CRITERIA_DESCRIPTION` and `CRITERIA_INSTRUCTION` must be of type `TEXT` (or equivalent) to support up to 16,000 characters of HTML content. If these columns are currently `VARCHAR(2000)`, they must be altered to `TEXT`.

**Migration script:**
```sql
ALTER TABLE AS_GSS_CRITERIA MODIFY COLUMN CRITERIA_DESCRIPTION TEXT;
ALTER TABLE AS_GSS_CRITERIA MODIFY COLUMN CRITERIA_INSTRUCTION TEXT;
```

The `AS_GSS_Criteria` CDT field types remain `Text` in Appian (Appian Text type supports arbitrary length when backed by a `TEXT` database column).

The `AS_GSS_Evaluation_SYNCEDRECORD` note states: "Solicitation Description and Evaluation Description are 5000 characters and will be trimmed in the synced record." Verify that the `AS_GSS_Criteria_SYNCEDRECORD` does not have similar trimming that would truncate the new 16,000-character content.

### 3.5 Audit Considerations

The audit configuration `AS_GSS_ADT_BL_auditConfig_Criteria` tracks changes to the `AS_GSS_Criteria` CDT. Since Description and Instruction are simple text fields in the audit config, the auditing module will continue to work. However, the audit trail will now store HTML content in the old/new values. The audit display interfaces (`AS_GSS_DSP_displayAuditCommentForEvaluationCdts`) should be reviewed to ensure HTML content renders correctly or is stripped for display.

## 4. Impact Analysis

### 4.1 Bundles Impacted

| Bundle | Type | Impact |
|--------|------|--------|
| `AS_GSS_Evaluation_RECORD - Continue Setup` | Action (234 objects) | Primary — editable factor fields |
| `AS_GSS_Evaluation_RECORD - Update Factors` | Action | Editable factor fields in update flow |
| `AS_GSS_Evaluation_RECORD` | Page (575 objects) | Factors tab, Summary tab read-only display |
| `AS_GSS_Complete_Evaluation` | Process (350 objects) | Complete Evaluation task read-only display |
| `AS_GSS_Complete_LPTA_Evaluation` | Process (100 objects) | LPTA Evaluation task read-only display |
| `AS_GSS_Evaluation_RECORD - Complete Factor` | Action (116 objects) | Factor completion read-only display |

### 4.2 Downstream Dependencies of `AS_GSS_CPS_evaluationFactorFields`

This interface is called by:
- `AS_GSS_CPS_evaluationFactorDetails` → called by `AS_GSS_CPS_evaluationCriteriaSetup`
- `AS_GSS_CPS_evaluationSubFactorDetails` → called by `AS_GSS_CPS_evaluationCriteriaSetup`

All six bundles that include `AS_GSS_CPS_evaluationFactorFields` will inherit the change:
- `AS_GSS_Evaluation_RECORD - Continue Setup`
- `AS_GSS_Evaluation_RECORD - Update Factors` (×4 variants)
- `AS_GSS_Evaluation_RECORD - Add Teams`

### 4.3 Document Generation Impact

The process `AS GSS Generate Vendor Factor Summary Document` uses factor descriptions to generate documents. The expression rule `AS_GSS_UT_formatEvaluationFactorSummaryCommentsSection` and `AS_GSS_UT_createDynamicTableSectionForVendorRatingDocument` must be reviewed to handle HTML content — either by rendering it in the document or stripping HTML tags using `AS_GSS_UT_convertRichTextToPlainText`.

## 5. Test Scenarios

### 5.1 Continue Setup — Add/Edit Screen (Editable)

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Open Continue Setup → Add new Factor → Verify Description field | Rich Text Editor component is displayed with formatting toolbar |
| 2 | Open Continue Setup → Add new Factor → Verify Instruction field | Rich Text Editor component is displayed with formatting toolbar |
| 3 | Enter text with bold, italic, underline, bullet list formatting in Description | Formatting is preserved on save |
| 4 | Enter exactly 16,000 characters in Description | Accepted without validation error |
| 5 | Enter 16,001 characters in Description | Validation error displayed |
| 6 | Open Continue Setup → Edit existing Factor → Verify Description loads with formatting | Previously saved rich text formatting is displayed correctly |
| 7 | Open Continue Setup → Add new Subfactor → Verify Description and Instruction fields | Rich Text Editor components displayed for both fields |
| 8 | Open Continue Setup → Edit existing Subfactor → Verify fields | Rich text content loads correctly with formatting preserved |

### 5.2 Factors Tab — Read-Only Display

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Navigate to Evaluation Record → Factors tab | Factor descriptions display as rich text (bold, italic, lists rendered correctly) |
| 2 | Expand a factor to view subfactors | Subfactor descriptions display as rich text |
| 3 | Verify show more/less behavior for long descriptions | Truncated preview shows plain text; expanded view shows full rich text |

### 5.3 Complete Evaluation Task

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Open Complete Evaluation task | Factor/Subfactor Description and Instruction display as rich text in read-only mode |
| 2 | Open Complete LPTA Evaluation task | Factor/Subfactor Description and Instruction display as rich text in read-only mode |
| 3 | Verify formatting (bold, italic, lists) renders correctly | All rich text formatting is visible |

### 5.4 Evaluation Summary Tab

| # | Scenario | Expected Result |
|---|----------|-----------------|
| 1 | Navigate to Evaluation Record → Summary tab | Factor descriptions in the summary panel display as rich text |
| 2 | Click on a subfactor link in summary | Subfactor description displays as rich text |
| 3 | Verify show more/less for long descriptions | Truncated preview is plain text; expanded shows full rich text |

## 6. Implementation Sequence

1. **Database migration** — Alter `CRITERIA_DESCRIPTION` and `CRITERIA_INSTRUCTION` columns to `TEXT` type
2. **Constant creation** — Create `AS_GSS_ENUM_RICHTEXT_FIELD_MAX_CHARACTERS` (16000) or confirm reuse of `AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS`
3. **Core edit interface** — Update `AS_GSS_CPS_evaluationFactorFields` to use `a!richTextEditorField` for Description and Instruction
4. **Shared display component** — Update `AS_GSS_CPS_richTextWithShowMoreLess` to handle HTML content with proper truncation
5. **Factors tab interfaces** — Update `AS_GSS_CPS_viewFactors`, `AS_GSS_GRD_ViewFactorsAndSubfactors`
6. **Summary tab interfaces** — Verify `AS_GSS_CPS_additionalEvaluationFactorDetailsForSummary`, `AS_GSS_CPS_additionalEvaluationSubfactorDetailsForSummary`
7. **Evaluation task interfaces** — Update `AS_GSS_FM_completeEvaluation`, `AS_GSS_CPS_completeEvaluationHeader`, `AS_GSS_FM_evaluationTaskForLpta`
8. **Subfactor grid** — Verify `AS_GSS_GRD_evaluationSubFactors`
9. **Document generation** — Update `AS_GSS_UT_formatEvaluationFactorSummaryCommentsSection` to handle HTML
10. **Audit display** — Review `AS_GSS_DSP_displayAuditCommentForEvaluationCdts` for HTML content handling
11. **Testing** — Execute all test scenarios above
