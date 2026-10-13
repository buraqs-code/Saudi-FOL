# Glossary

Recurring terms in this project. Add to this as new terms come up — don't let a term get used in a stage's `CONTEXT.md` without a definition here first.

- **saudiexchange.sa** — the Saudi Exchange's website; where this automation's data comes from. It blocks plain requests (fingerprints the connection itself).
- **The workbook** — the Excel file this automation writes into. It sits in a OneDrive folder, and is found by part of its name, since it gets renamed. `config.yaml` says whose OneDrive, which folder, and which words to look for. Stage 01 finds the exact file each run.
- **The market flows workbook** — a second, separate Excel file, used only by stage 08. It sits in the same OneDrive folder as the workbook and is found the same way, by part of its name (`market_flows_workbook` in `config.yaml`). "The workbook" on its own always means the main one, never this.
- **Weekly report** — the market flows report on saudiexchange.sa. It's published once a week, not daily, so most runs find nothing new and keep last week's figures.
- **Graph API session** — a `workbook-session-id` used for every read/write against the workbook so writes and reads stay consistent.
- **Today's row** — the row each run adds. It's dated **yesterday**: the day before the run, in Pakistan time. A row is added every day, even when the market was closed.
- **Market closed day** — a day with no trading: Friday, Saturday or a holiday. The site still shows the last open day. The row is filled by copying the row above. Stage 06 decides if a report goes out.
- **Report date** — the date that all figures in the report use. It is the last trading day: the date on the foreign ownership page. It is not always the date of the last row. On Sunday, the last row is Saturday, but the report date is Thursday.
- **No-report run** — a run that updates the sheets but sends no report. This occurs on Friday, on Saturday, and when there is no new trading day (a holiday, or the FO page did not update). Only a short status email goes out.
