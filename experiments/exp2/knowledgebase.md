# Proposal Analyser — Source Selection Knowledgebase

This document maps the Proposal Analyser feature (spec v2.7) to the existing Source Selection v2.6.0 Appian application. It captures the existing record types, interfaces, process models, expression rules, and data relationships that the new feature will integrate with or extend.

---

## 1. Application Overview

- Package: SourceSelectionv2.6.0.zip
- Total objects: 2,327 (950 Expression Rules, 472 Interfaces, 108 Process Models, 48 Record Types, 107 CDTs, 522 Constants, 14 Integrations, 6 Connected Systems, 4 Web APIs, 3 Sites)
- Bundled: 1,790 | Orphaned: 537

---

## 2. Core Record Types Relevant to Proposal Analyser

### 2.1 Evaluation Record

| Object | UUID | Description | Deps In / Out |
|---|---|---|---|
| `AS_GSS_Evaluation_SYNCEDRECORD` | `e6bc8561-...` | Main synced record for evaluations. Note: Solicitation Description and Evaluation Description are 5000 chars, trimmed in synced record. | 150 / 19 |
| `AS_GSS_Evaluation_RECORD` | `4db4a62e-...` | Record type for evaluations (views, actions, summary pages). 78 outbound deps — the most connected object in the app. | 22 / 78 |

Key relationships from `AS_GSS_Evaluation_SYNCEDRECORD`:
- Links to `AS_GSS_EvaluationVendor_SYNCEDRECORD` (vendors)
- Links to `AS_GSS_Criteria_SYNCEDRECORD` (factors/subfactors)
- Links to `AS_GSS_EvaluationDocument_SYNCEDRECORD` (documents)
- Links to `AS_GSS_EvaluationPhase_SYNCEDRECORD` (phases)
- Links to `AS_GSS_TMG_Task_SYNCEDRECORD` (tasks)
- Links to `AS_GSS_ConsensusReport_SYNCEDRECORD` (consensus)

### 2.2 Vendor Records

| Object | UUID | Description | Deps In / Out |
|---|---|---|---|
| `AS_GSS_EvaluationVendor_SYNCEDRECORD` | `b6081510-...` | Synced record for evaluation vendors. Present in 69 bundles. | 94 / 24 |
| `AS_GSS_EvaluationVendorAddress_SYNCEDRECORD` | `c4e07a80-...` | Vendor address data | 4 / 3 |
| `AS_GSS_EvaluationVendorBusinessType_SYNCEDRECORD` | `28b193af-...` | Vendor business type data | 5 / 1 |
| `AS_GSS_VendorPriceBreakup_SYNCEDRECORD` | `e29f01c8-...` | Pricing line items per vendor | 15 / 2 |

Key vendor-related expression rules:
- `AS_GSS_QR_getEvaluationVendors` — query vendors for an evaluation
- `AS_GSS_QR_getEvaluationVendorByIdentifier` — query single vendor
- `AS_GSS_mapDataFromVMToEvalVendorRecord` — map vendor data from Vendor Management
- `AS_GSS_formatVendorInfoForCards` — format vendor info for card display
- `AS_GSS_constructSelectedVendorsMap` — build vendor selection map

### 2.3 Factor / Criteria Records

| Object | UUID | Description | Deps In / Out |
|---|---|---|---|
| `AS_GSS_Criteria_SYNCEDRECORD` | `11dcc745-...` | Synced record for evaluation criteria (factors/subfactors). Present in 63 bundles. | 73 / 8 |
| `AS_GSS_CriteriaAssignment_SYNCEDRECORD` | `518a0b4d-...` | Criteria-to-evaluator assignments | 20 / 1 |
| `AS_GSS_R_Rating_SYNCEDRECORD` | (linked from Criteria) | Reference ratings | — |
| `AS_GSS_Rating_SYNCEDRECORD` | (linked from Criteria) | Transactional ratings | — |

