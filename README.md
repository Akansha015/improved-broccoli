# Sigma Computing — Technical Assessment
### Banking / Financial Dashboard + Agentic AI

**Submitted by:** Akansha Pruthi

---

## Demo Links

### 1. Public Embed
[View public embed](https://app.sigmacomputing.com/embed/37IAZXPhVbqBPa2DKs8PAj)

> ⚠️ **Known limitation:** The AI Assistant does not function inside public embeds. It will appear broken/unresponsive in this view — this is a platform limitation of public embedding, not an issue with the underlying workbook or agent configuration.

### 2. Sandbox Embed (JWT) — for evaluating the Agentic AI flow
[Open sandbox link](https://app.sigmacomputing.com/bc546210/workbook/ll00oUDTOq4aLkrekzoWh?:jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6Ijg2M2FkODkwNmZmZjE2MDg1NGYzZDZhNDkwYTQwOWI4ZGNkNjhjZDBiYTIyYzgwOTY2MzAzNmJjOTk4MmIyODIifQ.eyJzdWIiOiJha2Fuc2hhLnBydXRoaTE1QGdtYWlsLmNvbSIsImF1ZCI6InNpZ21hY29tcHV0aW5nIiwidmVyIjoiMS4xIiwianRpIjoiNjk4OGRhMzgtMDBhNy00OTVhLTliYmYtZjhiM2QzZTMzMjAzIiwiaWF0IjoxNzg0NDk5MDY5LCJleHAiOjE3ODQ1MDI2NjksImlzcyI6Ijg2M2FkODkwNmZmZjE2MDg1NGYzZDZhNDkwYTQwOWI4ZGNkNjhjZDBiYTIyYzgwOTY2MzAzNmJjOTk4MmIyODIifQ.yHUheB5vHm9P63wU-XpHa0I7rs5DKYi6fEzEF1jJVtU&:embed=true&:menu_position=bottom)

> ⚠️ **This link is single-use.** It is signed with a short-lived JWT and will not work if you refresh the browser or reopen it a second time. Please use this link **only** to evaluate the agentic AI flow, in one continuous session.

---

## End-to-End Workflow Diagram
[View on Miro](https://miro.com/app/live-embed/uXjVH6XtfJw=/?embedMode=view_only_without_ui&moveToViewport=-5054%2C-5652%2C17727%2C9210&embedId=69090948995)

---

## Demo Recording
[Watch on YouTube](https://youtu.be/KS6FSx8AZDE)

---

## Tasks Completed

### Part 1 — Data Modeling
- Dummy banking/financial dataset sourced and loaded into the warehouse: one fact table (`FACT_TRANSACTIONS`) and supporting dimension tables (`DIM_ACCOUNT`, `DIM_CUSTOMER`, `DIM_BRANCH`, `DIM_PRODUCT`, `DIM_DATE`).
- Sigma data model built joining all dimension tables to the fact table on the correct keys, forming a clean star schema at the transaction grain.
- Relationships configured using Sigma's native relationship/join tools (rather than manual SQL joins) to keep the model reusable across downstream reports, with join logic verified to avoid fan-out/duplication.

### Part 2 — Reporting Layer
- Home page report built on top of the Part 1 data model.
- KPIs wired correctly: total transaction volume, total balance, number of active accounts, average transaction size.
- Visualizations included for trend analysis and breakdowns by product, branch, and customer segment.
- Filters implemented and scoped consistently across all KPIs and visuals, including:
  - Account Number
  - Date Range
  - Branch
  - Product Type
- Control IDs applied consistently so all filters correctly control both KPI values and visual/tabular elements on the page.

### Part 3 — Embedded Sigma AI Agent
- AI agent (chatbot element) added to the home page, scoped to the underlying data model so answers are grounded in the actual data (no hallucinated figures).
- Agent instructions/context configured for the banking domain to support natural-language questions (e.g., transaction volume by branch/quarter, top accounts by balance).
- Agent-driven Input Table population wired correctly: a natural-language account reference (e.g., "Pull up account 10293847 for review") is mapped to the equivalent filter logic and populates the Input Table automatically, without the user opening the filter popup manually.
- Manual filter-popup path and agent-driven path both land in the same Input Table state, feeding into the same submit → review → approve → reset flow from Part 4.
- Handling included for ambiguous/invalid input (e.g., account not found, vague queries).
- **Known limitation:** as noted below, the AI Assistant does not function inside the public embed view — this is a Sigma public-embedding platform constraint, not a configuration issue with the agent itself. Use the sandbox JWT link above to evaluate the agentic flow.

### Part 4 — Interactive Input Table Workflow (Action Sequence)
- Trigger button/element added to the report; opens a popup/modal with the same filter controls as the main page (Account Number + additional filters), also triggerable via the AI agent per Part 3.
- Input Table populates correctly based on filter selection (manual or agent-driven) with the corresponding account's editable records.
- Edit & Submit flow wired correctly: users can edit values (e.g., balances, statuses) directly in the Input Table and submit.
- Change Review screen wired correctly: on submit, a summary popup displays modified records/columns with before/after values.
- Approval action wired correctly: clicking Approve sends the finalized batch of changes back to the warehouse via Sigma's write-back/Input Table sync, then clears/resets the Input Table for the next batch.
- Full action sequence chained correctly end-to-end: open modal → populate Input Table → submit → show change summary → approve → write back → reset.
- Edge cases handled: no changes made, partial edits, and canceling mid-flow.
- **Known limitation:** as noted below, the write-back step does not complete successfully in the public embed / free-trial environment due to missing grants, privileges, and user roles — not a gap in the action sequence logic itself, which is correctly wired end-to-end.

---

## Known Platform Limitations

- **Write-back / "Review & Submit" button:** This does not complete successfully in public embed contexts or on free-trial accounts (Snowflake and Sigma Computing), due to the lack of the grants, privileges, and user roles required to complete the write path.
  - That said, the button **is** functionally wired up correctly: it writes back to the original database, reflects the update in Sigma, and appends the new entry to Sigma's write-back table on the public/editable input data. The failure mode here is one of environment permissions, not implementation.

---

## Notes for Reviewers

- These are demo/sandbox artifacts intended purely for assessment purposes — not a production deployment.
- If either link stops working, it's most likely due to the limitations described above (JWT expiry on the sandbox link, or platform restrictions on public embeds) rather than an error in the workbook itself.
