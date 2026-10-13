# 04_calculate_FOxMktCap: add today's row to the FOxMktCap sheet

The Task: add one new row to the `FOxMktCap` sheet, so the sheet works out how much of each company is foreign-owned in money terms.

## How this sheet works

Two sheets already hold today's numbers, put there by stages 02 and 03:

- `Mkt_Cap` — what each company is worth, in millions of riyals
- `FO_Actual` — how much of it foreigners own, written as a decimal (`0.0296` means 2.96%)

`FOxMktCap` multiplies those two together, company by company. The answer is how much of each company is foreign-owned, in millions of riyals.

## Inputs
- `../../run/connection.md`

## Process

All three sheets are laid out the same way: one company per column, currently starting at column B (possible that the starting position changes), and one day per row going down. A company sits in the same column on every sheet, and a date sits on the same row on every sheet.

The formulas depend on that staying true. Steps 2 and 3 check it before anything is written, and step 4 checks there is data to work with. **If any of those checks fails, write nothing, add a line to `../../run/issues.md` saying what was wrong, and stop.**

Whenever this stage stops, the whole run stops. End the line in `issues.md` with "the run stopped", and skip straight to `stages/09_send-report/`.

1. **Open the workbook** and start a session. Everything below happens inside it.

2. **Check the companies line up.** Read the top two rows — `NOMU CODE` (row 1) and `TASI CODE` (row 2) — from all three sheets. Currently the companies start from column B. So column B must hold the same codes on all three, column C the same, and so on — up to the last column that has a code or a name in the heading rows. That last column moves as companies are added, so work it out fresh each run.

   If a column doesn't match, stop. Say which column it was and what each sheet had there. Don't just count how many companies each sheet has as a sheet could lose one company and gain another, and the count would still match while the columns no longer mean the same thing.

3. **Work out which row to fill in, and check the rows line up.**

   Read down column A of `FOxMktCap` to the last row that has a date. Call that the **last row**. The row directly below it is the **new row** — the one to fill in.

   Use those same two row numbers on all three sheets. Don't go looking for a matching date elsewhere — the point is to confirm the same row number refers to the same day on every sheet.

   Read column A at those two row numbers, on all three sheets, and check all three of these:

   - On the **last row**, all three sheets show the same date. This is the row being copied from, so it has to mean the same day everywhere.
   - On the **new row**, `Mkt_Cap` and `FO_Actual` both show a date, and it's the same date on both.
   - That date is **later** than the date on the last row.

   Then:

   - **Neither source sheet has a date on the new row**. Stages 02 and 03 didn't add today's row, so there's nothing to work out. Stop, and add a line to `../../run/issues.md` saying so. The stages run in order, so something went wrong earlier — whichever stage it was should already have logged the reason.
   - **Any of the three checks fails** — stop, and add a line to `../../run/issues.md` saying which check failed and what each sheet showed.
   - **All three pass** — carry on. The date the two source sheets show on the new row is the one to write in step 6.

4. **Check there are real numbers to multiply.** Look along the new row in `Mkt_Cap` and in `FO_Actual`.

   Stop only if either row is **completely** empty. That means the stage before it didn't run.

   A few empty cells along the row are not something to worry about.

5. **Check the new row is empty, then fill it in.**

   First look at the company cells on the new row. They should be empty. If anything is there:

   - **Formulas pointing at the new row's own number** — a previous run filled the row but stopped before writing the date. Carry on; writing the formulas again puts the same thing back, and step 6 finishes the job. Add a line to `../../run/issues.md` saying an earlier run stopped after copying the row but before writing the date, and that this run finished it.
   - **Anything else** — values, formulas pointing at a different row, or text. Stop, write nothing, and log what was found. Don't overwrite it.

   Then build the new row from the last row. For each company column: read the formula in the last row, change the row number inside it to the new row's number, and write the result into the new row. Read the last row's number formatting too and write that with it, so the new row looks like the ones above it.

   Where the last row has no formula — the divider column — leave that cell blank in the new row as well.

6. **Write the date** into column A of the new row, in `FOxMktCap`. Use the date from step 3.

7. **Check it worked, before closing the session.** Two things were written, by two separate requests, so check both.

   - The date cell in column A — it should show the date from step 3, written as a date rather than a plain number.
   - A few of the company cells — each should hold a formula pointing at the new row's number, not the old one. The answers should not all be zero.

