# 3b_write-to-sheet: write foreign ownership data into the workbook

The Task: write today's data into `FO_Actual`, using the figures `3a_fetch-data` already fetched and converted. 

## Inputs
- `../../../run/3a_fo.json`
- `../../../run/connection.md`
- `../../../run/2a_mkt_cap.json` — used to tell a real company apart from an ETF or fund (step 6)
- `../../../run/issues.md` — to see if stage 2b found the market closed

## Process

1. **Open the sheet** in the workbook.

2. **Read the header rows.** Same layout as `Mkt_Cap` — **NOMU code** (row 1), **TASI code** (row 2), **Bloomberg code** (row 3), **company name** (row 4), one company per column from column B onward. Read fresh each run. Not every filled cell in the code rows is a real code (there's a `TASI END` divider) — treat anything that isn't a valid number as "no code," not an error.

3. **Find the latest date already recorded:** Read down column A to the last row with a date in it.

4. **Pick the row.** The row date is the same as in stage 2b: yesterday, in Pakistan time. Compare it with the latest date in this sheet:
   - **Latest date is older** — use a new row, directly below it.
   - **Same date** — an earlier run already added this row. Use it.
   - **Latest date is newer** — something is wrong. Write nothing, add a line to `../../../run/issues.md` saying what each date was, and skip the rest of this stage.

5. **Decide what goes in the row:**
   - **Market closed** (`issues.md` has a line starting "Market closed:") — copy the row above. Stage 2b already logged it.
   - **Nothing fetched** (`run/3a_fo.json` has `"as_of_date": null` or an empty `companies` list) — stop. Write nothing to this sheet.
     - Stage 2b has already added the row date to `Mkt_Cap`. Remove that row, so the two sheets stay on the same day: if `Mkt_Cap`'s last date is the row date and `FO_Actual` doesn't have that date, clear that whole row in `Mkt_Cap` (date and figures). See "Technical details".
     - Add a line to `../../../run/issues.md` saying the FO page failed and why (e.g. "blocked: 403 Access Denied"). End it with "the run stopped", and skip straight to `stages/08_send-report/`.
   - **`as_of_date` is the same as the row date** — write the fetched figures.
   - **`as_of_date` is older** — the FO page is behind. It sometimes updates later than the market cap pages. Copy the row above. Add a line to `../../../run/issues.md` starting **"FO page behind:"**, giving the date the FO page showed and the row date.
   - **`as_of_date` is newer** — this shouldn't happen. Write nothing, not even the date. Log both dates in `../../../run/issues.md` and skip the rest of this stage.

6. **Write the row.** For a new row, write the row date in column A. Only fill blank cells. Never overwrite a value that's already there.

   - **Copying the row above:** copy each company's value unchanged. If a company's cell above is blank, leave it blank here too. Don't write an empty value for it.

   - **Writing the fetched figures:** for each company in `run/3a_fo.json`, find its column:
     - First try to match its `symbol` against the sheet's **TASI code** row.
     - If not found there, try the **NOMU code** row.
     - If it matches neither, look that symbol up in `run/2a_mkt_cap.json` to work out whether it's something that should have had a column:
       - **Not there either** — it's an ETF or a fund, not a company, so it was never expected to have a column here. Skip it and don't log it. This is normal and happens every run.
       - **Found there** — it's a real company with no column in this sheet yet, e.g. a new listing. Add a line to `../../../run/issues.md` naming the company and its symbol, then skip just that company.
       - **If `run/2a_mkt_cap.json` is missing or has no companies in it**, this check can't be made at all. Say so once in `../../../run/issues.md` and skip the unmatched symbols, rather than silently treating every one of them as a fund.
     - Once matched, write `fo_actual_decimal` into that column.
   - **When writing into an existing same-date row**, note in `../../../run/issues.md` how many companies actually got filled in versus how many were already there — including saying so if that number is zero ("today's data was already fully recorded, nothing new to add").

   **Before closing the session**, read back the date cell and at least one company cell just written, to confirm the value landed.

## Outputs
- `FO_Actual` updated in place
- `../../../run/issues.md` — added to wherever a step above says to

## Technical details

**Sheet layout** is identical to `Mkt_Cap`, same header rows, same company columns in the same order, same `TASI END` divider at the same column position. The matching logic from stage 2b applies here unchanged.

**Clearing the `Mkt_Cap` row** (step 5) — `POST .../worksheets('Mkt_Cap')/range(address='A3897:OH3897')/clear` with `{"applyTo": "Contents"}`. The row number and last column here are examples. Read back column A afterwards: that row should be empty, and the last date should be the one before.

**The row date** is worked out the same way as in stage 2b's "Technical details". `as_of_date` is written like `Sep 30, 2026`. Turn it into a real date before comparing.

**Dates work exactly like `Mkt_Cap`** — real Excel serial numbers, `numberFormat` `d-mmm-yy`, both set together when writing a new date (a brand-new row hasn't inherited the column's formatting).

**The percentage values need their number format set explicitly, every time a new cell is written** — Write the fraction (e.g. `0.49`) and set `numberFormat` to a percentage format (e.g. `0.0%`) on that cell in the same request, regardless of what format older rows in that column happen to use.

**Never write a cell as blank alongside real values** — Excel turns an empty value written next to real numbers into `0`. Leave unmatched or already-filled cells out of the write entirely.

**Session use**: one session per sheet (open, do that sheet's reads and write, verify, close) — same mechanics as stage 2b, including `Content-Length: 0` on `closeSession`.

**Reading efficiently**: same approach as stage 2b — get the shape first, then read only the header rows and a slice of the date column, not the whole used range's values. The used range can run far past the data. If the slice has no date, keep reading further up until one is found.
