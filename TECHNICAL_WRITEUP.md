# Technical Write-Up

This document walks through the design decisions behind each part of the assessment, referencing the actual files in this repo (`Dummy Datasets/`, `Sigma Dashboard/`, `Snowflake/Code/`, `Snowflake/ScreenShots/`).

---

## Architecture Overview

- **Warehouse:** Snowflake, account `bc54621` (Central India, Azure) — database `PC_SIGMA_DB`, schema `SILVER`, structured as a star schema.
- **BI / Embedding layer:** Sigma Computing workbook, connected to Snowflake via **OAuth** (not basic username/password auth) — see `Snowflake/Code/Sigma + Snowflake Integration.txt`. OAuth was used specifically because it's required for Sigma's write-back path to operate under a role-scoped, revocable credential rather than a shared static password.
- **Write-back layer:** A dedicated database, `SIGMA_WRITEBACK.PUBLIC`, separate from the read-only analytical schema (`PC_SIGMA_DB.SILVER`). This separation matters: the role used by Sigma (`PC_SIGMA_ROLE`) is granted `CREATE TABLE`, `CREATE STAGE`, and full `SELECT/INSERT/UPDATE/DELETE/TRUNCATE` only inside `SIGMA_WRITEBACK.PUBLIC` — the analytical schema stays untouched by the write-back mechanism, so a bad write can't corrupt the source-of-truth fact/dimension tables directly.

---

## Part 1 — Data Modeling

**Grain:** `FACT_TRANSACTIONS` is one row per transaction (`TRANSACTION_ID`), ~20,000 rows, with `ACCOUNT_ID`, `CUSTOMER_ID`, `BRANCH_ID`, `PRODUCT_ID`, and `DATE_ID` as foreign keys, plus `TRANSACTION_AMOUNT`, `RUNNING_BALANCE`, `TRANSACTION_TYPE`, `TRANSACTION_CHANNEL`, and `IS_FLAGGED_FRAUD`.

**Dimensions** (see `Dummy Datasets/`):
- `DIM_ACCOUNT` (1,500 rows) — keyed on `ACCOUNT_ID`, with `ACCOUNT_NUMBER` as the human-readable natural key used throughout the UI/filters.
- `DIM_CUSTOMERS` (1,000 rows), `DIM_BRANCH` (25 rows), `DIM_PRODUCT` (10 rows), `DIM_DATE` (~1,460 rows, roughly 4 years).

**Join / fan-out avoidance:** Each dimension is deduplicated at its own natural grain (one row per account, one per customer, etc.) before being related to the fact table, so joining any single dimension to `FACT_TRANSACTIONS` doesn't multiply transaction rows. Relationships were built using **Sigma's native relationship/join tool** rather than hand-written SQL joins in a custom query — this keeps the model declarative, so any new page or report added later can reuse the same relationships without redefining join SQL each time (see `Sigma Dashboard/Data Model - Join logic.png` and `Workbook Lineage.png` for the resulting relationship graph).

---

## Part 2 — Reporting Layer

The homepage (`Sigma Dashboard/KPI Overview - Homepage.png`) includes:
- **KPIs:** total transaction volume, total balance, active account count, average transaction size.
- **Visuals:** trend-over-time chart plus breakdowns by branch, product, and customer segment.
- A secondary drill-down page, **Customer Detailed Overview** (`Sigma Dashboard/Customer Detailed Overview - Page 2.png`), for account-level detail once a user narrows in from the homepage.

**Filter scoping:** Account Number, Date Range, Branch, and Product Type filters share a single control ID set across the page, so one filter interaction updates both the KPI values and every chart/table simultaneously — rather than each element carrying its own independent filter state.

---

## Part 3 — Embedded Sigma AI Agent

The chatbot element on the homepage (`Sigma Dashboard/Agent AI in Modal or PopUp.png`) is scoped directly to the workbook's data model — it answers only from `FACT_TRANSACTIONS` and the linked dimensions, not from open-ended knowledge, which is what keeps answers grounded rather than hallucinated.

Domain-specific instructions were configured so the agent maps banking terminology correctly (e.g., "balance" → `RUNNING_BALANCE`, "account" → `ACCOUNT_NUMBER` rather than the internal `ACCOUNT_ID` surrogate key).