Criteria relationships:
- `AS_GSS_Criteria_SYNCEDRECORD` → `AS_GSS_R_Rating_SYNCEDRECORD` (reference ratings)
- `AS_GSS_Criteria_SYNCEDRECORD` → `AS_GSS_EvaluatorTeam_SYNCEDRECORD` (team assignments)
- `AS_GSS_Criteria_SYNCEDRECORD` → `AS_GSS_CriteriaAssignment_SYNCEDRECORD` (evaluator assignments)
- `AS_GSS_Criteria_SYNCEDRECORD` → `AS_GSS_Evaluation_SYNCEDRECORD` (parent evaluation)
- `AS_GSS_Criteria_SYNCEDRECORD` → `AS_GSS_ConsensusReport_SYNCEDRECORD` (consensus)

Key factor expression rules:
- `AS_GSS_QR_getCriteria` — query criteria/factors
- `AS_GSS_QR_getCriteriaAssignment` — query criteria assignments
- `AS_GSS_UT_constructAllFactorAssignmentInformation` — build factor assignment info
- `AS_GSS_UT_returnsSumOfFactorsWeight` — sum factor weights
- `AS_GSS_DEC_FACTOR_WEIGHT_TOTAL_POINTS` — max factor weight points constant

### 2.4 Document Records

| Object | UUID | Description | Deps In / Out |
|---|---|---|---|
| `AS_GSS_EvaluationDocument_SYNCEDRECORD` | `9c497e08-...` | Evaluation documents. Present in 61 bundles. | 49 / 10 |
| `AS_GSS_FactorDocumentMapping_SYNCEDRECORD` | `acd503e1-...` | Maps documents to factors. Present in 24 bundles. | 17 / 2 |
| `AS_GSS_DeletedDocument_SYNCEDRECORD` | — | Tracks deleted documents | — |

Document relationships:
- `AS_GSS_EvaluationDocument_SYNCEDRECORD` → `AS_GSS_FactorDocumentMapping_SYNCEDRECORD` (factor-doc mapping)
- `AS_GSS_EvaluationDocument_SYNCEDRECORD` → `AS_GSS_EvaluationVendor_SYNCEDRECORD` (vendor association)
- `AS_GSS_EvaluationDocument_SYNCEDRECORD` → `AS_GSS_Criteria_SYNCEDRECORD` (factor association)

Key document expression rules:
- `AS_GSS_QR_getEvaluationDocuments` — query evaluation documents
- `AS_GSS_QE_getEvaluationDocuments` — entity query for documents
- `AS_GSS_QE_getEvaluationFactorDocumentMapping` — factor-document mapping query
- `AS_GSS_BL_defaultFactorDocumentMapping` — default factor-doc mapping logic

Document type constants:
- `AS_GSS_REF_ID_DOC_TYPE_VENDOR` = 24
- `AS_GSS_REF_ID_DOC_TYPE_FACTOR` = 25
- `AS_GSS_REF_ID_DOC_TYPE_EVALUATOR` = 26
- `AS_GSS_REF_ID_DOC_TYPE_CONSENSUS` = 27
- `AS_GSS_REF_ID_DOC_TYPE_RECOMMENDATION` = 28
- `AS_GSS_REF_ID_DOC_TYPE_REFERENCE` = 58

### 2.5 Evaluation Responses

| Object | UUID | Description |
|---|---|---|
| `AS_GSS_EvaluationResponses_SYNCEDRECORD` | `553858bd-...` | Synced record for evaluation responses (strengths/weaknesses/deficiencies) |

Response type constants:
- `AS_GSS_REF_ID_RESPONSE_TYPE_STRENGTH` = 42
- `AS_GSS_REF_ID_RESPONSE_TYPE_WEAKNESS` = 43
- `AS_GSS_REF_ID_RESPONSE_TYPE_DEFICIENCY` = 44

---

## 3. Evaluation Summary — Existing UI Components

The Proposal Analyser's "Vendor Analysis" tab will be added to the Evaluation Summary. Here are the existing summary interfaces:

