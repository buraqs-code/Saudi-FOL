# 2b_write-to-sheet: write market cap data into the workbook

The Task: convert the fetched market cap figures to millions, then write today's row into the `Mkt_Cap` sheet.

## Inputs
- `../../../run/2a_mkt_cap.json`
- `../../../run/connection.md`

## Process

1. **Convert to millions.** Read `run/2a_mkt_cap.json`. Build a new JSON, same shape, with each company's `market_cap_sar` replaced by `market_cap_millions` = `market_cap_sar ÷ 1,000,000`. Write the full value and let the sheet's existing number format decide how it's displayed. Save this as `../../../run/2b_mkt_cap_millions.json`.

   If `run/2a_mkt_cap.json` is missing, or its `companies` list is empty, there's nothing to convert. Don't skip this stage. Carry on.

2. **Work out the row date.** Every run adds one row, dated **yesterday**: the day before the run, in Pakistan time. This happens every day, even when the market was closed. Use the real calendar: e.g. after 30 Sep comes 1 Oct, after 31 Dec comes 1 Jan. See "Technical details".

3. Open the `Mkt_Cap` sheet.

4. **Read the header rows.** The top four rows are, in order: **NOMU code** (row 1), **TASI code** (row 2), **Bloomberg code** (row 3), **company name** (row 4) — one company per column, starting at column B. Read all four across every used column fresh each run; this is how companies get matched, and the set of columns can grow over time.

   Not every filled cell in the code rows is an actual code. See "Technical details".

5. **Find the latest date already recorded.** Read down column A (below the header rows) to the last row that has a date in it. This is the latest date in the sheet.

6. **Pick the row.** Compare the row date with the latest date in the sheet:
   - **Latest date is older** — use a new row, directly below it.
   - **Same date** — an earlier run already added this row. Use it.
   - **Latest date is newer** — something is wrong. Write nothing, add a line to `../../../run/issues.md` saying what each date was, and skip the rest of this stage.

7. **Was the market open?** First check the fetch worked:
   - **Not everything was fetched** (`run/2a_mkt_cap.json` missing or empty, or TASI or NOMU missing from `pages_fetched`):
     - **Friday or Saturday** — the market is closed anyway. Copy the row above and log it as "Market closed:" (see below).
     - **Any other day** — stop. Write nothing. Add a line to `../../../run/issues.md` saying which page failed and why (e.g. "blocked: 403 Access Denied"). End it with "the run stopped", and skip straight to `stages/08_send-report/`.

   Otherwise, compare `market_date` (from the JSON) with the row date:
   - **Same date** — the market was open. Write the fetched figures.
   - **`market_date` is older** — the market was closed (weekend or holiday). The site still shows the last open day. Copy the row above. Add a line to `../../../run/issues.md` starting **"Market closed:"**, giving the row date, the last open day, and whether the row date is a weekend (Friday or Saturday) or a weekday. This line is for information only. Stage 06 decides if the report goes out.
   - **`market_date` is newer** — this shouldn't happen. Write nothing, not even the date. Log both dates in `../../../run/issues.md` and skip the rest of this stage.

   **The last open day** is the day the copied figures really come from. The row above may itself be a copy, so don't just use its date. Use the date the site shows. If nothing was fetched, go up from the row above, past rows with exactly the same figures. The earliest of those is the last open day.

8. **Write the row.** For a new row, write the row date in column A. Only fill blank cells. Never overwrite a value that's already there.

   - **Copying the row above:** copy each company's value unchanged. If a company's cell above is blank, leave it blank here too. Don't write an empty value for it.

   - **Writing the fetched figures:** for each company in `run/2b_mkt_cap_millions.json`, find its column:
     - First try to match its `symbol` against the sheet's **TASI code** row.
     - If not found there, try the **NOMU code** row.
     - If it matches neither, this is a company with no column yet (e.g. a new listing). Don't create one. Add a line to `../../../run/issues.md` naming the company and its symbol, and skip just that company.
     - Once matched, write `market_cap_millions` into that column. Only send cells that actually have a value to write — see "Technical details" for why a blank cell needs different handling than a real one.

   - **When writing into an existing same-date row** (step 6's "same date" case), note in `../../../run/issues.md` how many companies actually got filled in versus how many were already there — including saying so if that number is zero ("today's data was already fully recorded, nothing new to add")

   **Before closing the session**, read back the date cell and at least one company cell just written, in that same session. Confirm the value actually landed before moving on.

## Outputs
- `../../../run/2b_mkt_cap_millions.json` — the converted figures, same shape as `2a_mkt_cap.json` but in millions
- The `Mkt_Cap` sheet, updated in place
- `../../../run/issues.md` — added to wherever a step above says to

## Technical details

**Yesterday in Pakistan time** — take the current time in UTC and add 5 hours. That gives today's date in Pakistan (UTC+5 all year, no daylight saving). The row date is the day before. Don't rely on the server's time zone.

`market_date` is written as the page shows it, e.g. `2026/09/30`. Turn it into a real date before comparing.

**Sheet layout:**
- Column A holds row labels (`NOMU CODE`, `TASI CODE`, `Bloomberg_Code`, `NAME`) and, from row 5 down, the date of each row. Companies start at column B and run to whatever the sheet's used range currently covers. Read the shape fresh each run with rather than assuming a fixed last column.
- Rows 1–4 are the header rows described in step 4. A transferred company (NOMU → TASI) has both row 1 and row 2 filled for its column; a handful of these already exist in the sheet.
- TASI/NOMU code cells are stored as numbers (e.g. `2030`), not text — convert types consistently when matching against the JSON's `symbol` (a string).
- **All code rows aren't purely codes.** Partway through the TASI code row there's a text cell reading `TASI END` which is a section-divider and not a company. Treat any code-row cell that isn't a valid number as "no code" for that column (skip it, don't error on it, don't treat it as a symbol) rather than assuming every filled cell is numeric.

**Dates are real Excel date serial numbers, not text** — e.g. `16-Sep-2026` is stored as `46281`, with `numberFormat` `d-mmm-yy` on that cell. To write a new date:
- Convert the row date to the serial number Excel uses (days since 1899-12-30).
- Write the serial number **and** set `numberFormat` to `d-mmm-yy` on that cell in the same request. A brand-new row will not have inherited the column's formatting, so skipping this would a raw number instead of a date which is not what we want.

**Company cells use plain `General` formatting** — no special number format is needed when writing `market_cap_millions` values.

**Never send an empty value in the same write as real numbers** — Excel turns it into `0` instead of leaving it blank. Leave unmatched companies and already-filled cells out of the write entirely, rather than writing them as empty.

**Session use:** open one workbook session (`createSession`, `persistChanges: true`), do all of this stage's reads and the write inside it, then close it. Without a session, a read straight after a write can still hand back the old value.
- Verify the write before closing the session (see step 8).
- `closeSession` sends no body, but some HTTP clients need `Content-Length: 0` set explicitly, or it fails with `411 Length Required` instead of `204`.

**Reading efficiently:** don't pull the whole used range's values. Get the shape first, then read only what's needed: `A1:{lastCol}4` for the header rows, and a small slice of column A (working backwards from the last used row) to find the latest date. The used range can run far past the data (e.g. to row 4075 when the last date is on row 3888). If the slice has no date, keep reading further up until one is found.
