Source Selection: Proposal Analyser

PO: Subash Narayana B | UX: Vedant Khaire		 Team: GAM Source Selection
Targeted Release: Source Selection 2.7

Summary:
We want to provide an AI based tool to help Evaluators complete their Evaluation tasks accurately and on time. Evaluators will be able to see an analysis of the vendor’s submitted documents related to the factor when filling up their Evaluation form. Additionally, COs and Factor chairs of the Evaluation will be able to view the analysis across factors and vendors.
Guiding Use Cases
Contracting Officers and Factor Chairs with access to the Evaluation will be able to see Vendor wise response analysis that shows a vendor wise findings organised by factors.
Evaluators are able to see a snippet of the analysis for the vendor/factor when filling up an Evaluation form
Use Cases Not Supported
Use AI to generate the Evaluation form elements - Strengths, Weakness section
Identify context outside of the solicitation
Why do we need this?
Proposal Analyser will significantly benefit Evaluators by:
Saving Time: Reducing the need to meticulously review all documents.
Ensuring Thoroughness: Helping to prevent the oversight of critical information.
This feature will be a significant market differentiator, giving us a competitive edge. By intelligently applying AI, this feature addresses major end-user challenges and significantly enhances the value of our platform.

The feature will also enhance sales demonstrations. It provides a tangible, practical showcase of our technology that improves customer outcomes and efficiency.
What is it?
Vendor Response Analysis - Backend
We want to kick off vendor response analysis based on the data that we have on each factor/subfactor including the description and instruction field and the Solicitation document (optional). The analysis will be kicked off for each vendor-factor combination. 

We will analyse each vendor document associated with that factor and get the following information:

Summary of the findings for that vendor - factor
Collection of findings and for each finding:
Define what is it (High level category of the finding)
Finding Text (AI generates a factual one line for the finding)
Actual Text
Source mentioning the document, section and page
Finding Type that is one of:
Highlights
Risks
Observations
AI Confidence that is one of:
High: Clear, explicit statements in source docs
Medium: Implied or partially supported
Low: Inferred with limited supporting evidence

For a vendor, using this factor wise observation and findings, we will generate one overall summary and the fitment of each vendor (Strong, Moderate, Weak).

For each factor, look at the vendor's findings and get a score based on the following conditions:



Conditions
Score
Strong
>=2 highlights AND
Highlights outnumber risks by at least 2:1 AND
No more than 1 risk
1
Moderate
Only 1 Highlights, OR
Highlights roughly equal to risks OR
Doesn't meet the bar for strong, but isn't clearly weak
0.6
Weak
Risks outnumber differentiators, OR
2+ high-confidence risks (automatic weak regardless of positives), OR
No response/findings at all
0.25



Calculate overall score for the whole vendor by doing a weighted sum of individual factors’ scores and if overall score:
>0.75 - Strong
0.45 to 0.74 - Moderate
<0.25 Weak

With respect to AI Confidence, we will do an average per factor and roll it up based on the factor weights to get a weighted confidence score per vendor. 
Description and Instructions field update in Evaluations
We will update both of these fields to be rich text and expand the length of these fields to (4000 characters?)

Our goal is to ensure the COs are able to add the necessary context for the GSS users and AI to do an extensive analysis of Vendor proposals. Hence we will update the fields so that COs can add formatted text that is readable by Evaluators and AI.

Descriptions (Solicitation Requirements) should include:
Specific minimum requirements (e.g., "5,000 concurrent users", "FedRAMP Moderate minimum")
Concrete benchmarks (e.g., "IGCE is $4.5M", "$20M over 5 years")
Required certifications with specifics
What vendors must demonstrate

Instructions (Internal Evaluation Guidance) should include:
Explicit Strengths criteria (what exceeds requirements)
Explicit Weakness/Deficiency indicators (what falls short or raises concerns)
Weighting guidance (e.g., "DoD experience weighted higher than civilian")
Threshold values (e.g., "CPARS ≥4.5/5.0", "pricing >10% above IGCE")

This gives AI the context to:
Compare vendor responses against stated requirements
Identify when responses exceed thresholds → tag as Differentiator
Identify when responses fall short or have gaps → tag as Risk
Generate justified reasoning based on the criteria


Contract Writing: Additional Data Points
We will integrate specific data points from the related Solicitation within Contract Writing. The following fields, located under the Contract Text section, are required:
Description/specifications/statement of work: All text fields populated in the Solicitation.
Instructions, conditions, and notices to offerors or respondents: All text fields populated in the Solicitation.
Evaluation factors for award: All text fields populated in the Solicitation.


New tab in Evaluation Summary with Vendor wise Analysis
Once an Evaluation is started, the analysis will be kicked off. Once that is complete we will show the analysis in a separate tab named “Vendor Analysis” in Evaluation Summary for COs and Factor Chairs. We will show all vendors with their fit (Strong, Moderate or Weak) and clicking on a vendor we will show the detailed analysis for that vendor.

In the analysis, we will show an Overall Summary for that Vendor followed by a Factor wise list of findings. For each finding we will show:
What
Summary of the finding
Source (clickable link)
Clicking on the source users are taken to the document and section/page where that finding was inferred from in a popup.
Evaluation form with analysis for Vendor-Factor
In the Evaluation form, Evaluators are able to see the analysis in the right pane (along with documents). In this tab we will show the findings for that vendor/factor. Clicking on the source, Evaluators are taken to the document and section/page in a popup.