| Interface | Description | Deps Out |
|---|---|---|
| `AS_GSS_FM_evaluationSummary` | Main summary interface for Evaluation. Called by `AS_GSS_Evaluation_RECORD`. | 23 |
| `AS_GSS_CPS_evaluationInformationForLeftPanelOfSummary` | Left panel of summary (evaluation info) | — |
| `AS_GSS_CPS_evaluationInformationForRightPanelOfSummary` | Right panel of summary (factor details, vendor ratings) | — |
| `AS_GSS_CPS_evaluationDetailsForSummary` | Evaluation details section | — |
| `AS_GSS_CPS_evaluationFactorDetailsForSummary` | Factor details on summary left side | 18 |
| `AS_GSS_CPS_additionalEvaluationFactorDetailsForSummary` | Additional factor details | 13 |
| `AS_GSS_CPS_mainEvaluationSubfactorDetailsForSummary` | Subfactor details on summary | 14 |
| `AS_GSS_GRD_subFactorGridforEvaluationSummary` | Subfactor grid per factor | 21 |
| `AS_GSS_CPS_vendorsInfoForEvaluationSummary` | Vendors info section in summary | 14 |
| `AS_GSS_CPS_factorWeightDetailsforSummary` | Factor weight pie chart | 18 |
| `AS_GSS_CPS_vendorRatingsPerFactor` | Vendor rating pie charts per factor/subfactor | 17 |
| `AS_GSS_SCT_evaluationSummaryPhases` | Evaluation phases read-only display | 10 |

The `AS_GSS_FM_evaluationSummary` interface calls:
- `AS_GSS_QR_getEvaluation` — fetch evaluation data
- `AS_GSS_UT_updateEvaluationRecordWithAllVendors` — load all vendors
- `AS_GSS_UT_constructAllFactorAssignmentInformation` — load factor assignments
- `AS_GSS_BL_validateMissingInformation` — check missing setup info
- `AS_GSS_BL_determineIfUserIsEvaluator` — role check
- Security groups: `AS_GSS_GRP_CONTRACTING_PERSONNEL`, `AS_GSS_GRP_EVALUATION_CHAIRS`

---

## 4. Evaluation Task Form — Existing Components

The evaluator's task form where the analysis pane will be added:

| Interface | Description |
|---|---|
| `AS_GSS_FM_completeEvaluation` | Main task UI for Complete Evaluation (29 deps out) |
| `AS_GSS_CPS_completeEvaluationHeader` | Header layout |
| `AS_GSS_CPS_evalDocsForCompleteEvaluation` | Document display in evaluation task |
| `AS_GSS_CPS_evaluationResponses` | Strengths/weaknesses/deficiencies section |
| `AS_GSS_CPS_finalRatingsForCompleteEvaluation` | Final rating selection |
| `AS_GSS_CPS_ratingJustification` | Rating justification paragraph |
| `AS_GSS_BOX_editEvaluationResponse` | Edit response box |
| `AS_GSS_CRD_addEvaluationResponse` | Add response card |
| `AS_GSS_SCT_evaluationTaskDocumentDisplay` | Document display in task |
| `AS_GSS_SCT_evaluationDocumentsList` | Documents list section |

Process flow for Complete Evaluation:
```
Start → Set PVs → Complete Evaluation task (USER_TASK, form: AS_GSS_FM_completeEvaluation)
  → Docs to Delete empty? → [Delete docs] → Join
  → Write Evaluation Task (subprocess: AS GSS Write Evaluation Task)
  → End
```

Write Evaluation Task subprocess flow:
```
Start → Cancel/Eval Docs null?
  → Update Eval & delete docs → Eval Docs null?
    → Write to Eval doc & Capture Audit (GAM ADT Audit Process)
    → Delete Eval Docs null? → [Write Deleted Eval docs]
  → Update evaluationResponse and meta data
  → Is Evaluation Response blank?
    → Write to Evaluation Response → Capture Evaluation Responses audit
  → User action?
    → Save Draft: Update task → Write to Task → Audit → End
    → Submit: Task Completion → End
```

---

## 5. Existing AI Infrastructure

### 5.1 AI Summarization Process Models

