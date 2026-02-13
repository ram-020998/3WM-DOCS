# Proposal Analyser — Feature Summary

**Release:** Source Selection 2.7
**Team:** GAM Source Selection

---

## Purpose

AI-powered tool that analyses vendor proposal documents against solicitation factors/subfactors, producing structured findings to help Evaluators complete evaluations faster and more thoroughly. COs and Factor Chairs get a full dashboard view; Evaluators get a contextual snippet within their evaluation form.

---

## Core Concepts

### Analysis Unit
- Analysis runs per **vendor–factor** combination.
- Input: factor/subfactor description, instruction fields, vendor-uploaded documents, and (optionally) the solicitation document.

### Finding Object
Each finding produced by the AI contains:

| Field | Description |
|---|---|
| What | High-level category of the finding |
| Finding Text | AI-generated one-line factual summary |
| Actual Text | Verbatim excerpt from the source document |
| Source | Document name, section, and page number (clickable link) |
| Finding Type | One of: **Highlight**, **Risk**, **Observation** |
| AI Confidence | **High** (explicit in source), **Medium** (implied/partial), **Low** (inferred, limited evidence) |

### Vendor Fitment Scoring

**Per-factor score** based on findings:

| Fitment | Conditions | Score |
|---|---|---|
| Strong | ≥2 highlights AND highlights outnumber risks ≥2:1 AND ≤1 risk | 1 |
| Moderate | 1 highlight, OR highlights ≈ risks, OR doesn't qualify as strong/weak | 0.6 |
| Weak | Risks > highlights, OR 2+ high-confidence risks, OR no findings | 0.25 |

**Overall vendor score** = weighted sum of per-factor scores (using factor weights).
- \>0.75 → Strong
- 0.45–0.74 → Moderate
- <0.45 → Weak

**AI Confidence rollup** = weighted average of per-factor confidence scores (same factor weights).

### Per-Vendor Output
- Overall summary (AI-generated narrative)
- Fitment label (Strong / Moderate / Weak)
- Factor-wise list of findings with sources

---

## UI Components

### 1. Vendor Analysis Tab (CO / Factor Chair view)

Location: New tab in **Evaluation Summary** called "Vendor Analysis".

**Layout (from mockup):**
- **Left sidebar:** List of all vendors, each showing vendor name, fitment badge/score. First vendor selected by default. No empty state when vendors exist — empty state only shown while analysis is in progress.
- **Right pane (vendor detail):**
  - Vendor name, document count, confidence indicator, last generated date
  - Overall fitment score badge (e.g., "11 Moderate")
  - AI-generated overall summary (expandable "show more")
  - **Factor Analysis** section with filter chips (Highlights / Risks / Observations)
  - Expandable accordion cards per factor (e.g., "4060.1 Technical Feasibility")
    - Factor-level summary paragraph
    - Grouped findings by subfactor/category (e.g., Deployment, Scalability, Architecture)
    - Each finding shows: finding text, source link (e.g., "Volume 2, Section 3, Page 12–14")
    - First factor card expanded by default; rest collapsed

**Interaction:** Clicking a source link opens the referenced document at the specific section/page in a popup.

### 2. Evaluation Form Analysis Pane (Evaluator view)

Location: Right pane of the **Evaluation Form** (alongside the documents tab).

- Shows findings for the specific **vendor–factor** being evaluated.
- Smaller/condensed version of the factor findings from the main dashboard.
- Source links open document at section/page in a popup.

### 3. Description & Instructions Fields (Evaluation Setup)

- Both fields upgraded to **rich text** editors.
- Max length expanded (target: 4000 characters).
- These fields feed the AI analysis context, so COs should populate them with:
  - **Description:** Minimum requirements, benchmarks, certifications, what vendors must demonstrate.
  - **Instructions:** Strength/weakness criteria, weighting guidance, threshold values.

### 4. Contract Writing Integration

New data points pulled from the related Solicitation into Contract Writing under "Contract Text":
- Description/specifications/statement of work
- Instructions, conditions, and notices to offerors
- Evaluation factors for award

---

## Key Data Relationships (for building components)

```
Evaluation
 └── Vendors[]
      └── VendorAnalysis
           ├── overallSummary: string
           ├── fitment: Strong | Moderate | Weak
           ├── overallScore: number
           ├── confidenceScore: number
           └── FactorAnalysis[]
                ├── factor: Factor reference
                ├── factorSummary: string
                ├── factorScore: number
                ├── factorConfidence: number
                └── Findings[]
                     ├── what: string
                     ├── findingText: string
                     ├── actualText: string
                     ├── source: { document, section, page }
                     ├── findingType: Highlight | Risk | Observation
                     └── aiConfidence: High | Medium | Low
```

---

## Scope Boundaries

**In scope:**
- AI analysis of vendor documents against factor criteria
- Structured findings with source traceability
- Vendor fitment scoring and rollup
- Dashboard for COs/Factor Chairs, contextual pane for Evaluators
- Rich text for description/instructions fields
- Contract Writing data integration

**Out of scope:**
- AI-generated evaluation form content (strengths/weaknesses sections)
- Identifying context outside the solicitation
