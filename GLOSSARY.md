# Glossary

Recurring terms in this project. Add to this as new terms come up — don't let a term get used in a stage's `CONTEXT.md` without a definition here first.

- **saudiexchange.sa** — the Saudi Exchange's website; where this automation's data comes from. It blocks plain requests (fingerprints the connection itself).
- **The workbook** — the Excel file this automation writes into. It sits in a OneDrive folder, and is found by part of its name, since it gets renamed. `config.yaml` says whose OneDrive, which folder, and which words to look for. Stage 01 finds the exact file each run.
- **Graph API session** — a `workbook-session-id` used for every read/write against the workbook so writes and reads stay consistent.
- **Today's row** — the row each run adds. It's dated **yesterday**: the day before the run, in Pakistan time. A row is added every day, even when the market was closed.
- **Market closed day** — a day with no trading: Friday, Saturday or a holiday. The site still shows the last open day. The row is filled by copying the row above, and no report goes out, only a status email.