| Process Model | Description |
|---|---|
| `AS GSS Summarize Responses Using AI` | Entry point: generates summary for consensus responses per vendor. Calls AI Summarizer → writes to AI Integration Log → captures metrics. |
| `AS GSS Summarize With AI` | Router: decides between Azure OpenAI or Appian Native AI based on toggle. |
| `AS GSS Summarize With Azure OpenAI` | Calls Azure Open AI Chat Completion integration. |
| `AS GSS Summarize With Native AI` | Executes Appian Generative AI Skill. |
| `AS GSS Combine Responses Using AI` | Combines multiple response summaries. |

### 5.2 AI Toggle & Metrics

- `AS_GSS_EnableOrDisableAppianAIForSummarization_wrapper` — toggle for native vs Azure AI
- `AS_GSS_ENT_CONSENSUS_AI_SUGGESTION` — entity for AI suggestions
- Metrics rules for tracking success/failure of AI calls (both Azure and Native)

### 5.3 Document Chat (existing AI feature)

- `AS_GSS_CPS_chatWithDocument` — interface for document chat
- `AS_GSS_UT_selectedDocumentSavesForDocChat` — document selection for chat
- `AS_GSS_UT_selectedExtDocumentSavesForDocChat` — external document selection

---

## 6. Factor/Subfactor Setup Interfaces

These are the interfaces used during evaluation setup for factors — relevant because the Proposal Analyser reads factor descriptions and instructions:

| Interface | Description |
|---|---|
| `AS_GSS_CPS_evaluationFactorFields` | Fields for factor details (description, instructions). 29 deps out — heavily connected. |
| `AS_GSS_CPS_evaluationFactorDetails` | Factor details display with edit/delete |
| `AS_GSS_CPS_evaluationSubFactorDetails` | Subfactor details display |
| `AS_GSS_CPS_evaluationFactorWeights` | Factor weights step in wizard |
| `AS_GSS_BOX_parentFactorWeightInputs` | Factor weight input boxes |
| `AS_GSS_INP_factorWeight` | Individual factor weight input |
| `AS_GSS_CPS_assignFactors` | Factor assignment step |
| `AS_GSS_GRD_ViewFactorsAndSubfactors` | Grid to view factors and expandable subfactors |

---

## 7. Key Constants for the Feature

### Evaluation Status
- `AS_GSS_REF_ID_EVALUATION_STATUS_SETTING_UP` = 1
- `AS_GSS_REF_ID_EVALUATION_STATUS_INPROGRESS` = 2
- `AS_GSS_REF_ID_EVALUATION_STATUS_COMPLETE` = 3
- `AS_GSS_REF_ID_EVALUATION_STATUS_AWARDEES_SELECTED` = 79

### Security Groups
- `AS_GSS_GRP_CONTRACTING_PERSONNEL` — CO/CS/CM group
- `AS_GSS_GRP_EVALUATION_CHAIRS` — Evaluation Chairs (Factor Chairs)
- `AS_GSS_GRP_FACTOR_ADVISOR` — Factor Advisor group
- `AS_GSS_GRP_ALL_BUSINESS_USERS` — All business users

### Field Limits
- `AS_GSS_ENUM_TEXT_LIMIT_FOR_EVALUATION_DESCRIPTION` = 255 (current, spec wants 4000)
- `AS_GSS_INT_MARIADB_TEXT_FIELD_MAX_CHARACTERS` = 16,000 (MariaDB text field max)

---

## 8. Start Evaluation Process — Analysis Trigger Point

The Proposal Analyser analysis kicks off when an evaluation is started. The existing `AS GSS Start Evaluation` process model (bundle: `AS_GSS_Evaluation_RECORD_-_Start_Evaluation`) is the hook point.

### Process Flow

```
Start → Is cancel?
  → [No] Update Evaluation Record → Sync Eval Status in GCW
    → Populate Parent And Child Ratings → Write Rating Records
    → Update Factors And SubFactors With Ratings
    → Identify Workflow:
        → LPTA: Generate LPTA Task → Capture Audit → End
        → Standard: Generate Evaluation Tasks → Create Consensus Reports → Capture Audit → End
        → On The Spot Consensus: Create Consensus Reports → Capture Audit → End
  → [Cancel] End
```

