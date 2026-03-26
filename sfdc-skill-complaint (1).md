---
name: BuildOps Salesforce Semantic Layer
description: Use BuildOps' canonical Salesforce + Snowflake definitions for industry, ARR, pipeline, and sales performance questions.
---

## When to use this skill

Use this skill whenever:

- The user asks about **pipeline, ARR, revenue, bookings, win rate, forecast, coverage, or similar GTM metrics**.
- The user asks about **Industry / Sub-industry / Segment** for an account or opportunity.
- The user asks for **lists or rankings of opportunities/accounts** (e.g., “top open opps by ARR”, “pipeline by industry”).
- The question explicitly mentions **Salesforce** or clearly relies on CRM data, even if Salesforce is not named.

If the question is purely about unstructured content (e.g., “summarize recent emails with Acme”), you may not need this skill; fall back to normal Glean search behavior.

---

## Source-of-truth and data prioritization

Always follow this **order of precedence** when answering metric or industry questions:

1. **Snowflake semantic layer (preferred)**  
   - Use **semantic-layer models / views** that encode BuildOps’ business definitions.  
   - Examples (replace with real names):  
     - `MODEL: <REPLACE_WITH_PIPELINE_MODEL_NAME>` – canonical pipeline definition  
     - `MODEL: <REPLACE_WITH_ARR_MODEL_NAME>` – canonical ARR / revenue definition  

2. **Canonical Salesforce fields (fallback)**  
   Only if there is no appropriate Snowflake semantic-layer view for the question, or Snowflake is unavailable:

   - Industry:  
     - Canonical field: `<REPLACE_WITH_CANONICAL_INDUSTRY_API_NAME>`  
     - Use only this field for “industry” unless the user explicitly asks for a specific source (e.g., ZoomInfo vs Clearbit).
   - ARR / revenue:  
     - Canonical field: `<REPLACE_WITH_CANONICAL_ARR_API_NAME>`
   - Other key fields:  
     - Sub-industry: `<REPLACE_WITH_SUB_INDUSTRY_API_NAME>`  
     - Parent industry: `<REPLACE_WITH_PARENT_INDUSTRY_API_NAME>`  
     - Segment / tier: `<REPLACE_WITH_SEGMENT_API_NAME>`

3. **Other sources (context only, not authoritative)**  
   - Slack messages, emails, docs, ad-hoc spreadsheets, or legacy Salesforce fields **must not override** Snowflake semantic-layer definitions or canonical Salesforce fields.  
   - You may reference them for qualitative context (“sales notes indicate…”) but do not change metric values based on them.

If data from multiple sources conflict, **trust the Snowflake semantic-layer view first, then canonical Salesforce fields.** Call out conflicts explicitly rather than silently choosing a different value.

---

## Required preparation: consult the BuildOps data dictionary

Before answering any question about **pipeline, ARR, bookings, revenue, industry, or segments**:

1. **Locate the BuildOps Salesforce & Snowflake Data Dictionary**  
   - First, search for the internal data dictionary document.  
   - Use Glean search with queries like:  
     - `BuildOps Salesforce data dictionary`  
     - `BuildOps Snowflake semantic layer dictionary`  
   - Once found, treat definitions in that document as the **authoritative business meaning** of each term.

2. **Align question terms to dictionary definitions**  
   - Map ambiguous terms like **“pipeline”** to the exact dictionary definition (e.g., which stages, which opportunity types, which ARR field).  
   - If the user’s wording conflicts with the dictionary (for example, they say “pipeline” but seem to mean “created pipeline this quarter”), follow the dictionary, then either:  
     - Ask a concise clarifying question, **or**  
     - Answer using the dictionary definition and state what definition you used.

Never silently invent your own definition.

---

## How to choose between Salesforce SOQL, Glean Salesforce Search, and Snowflake SQL

When you need live structured data:

1. **Prefer Snowflake semantic-layer SQL when:**
   - The question is about **metrics over time** (e.g., pipeline by quarter, ARR by segment).
   - The data dictionary states the metric is defined in **Snowflake semantic-layer models or views**.

   **Steps:**
   1. Identify the relevant semantic-layer view(s) from the dictionary.  
   2. Use the **Snowflake SQL action** to query only those views and only the necessary fields.  
   3. Apply filters exactly as described in the dictionary (e.g., which stages count as “pipeline”).