**Agent-driven Input Table population:** the agent resolves a natural-language account reference (e.g., "pull up account X for review") to an Account Number filter value and triggers the **same modal** a user reaches manually via the trigger button (`Sigma Dashboard/filter pop up - automatic.png`). Because both paths converge on the same modal/filter mechanism, the resulting Input Table state is identical regardless of whether the account was selected via chat or by hand.

---

## Part 4 — Interactive Input Table Workflow (Action Sequence)

This is implemented as a chain across Sigma actions, a Snowflake view, and a Snowflake stored procedure:

1. **Trigger:** button opens a modal containing filter controls (Account Number + others), reachable manually or via the agent (Part 3).
2. **Populate Input Table:** the table is populated from `SIGMA_WRITEBACK.PUBLIC.EDITABLE_INPUT_TABLE` — a Sigma-generated warehouse view (`Snowflake/Code/write_back.Editable_input - warehouse view`) that joins `FACT_TRANSACTIONS → DIM_ACCOUNT → DIM_CUSTOMERS` to surface each account's current values (name, account status, email, phone, KYC flag) alongside blank `UPDATED_*` columns for editing, plus `LAST_UPDATED_AT`/`LAST_UPDATED_BY` audit columns.
3. **Edit & Submit:** the user edits only the `UPDATED_*` columns for the account in question; untouched fields are left `NULL`.
4. **Change Review screen:** on submit, a summary modal (`Sigma Dashboard/Review Changes Summary.png`) shows before/after values, sourced directly from the view's paired original vs. `UPDATED_*` columns (e.g. `ACCOUNT_STATUS` vs `UPDATED_ACCOUNT_STATUS`).
5. **Approve → write back:** approval calls the stored procedure `SIGMA_WRITEBACK.PROCEDURES.WRITE_BACK_FROM_INPUT_TABLE(SOURCE_TABLE)` (`Snowflake/Code/Call Store Proc.txt`), which:
   - `MERGE`s `UPDATED_ACCOUNT_STATUS` into `DIM_ACCOUNT`, filtered to only rows where that field is non-null.
   - `MERGE`s `UPDATED_EMAIL` / `UPDATED_PHONE_NUMBER` / `UPDATED_KYC_VERIFIED` into `DIM_CUSTOMERS` (joined back from account → customer), using `COALESCE` so any field the user *didn't* edit is preserved rather than overwritten with null.
   - Returns a row-count summary string (e.g. "DIM_ACCOUNT: N rows updated. DIM_CUSTOMERS: N rows updated."), giving an audit trail of exactly what the batch touched.
6. **Reset:** after a successful write, the Input Table is cleared via the view's row-versioning/tombstone mechanism (`ROW_VERSION` / `TOMBSTONE_VERSION` in the underlying Sigma dataset), so it's empty and ready for the next batch.

**Edge case handling:** because unedited columns stay `NULL` and the merge logic explicitly filters/`COALESCE`s on non-null values, a submission with no changes or only partial edits safely no-ops on the untouched fields instead of overwriting them with blanks — this was a deliberate design choice to make the write-back idempotent-safe against partial edits.

---

## Known Limitations

- **Public embed AI Assistant:** does not function inside Sigma's public embed mode — a platform constraint of public embedding, not a configuration gap in the agent (see sandbox JWT link in the main README to evaluate the agentic flow directly).
- **Write-back in public embed / free trial:** the `Review → Approve` write path does not complete successfully under the public embed or free-trial permission set, since the required Snowflake grants/roles for external write access aren't available in that context. The procedure and MERGE logic themselves are complete and correct — the limitation is environmental (missing grants/privileges), not a gap in the action sequence.

---

## Repo Map

| Path | Contents |
|---|---|
| `Dummy Datasets/` | CSV source data for the fact + 5 dimension tables |
| `Sigma Dashboard/` | Screenshots of the homepage, drill-down page, filter modal, agent modal, review-changes modal, data model/lineage |
| `Snowflake/Code/` | DDL for all 6 tables, the OAuth integration setup, the write-back view SQL, and the write-back stored procedure |
| `Snowflake/ScreenShots/` | Snowflake-side views of the database/schema/procedure/table objects |