### Key Dependencies

| Object | Type | Role |
|---|---|---|
| `AS_GSS_FM_startEvaluation` | Interface | Confirmation screen for starting evaluation |
| `AS_GSS_BL_getRelatedActionVisibilityForStartEvaluation` | Expression Rule | Controls visibility of Start Evaluation action |
| `AS GSS Generate Evaluation Tasks` | Process Model | Creates tasks per evaluator-factor assignment |
| `AS GSS Create Consensus Report` | Process Model | Creates consensus report records |
| `AS_GSS_REF_ID_EVALUATION_METHOD_LPTA` | Constant | Value: 67 — used to branch LPTA vs standard workflow |

### Where to Hook Analysis

The analysis process should be triggered after the "Identify Workflow" gateway for non-LPTA evaluations, either:
- As a parallel subprocess alongside "Generate Evaluation Tasks", or
- As a new subprocess after task generation completes

The context data loaded in the Start Evaluation action's CONTEXT expression already fetches: evaluation record, all factors/subfactors (`AS_GSS_QR_getCriteria`), and all vendors (`AS_GSS_QR_getEvaluationVendors`).

---

## 9. CDT Structures (Underlying Data Types)

These are the core CDTs backing the synced record types. Relevant for building new CDTs and understanding field structures.

### Transactional CDTs

| CDT | Description |
|---|---|
| `AS_GSS_Evaluation` | Core evaluation data type (description, status, dates, etc.) |
| `AS_GSS_EvaluationVendor` | Vendor data per evaluation |
| `AS_GSS_Criteria` | Factor/subfactor data (description, instructions, weight, parent-child) |
| `AS_GSS_Criteria_Assignments` | Maps criteria to evaluators |
| `AS_GSS_EvaluationDocument` | Document metadata (type, vendor, factor associations) |
| `AS_GSS_Evaluation_Responses` | Evaluator responses (strengths/weaknesses/deficiencies) |
| `AS_GSS_EvaluationPhase` | Evaluation phase tracking |
| `AS_GSS_Consensus_Report` | Consensus report data |
| `AS_GSS_Consensus_Response` | Individual consensus responses |
| `AS_GSS_EvaluationComments` | Award comments |
| `AS_GSS_EvaluationVendor_BusinessType` | Vendor business type data |

### AI-Related CDTs (Pattern for New CDTs)

| CDT | Description |
|---|---|
| `AS_GSS_ConsensusAiSuggestion` | Stores AI prompt and response for consensus suggestions |
| `AS_GSS_ConsensusResponseAiSummary` | Stores AI-generated summary of consensus responses |

These two CDTs are the existing pattern for storing AI results. The new Vendor Analysis feature should follow a similar pattern for its finding/score storage CDTs.

### Audit CDTs

Each transactional CDT has corresponding audit CDTs following the pattern:
- `AS_GSS_A_R_{Name}` — audit record
- `AS_GSS_A_R_{Name}_Field` — field-level audit

New CDTs for Vendor Analysis will need corresponding audit CDTs.

### View CDTs

| CDT | Description |
|---|---|
| `AS_GSS_V_evaluationDocAdditionalInfo` | Joins document table with vendor name, criteria name, and ref data — useful pattern for analysis views |

---

## 10. GCW (Contract Writing) Cross-Application Integration

The spec requires pulling solicitation data into Contract Writing. Here are the existing cross-app references between Source Selection and GCW:

### Existing GCW App References

