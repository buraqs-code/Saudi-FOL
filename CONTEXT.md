# FOL Automation

**What this does:** fetches data from the Saudi Exchange website (saudiexchange.sa), writes it into an Excel workbook over the internet through Microsoft's Graph API, produces a report from that data using formulas, and emails the report out.

**How it runs:** unattended, on a schedule, every day (Friday and Saturday included), on Anthropic's servers — not on anyone's desktop. Every read and write goes through a live Graph API session against the workbook wherever it lives.

**How it's written:** as instructions rather than code. Each stage is a folder with a `CONTEXT.md` file describing what to do, and the agent reads those instructions and does the work directly. Small throwaway scripts are fine where a step is really just parsing or comparing data. There's no application to maintain.

**Read as you go, not all at once.** Read this file and `GLOSSARY.md` once, at the start. After that, read one stage's `CONTEXT.md` at a time — only the one you're currently on.

**Status:** all eight stages below are settled. Follow each stage's `CONTEXT.md` as written.

## Settings — `config.yaml`

Three things live in `config.yaml` at the top level: where the workbook is (whose OneDrive, which folder, how its name is recognised), the web addresses the data is fetched from, and who the report goes to. If one of those needs changing, change it there rather than in a stage's instructions.

Everything else stays in the stage that uses it — sheet names, how the workbook is laid out, how the site is fetched from, sign-in details. No secret is in `config.yaml`, or anywhere else in this project.
## Stages

| Stage | What it does |
|---|---|
| `stages/01_authentication/` | Get permission to open the workbook and find out exactly which file it is |
| `stages/02_update_mkt_cap/` | Fetch the day's market cap data from saudiexchange.sa and write it into the workbook — split into its own steps, see its own `CONTEXT.md` |
| `stages/03_update_FO/` | Fetch the day's foreign ownership data from saudiexchange.sa and write it into `FO_Actual` sheet — split into its own steps, see its own `CONTEXT.md` |
| `stages/04_calculate_FOxMktCap/` | Add today's row to `FOxMktCap`, where the workbook works out how much of each company is foreign-owned in money terms. Only the latest 4 rows keep formulas |
| `stages/05_calculate_Diff_FOxMktCap/` | Add today's row to `Diff_FOxMktCap`, where the workbook works out the riyal value of the day's change in foreign ownership. Only the latest 4 rows keep formulas |
| `stages/06_build_report/` | Set the report date in `EMAIL.1` to the last trading day (the date on the FO page). On Friday, Saturday, or when there is no new trading day, set no date and send no report |
| `stages/07_export_pdf/` | Make two PDFs: `EMAIL.1` with `EMAIL.CHARTS` (the report), and `Email.0` with `EMAIL.2` for the email body, each from its own throwaway copy of the workbook |
| `stages/08_send-report/` | Email the report, then a "Run Succeeded" or "Run Stopped" email. On a no-report run, only a "No report" email. Every run ends here, even one that stopped early |

## The `run/` folder — files shared between stages, for one run

Some files aren't owned by any single stage: one stage writes them and a later stage reads them, within the same run. They live in `run/` at the top level, not inside a stage's own folder. Each file is named after the step that wrote it, e.g. `2a_mkt_cap.json`.

- `run/connection.md` — which workbook this run is writing into, and the codes needed to open it. Written by `stages/01_authentication/`, read by every stage that touches the workbook.
- `run/2a_mkt_cap.json`, `run/2b_mkt_cap_millions.json` — see `stages/02_update_mkt_cap/2a_fetch-data/CONTEXT.md` and `2b_write-to-sheet/CONTEXT.md` for what's in each and who reads them.
- `run/3a_fo.json` — see `stages/03_update_FO/3a_fetch-data/CONTEXT.md` for what's in it and who reads it.
- `run/7_report.pdf`, `run/7_body.pdf` — the report (`EMAIL.1` and `EMAIL.CHARTS`) as a PDF, and `Email.0` with `EMAIL.2` for the pictures in the email body. Written by `stages/07_export_pdf/`, read by `stages/08_send-report/`.
- **`run/issues.md`** — a running list of anything a person needs to see after the run: failures, ambiguous cases, anything worth attention. Any stage adds to it. A stage that stops the run ends its line with "the run stopped". `stages/08_send-report/` reads it to decide which emails to send. A line that starts with "No report:" tells stages 07 and 08 that no report goes out.

`run/` doesn't exist in a fresh copy of the project — whichever stage writes to it first creates it.