2. **Use Salesforce SOQL when:**
   - The question is primarily about **current CRM state** (e.g., “top 20 open opps by ARR”, “accounts with active opportunities in manufacturing industry”).  
   - The canonical definition for the metric is a **Salesforce field**, not a Snowflake model.

   **Steps:**
   1. Identify the correct **object(s)** and canonical fields from the dictionary.  
      - For example, `Opportunity.<REPLACE_WITH_CANONICAL_ARR_API_NAME>` for ARR.  
   2. Write **permission-aware SOQL** that:  
      - Filters to the right **stages**, **record types**, and **time windows** per the dictionary.  
      - Avoids deprecated or non-canonical fields even if they exist.  
   3. Use the **Salesforce SOQL action** to run the query and base your answer only on the canonical fields.

3. **Use Glean Salesforce Search (app:salescloud) when:**
   - The question is about **finding or inspecting a small number of specific records** (e.g., “show me the latest notes for the Acme Corp opportunity”).  
   - You do not need aggregate metrics or complex filters.

In those cases, Glean search + reading the record is sufficient; you do not need SOQL or Snowflake.

If you are unsure which path is correct, default to **Snowflake semantic-layer SQL for aggregate metrics** and **SOQL for record-level CRM questions**, following the dictionary.

---

## Field and view mapping (to be customized)

Replace the placeholders below with BuildOps’ real API names and view names before using this skill.

### Canonical Salesforce fields

- Industry: `<REPLACE_WITH_CANONICAL_INDUSTRY_API_NAME>`  
- Sub-industry: `<REPLACE_WITH_SUB_INDUSTRY_API_NAME>`  
- Parent industry: `<REPLACE_WITH_PARENT_INDUSTRY_API_NAME>`  
- ARR: `<REPLACE_WITH_CANONICAL_ARR_API_NAME>`  
- Bookings / TCV: `<REPLACE_WITH_TCV_API_NAME>`  
- Segment / tier: `<REPLACE_WITH_SEGMENT_API_NAME>`  
- Any other metric fields that the data dictionary calls out as canonical.

### Snowflake semantic-layer models / views

- Pipeline view: `<REPLACE_WITH_PIPELINE_VIEW_NAME>`  
- ARR / revenue view: `<REPLACE_WITH_ARR_VIEW_NAME>`  
- Customer performance view(s): `<REPLACE_WITH_CUSTOMER_PERF_VIEW_NAME>`  
- Any departmental rollups (Sales, CS, PS) that should be preferred for those teams.

Document any **known exclusions** (e.g., “Exclude opportunities with stage = Closed Lost from pipeline; exclude certain record types”). Mirror exactly what the data dictionary specifies.

---

## Answer construction

When you’ve pulled the correct data:

1. **Explain the definition you used**  
   - Briefly restate the business definition (e.g., which stages, which ARR field, which time window).

2. **Summarize the result, then show key details**  
   - For exec questions, lead with a concise summary (e.g., total pipeline, top 3 drivers, key segments).  
   - For seller questions, focus on **prioritized lists or next-best-actions**, not only tables.

3. **Be transparent about sources and filters**  
   - Mention whether the answer is based on **Snowflake semantic-layer views** or **Salesforce fields**, and which ones.  
   - If relevant, call out major filters (date range, stages, segments) in plain language.

4. **Flag data limitations or conflicts**  
   - If Snowflake and Salesforce disagree, say so and indicate which you trusted and why.  
   - If key fields are missing or null for many records, mention the impact on the result.

---

## Safety and guardrails

- **Never**:
  - Infer financial metrics or ARR from unstructured text when a canonical field or view exists.  
  - Mix canonical and non-canonical fields in a way that changes metric definitions.  
  - Override the data dictionary definitions based on your own interpretation.

- **When in doubt**:
  - Prefer **not answering precisely** over answering with a made-up definition.  
  - Offer a safe fallback, for example:  
    > “I can’t confidently answer this without a clearer definition of \<TERM>. Here are the standard BuildOps definitions I see; which one should I use?”

---

## Handling ambiguous or risky questions

When terms are ambiguous (for example, “pipeline”, “bookings”, “open opportunities”, “active customers”):

1. **Check the dictionary** for the exact business definition.  
2. If the user’s phrasing does not clearly match a single definition:
   - Ask **one short clarifying question** (“When you say pipeline, do you mean all open opportunities, or only Stage X and above per the BuildOps definition?”), **or**  
   - If you must proceed, choose the primary dictionary definition and clearly state:  
     > “In this answer, I use BuildOps’ standard definition of pipeline: \<insert definition from dictionary>.”

Never silently invent your own definition.