| Object | Description |
|---|---|
| `AS_GSS_APPREF_GCW_GETDATA_getLatestOrBaseSolicitationDetails` | Fetches solicitation details from GCW (9 bundles) |
| `AS_GSS_APPREF_GCW_GETDATA_getSolicitationDataForDraftingEvaluation` | Fetches solicitation data for evaluation creation |
| `AS_GSS_APPREF_GCW_DISPLAY_relatedProcurementDetailsForEvalSummary` | Displays procurement details in evaluation summary |
| `AS_GSS_APPREF_GCW_STARTPROCESS_updateSolicPublicFolderSecurityOnCreateEval` | Updates solicitation folder security |
| `AS_GSS_APPREF_GCW_STARTPROCESS_updateEvalSolicMapping` | Updates evaluation-solicitation mapping |
| `AS_GSS_BL_exposeGcwIntegration` | Boolean toggle for GCW integration visibility |
| `AS_GSS_REF_ID_SOURCE_APPLICATION_GCW` | Constant: 60 — identifies GCW as source application |

### Solicitation Mapping

| Object | Description |
|---|---|
| `AS_GSS_UT_mapSolicitationToEvaluationFromGcw` | Maps solicitation data to evaluation (15 deps out) |
| `AS_GSS_UT_mapSolicitationToEvaluationFromAm` | Maps solicitation data from AM (Acquisition Management) |
| `AS_GSS_UT_mapSolicitationToEvaluationWrapper` | Router: determines GCW vs AM mapping |
| `AS GSS Update Evaluation Solicitation Mapping` | Process model to update eval-solic mapping table |

### Contract Writing Integration Notes

The new "Contract Text" data points (description/specifications, instructions/conditions, evaluation factors) will need new `APPREF` expression rules following the existing pattern. The GCW integration is gated by `AS_GSS_BL_exposeGcwIntegration` and the `AS GCW GSS Access Group` security group.

---

## 11. AI Infrastructure — Detailed Integration Objects

### Connected System

| Object | Type | Description |
|---|---|---|
| `AS GSS Azure OpenAI` | Connected System | Azure OpenAI connected system for GSS |

### Integration Object

| Object | Description |
|---|---|
| `AS_GSS_INT_summarize_ConsensusResponsesOrFinalComment` | Azure OpenAI Chat Completion integration. Uses `AS_GSS_SUGGEST_FINAL_COMMENT_PROMPT` constant for prompt template. Only integration currently using the Azure OpenAI connected system. |

### AI Process Model Chain

```
AS GSS Summarize Responses Using AI (entry point, 13 objects)
  → AS GSS Summarize With AI (router, 4 objects)
      → Toggle check: AS_GSS_EnableOrDisableAppianAIForSummarization_wrapper
      → Azure path: AS GSS Summarize With Azure OpenAI (2 objects)
          → Uses: AS_GSS_INT_summarize_ConsensusResponsesOrFinalComment
          → Uses: AS GSS Azure OpenAI (Connected System)
      → Native path: AS GSS Summarize With Native AI (1 object)
          → Uses: Appian Generative AI Skill
```

### Reuse Pattern for Proposal Analyser

The new analysis feature should follow this same router pattern:
1. Create a new integration object for the analysis prompt (or extend the existing one)
2. Create a new process model chain: `AS GSS Analyse Vendor Proposals` → `AS GSS Analyse With AI` (router) → Azure/Native paths
3. Reuse the toggle mechanism (`AS_GSS_EnableOrDisableAppianAIForSummarization_wrapper`) or create a separate toggle for analysis

---

## 12. Additional Constants

### Evaluation Method

| Constant | Value | Description |
|---|---|---|
| `AS_GSS_REF_ID_EVALUATION_METHOD_LPTA` | 67 | LPTA evaluation method — analysis likely N/A for LPTA |

### Evaluation Status (Extended)

| Constant | Value | Description |
|---|---|---|
| `AS_GSS_REF_ID_EVALUATION_STATUS_TEMP` | 34 | Temporary/draft status |

### Task Behavior Subtypes (Relevant)

| Constant | Value | Description |
|---|---|---|
| `AS_GSS_TMG_ENUM_TASK_BEHAVIOR_SUBTYPE_CODE_COMPLETE_EVALUATION` | — | Complete Evaluation task type |
| `AS_GSS_TMG_ENUM_TASK_BEHAVIOR_SUBTYPE_CODE_COMPLETE_LPTA_EVALUATION` | — | Complete LPTA Evaluation task type |