8. **Turn older formula rows into plain numbers.** Only the latest 4 rows keep formulas, so the file stays light. The new row counts as one of the 4.

   Any row above those 4 that still has formulas gets replaced by the numbers it shows, like Copy, then Paste as values in Excel. Usually that's one row. Keep the number formatting as it is.

   - Leave the `TASI END` divider column blank.
   - If a row shows an error (like `#N/A`) in any cell, leave that row as formulas. Add a line to `../../run/issues.md` naming the row's date and the cell, and carry on.

   Then read back a few cells from each changed row. They should hold numbers, not formulas, and the same numbers as before.

9. **Close the session.**

## Outputs
- The `FOxMktCap` sheet, with one new row, and only the latest 4 rows holding formulas
- `../../run/issues.md` — added to wherever a step above says to

## Technical details

**The formula in every company cell** looks like this, where `B` is the company's column and `3888` is the row:

```
=+Mkt_Cap!B3888*FO_Actual!B3888
```

This stage never types that formula out from scratch. Step 5 reads it from the last row and changes the row number. The formula is written here only so a person can check that a row looks right.

**This sheet has five heading rows, not four.** `NOMU CODE`, `TASI CODE`, `Bloomberg_Code`, `NAME`, then `Sector`. The dates start on row 6. `Mkt_Cap` and `FO_Actual` have four heading rows and start on row 5. Despite that, a given date still sits on the same row number on all three sheets — which is why step 3 checks it rather than assuming it.

**Filling the new row** — there is no copy-and-paste in this API, so the new row is built from the last one. First read what the last row holds:

```
GET .../worksheets('FOxMktCap')/range(address='B3887:OH3887')?$select=formulas,numberFormat
```

Then write the new row, with the row number inside each formula changed to the new row's:

```
PATCH .../worksheets('FOxMktCap')/range(address='B3888:OH3888')
{
  "formulas": [["=+Mkt_Cap!B3888*FO_Actual!B3888", "...one per column..."]],
  "numberFormat": [["_(* #,##0.00_);_(* (#,##0.00);_(* \"-\"??_);_(@_)", "...one per column..."]]
}
```

Take the number formats from what the last row returned rather than typing them out. That way the new row always matches whatever the sheet currently uses.

**The rows and columns above are an example, not fixed.** Work out both ends fresh each run: a company column is one with a code or a name in the heading rows, so the first and last of those are where the row starts and ends. Don't assume `B` or `OH` — companies get added, and a column can be inserted anywhere. The row numbers come from step 3.

**Changing the row number:** `=+Mkt_Cap!B3887*FO_Actual!B3887` becomes `=+Mkt_Cap!B3888*FO_Actual!B3888`. Check afterwards that the old row number is gone from every formula built.

**An empty entry in `formulas` leaves the cell empty.** This is not true of `values`, where an empty entry written next to numbers becomes `0`. So the divider column is written as an empty entry in the formulas list, and stays blank.

**Writing the date** — dates are stored as Excel serial numbers, not text. Write the number and set the format in the same request, because a brand new row has no formatting of its own yet:

```
PATCH .../worksheets('FOxMktCap')/range(address='A3888')
{
  "values": [[46293]],
  "numberFormat": [["d-mmm-yy"]]
}
```

`46293` is 28 September 2026. To work out the number for a date, count the days since 30 December 1899.

**Session use:** open one session (`createSession`, `persistChanges: true`), do the reads and both writes inside it, then close it. `closeSession` sends no body, but some HTTP clients need `Content-Length: 0` set explicitly, or it fails with `411 Length Required` instead of `204`.

**Turning formulas into numbers** (step 8) — read the rows with `?$select=formulas,values,valueTypes`. A row has formulas if any company cell's formula starts with `=`. The new row is the bottom row. Keep formulas in the new row and the 3 rows above it. Start at the row above those 4, and go up until you reach a row with no formulas. For example, if the new row is 3888, keep 3885–3888, and start at 3884. Then write each of those rows' `values` back:

```
PATCH .../worksheets('FOxMktCap')/range(address='B3884:OH3884')
{
  "values": [[1234.56, "...one per column...", null, "..."]]
}
```

Put `null` for the divider column. A `null` leaves that cell unchanged, so it stays blank. Don't use an empty string there, because next to numbers it becomes `0`. This write doesn't touch number formats, so they stay as they are. A cell with `valueTypes` of `Error` means that row is skipped.

Requests can time out; try again rather than treating the first failure as final.
