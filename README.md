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

## Known Platform Limitations

- **Write-back / "Review & Submit" button:** This does not complete successfully in public embed contexts or on free-trial accounts (Snowflake and Sigma Computing), due to the lack of the grants, privileges, and user roles required to complete the write path.
  - That said, the button **is** functionally wired up correctly: it writes back to the original database, reflects the update in Sigma, and appends the new entry to Sigma's write-back table on the public/editable input data. The failure mode here is one of environment permissions, not implementation.

---

## Notes for Reviewers

- These are demo/sandbox artifacts intended purely for assessment purposes — not a production deployment.
- If either link stops working, it's most likely due to the limitations described above (JWT expiry on the sandbox link, or platform restrictions on public embeds) rather than an error in the workbook itself.