### Task Status

| Constant | Value |
|---|---|
| `AS_GSS_TMG_REF_ID_TASK_STATUS_QUEUED` | 1 |
| `AS_GSS_TMG_REF_ID_TASK_STATUS_ASSIGNED` | 2 |
| `AS_GSS_TMG_REF_ID_TASK_STATUS_IN_PROGRESS` | 3 |
| `AS_GSS_TMG_REF_ID_TASK_STATUS_COMPLETE` | 4 |
| `AS_GSS_TMG_REF_ID_TASK_STATUS_NOT_NEEDED` | 5 |

---

## 13. Integration Points

### 13.1 What Needs to Be Created (New)

1. **New Record Type(s):** Vendor Analysis results storage (findings, scores, summaries)
2. **New Process Model:** "Analyse Vendor Proposals" — triggered when evaluation starts, runs per vendor-factor combination
3. **New Interface:** "Vendor Analysis Tab" — added to `AS_GSS_FM_evaluationSummary` as a new tab
4. **New Interface:** "Analysis Pane for Evaluator" — added to `AS_GSS_FM_completeEvaluation` right pane
5. **Rich Text upgrade:** `AS_GSS_CPS_evaluationFactorFields` description and instructions fields → rich text, expanded to 4000 chars

### 13.2 What Needs to Be Modified (Existing)

| Object | Change |
|---|---|
| `AS_GSS_FM_evaluationSummary` | Add "Vendor Analysis" tab alongside existing summary content |
| `AS_GSS_FM_completeEvaluation` | Add analysis pane in right panel (alongside documents) |
| `AS_GSS_CPS_evaluationFactorFields` | Upgrade description/instructions to rich text, expand char limit |
| `AS_GSS_ENUM_TEXT_LIMIT_FOR_EVALUATION_DESCRIPTION` | Update from 255 to 4000 |
| `AS GSS Start Evaluation` (or related) | Trigger analysis kickoff when evaluation starts |
| AI infrastructure (`AS GSS Summarize With AI` pattern) | Reuse/extend for proposal analysis AI calls |

### 13.3 Data Flow for Analysis

```
Evaluation Started
  → For each Vendor:
      → For each Factor/Subfactor:
          → Gather: factor description, instructions, vendor documents (via FactorDocumentMapping)
          → Call AI Analysis (extend existing AI infrastructure)
          → Store: findings[], factorSummary, factorScore, factorConfidence
      → Aggregate: overallSummary, fitment (Strong/Moderate/Weak), overallScore, confidenceScore
  → Store results in new VendorAnalysis record type
  → Display in Vendor Analysis Tab (CO/Factor Chair view)
  → Display condensed version in Evaluation Task form (Evaluator view)
```

---

## 14. Naming Convention Reference

The application follows this naming pattern:
- **Record Types:** `AS_GSS_{Name}_SYNCEDRECORD` or `AS_GSS_{Name}_RECORD`
- **Interfaces:** `AS_GSS_{Prefix}_{name}` where prefix is:
  - `FM_` = Form
  - `CPS_` = Composite/Section
  - `GRD_` = Grid
  - `CRD_` = Card
  - `SCT_` = Section
  - `BOX_` = Box layout
  - `BTN_` = Button
  - `DSP_` = Display
  - `INP_` = Input
  - `SEC_` = Section (read-only)
  - `HCL_` = Header Content Layout
  - `CP_` = Component
- **Expression Rules:** `AS_GSS_{Prefix}_{name}` where prefix is:
  - `UT_` = Utility
  - `BL_` = Business Logic
  - `QR_` = Query Record
  - `QE_` = Query Entity
  - `VD_` = Validation
  - `ADT_` = Audit
  - `RA_` = Record Action
- **Process Models:** `AS GSS {Name}` (spaces, not underscores)
- **Constants:** `AS_GSS_{PREFIX}_{NAME}` (all caps for enums/refs)
