# 05_calculate_Diff_FOxMktCap: add today's row to the Diff_FOxMktCap sheet

The Task: add one new row to the `Diff_FOxMktCap` sheet, so the workbook works out the riyal value of the day's change in foreign ownership, for each company.


This sheet calculates

> previous day's market cap × (today's foreign ownership − previous day's foreign ownership)

"Previous day" means the row above. New rows are added every day, but older rows skip weekends and holidays, so the row above isn't always the calendar day before.

This sheet's new row needs numbers from **two different days**, not one:

- `Mkt_Cap` — the **previous day's** row only
- `FO_Actual` — **both** the previous day's and today's row

## Inputs
- `../../run/connection.md`

## Process

All four sheets involved (`Mkt_Cap`, `FO_Actual`,`Diff_FOxMktCap`) are laid out the same way: one company per column, and one day per row going down. A company sits in the same column on every sheet, and a date sits on the same row on every sheet.

The formula depends on that staying true. Steps 2 and 3 check it before anything is written, and step 4 checks there is data to work with. **If any of those checks fails, write nothing, add a line to `../../run/issues.md` saying what was wrong, and stop.**

Whenever this stage stops, the whole run stops. End the line in `issues.md` with "the run stopped", and skip straight to `stages/08_send-report/`.

1. **Open the workbook** and start a session. Everything below happens inside it.

2. **Check the companies line up.** Read the top two rows — `NOMU CODE` (row 1) and `TASI CODE` (row 2) — from `Mkt_Cap`, `FO_Actual`, and `Diff_FOxMktCap`.

   A company column is one with a code or a name in the heading rows. Work out which columns those are on each sheet, then compare them in order: the first company column must hold the same codes on all three sheets, the second the same, and so on to the last one.

   If a column doesn't match, stop. Say which column it was and what each sheet had there. Don't just count how many companies each sheet has as a sheet could lose one company and gain another, and the count would still match while the columns no longer mean the same thing.

3. **Work out which row to fill in, and check the rows line up.**

   Read down column A of `Diff_FOxMktCap` to the last row that has a date. Call that the **last row**. The row directly below it is the **new row** — the one to fill in.

   Use those same two row numbers on all three sheets. Don't go looking for a matching date elsewhere — the point is to confirm the same row number refers to the same day on every sheet.

   Read column A at those two row numbers, on all three sheets, and check all three of these:

   - On the **last row**, all three sheets show the same date. This is the row the new formula points back to, so it has to mean the same day everywhere.
   - On the **new row**, `Mkt_Cap` and `FO_Actual` both show a date, and it's the same date on both.
   - That date is **later** than the date on the last row.

   Then:

   - **Neither source sheet has a date on the new row.** Stages 02 and 03 didn't add today's row, so there's nothing to work out. Stop, and add a line to `../../run/issues.md` saying so. The stages run in order, so something went wrong earlier — whichever stage it was should already have logged the reason.
   - **Any of the three checks fails** — stop, and add a line to `../../run/issues.md` saying which check failed and what each sheet showed.
   - **All three pass** — carry on. The date the two source sheets show on the new row is the one to write in step 6.

4. **Check there are real numbers to work with.** This sheet's formula needs three things:

   - `Mkt_Cap`, on the **last row**
   - `FO_Actual`, on the **last row**
   - `FO_Actual`, on the **new row**

   Stop only if any of those three is **completely** empty. That means an earlier stage or an earlier run of this one didn't go through.

   A few empty cells dotted along a row are normal — those companies come out as `-`. 

5. **Check the new row is empty, then fill it in.**

   First look at the company cells on the new row. They should be empty. If anything is there:

   - **Formulas pointing at the new row's own number** — a previous run filled the row but stopped before writing the date. Carry on; writing the formulas again puts the same thing back, and step 6 finishes the job. Add a line to `../../run/issues.md` saying an earlier run stopped after filling the row but before writing the date, and that this run finished it.
   - **Anything else** — values, formulas pointing at a different row, or text. Stop, write nothing, and log what was found. Don't overwrite it.

   Then build the new row from the last row. For each company column: read the formula in the last row, and change **both** row numbers inside it to move forward by one — the reference to the last row becomes a reference to the new row, and the reference to the row before that becomes a reference to the last row. Read the last row's number formatting too and write that with it, so the new row looks like the ones above it.

   Where the last row has no formula — the divider column — leave that cell blank in the new row as well.

6. **Write the date** into column A of the new row, in `Diff_FOxMktCap`. Use the date from step 3.

7. **Check it worked, before closing the session.** Two things were written, by two separate requests, so check both.

   - The date cell in column A — it should show the date from step 3, written as a date rather than a plain number.
   - A few of the company cells — each should hold a formula pointing at the new row and the last row, and nothing earlier than that. The answers should not all be zero — unless FO figures were copied from the row above today (a line in `../../run/issues.md` starting "Market closed:" or "FO page behind:"). Then all zeros is expected.

8. **Turn older formula rows into plain numbers.** Only the latest 4 rows keep formulas, so the file stays light. The new row counts as one of the 4.

   Any row above those 4 that still has formulas gets replaced by the numbers it shows, like Copy, then Paste as values in Excel. Usually that's one row. Keep the number formatting as it is.

   - Leave the `TASI END` divider column blank.
   - If a row shows an error (like `#N/A`) in any cell, leave that row as formulas. Add a line to `../../run/issues.md` naming the row's date and the cell, and carry on.

   Then read back a few cells from each changed row. They should hold numbers, not formulas, and the same numbers as before.

9. **Close the session.**

## Outputs
- The `Diff_FOxMktCap` sheet, with one new row, and only the latest 4 rows holding formulas
- `../../run/issues.md` — added to wherever a step above says to

## Technical details

**The formula in the new row's company cells** looks like this. Here the new row is `3889`, the last row is `3888`, and `B` is the company's column:

```
=+Mkt_Cap!B3888*(FO_Actual!B3889-FO_Actual!B3888)
```

This stage never types that formula out from scratch. Step 5 reads it from the last row and changes the row numbers. The formula is written here only so a person can check that a row looks right.

**This formula has two row numbers in it, not one — both need shifting by the same amount.** Compare the last row's formula to the one before it to see the pattern:

```
row 3888: =+Mkt_Cap!B3887*(FO_Actual!B3888-FO_Actual!B3887)
row 3889: =+Mkt_Cap!B3888*(FO_Actual!B3889-FO_Actual!B3888)
```

`3887` became `3888`, and `3888` became `3889` — every row number in the formula moves forward by the same one row. Change every one of them; changing only one would break the formula's meaning.

**This sheet has five heading rows, not four.** `NOMU CODE`, `TASI CODE`, `Bloomberg_Code`, `NAME`, then `Sector`. The dates start on row 6. `Mkt_Cap` and `FO_Actual` have four heading rows and start on row 5. Despite that, a given date still sits on the same row number on all three sheets — which is why step 3 checks it rather than assuming it.

**Filling the new row** — there is no copy-and-paste in this API, so the new row is built from the last one. First read what the last row holds:

```
GET .../worksheets('Diff_FOxMktCap')/range(address='B3888:OH3888')?$select=formulas,numberFormat
```

Then write the new row, with both row numbers inside each formula shifted forward by one:

```
PATCH .../worksheets('Diff_FOxMktCap')/range(address='B3889:OH3889')
{
  "formulas": [["=+Mkt_Cap!B3888*(FO_Actual!B3889-FO_Actual!B3888)", "...one per column..."]],
  "numberFormat": [["...whatever the last row returned...", "...one per column..."]]
}
```

Take the number formats from what the last row returned rather than typing them out. That way the new row always matches whatever the sheet currently uses.

**The rows and columns above are an example, not fixed.** Work out both ends fresh each run: a company column is one with a code or a name in the heading rows, so the first and last of those are where the row starts and ends. Don't assume `B` or `OH` — companies get added, and a column can be inserted anywhere. The row numbers come from step 3.

A correct formula still contains the last row's number, because it's the previous-day reference, so don't treat that as a leftover.

**An empty entry in `formulas` leaves the cell empty.** This is not true of `values`, where an empty entry written next to numbers becomes `0`. So the divider column is written as an empty entry in the formulas list, and stays blank.

**Writing the date** — dates are stored as Excel serial numbers, not text. Write the number and set the format in the same request, because a brand new row has no formatting of its own yet:

```
PATCH .../worksheets('Diff_FOxMktCap')/range(address='A3889')
{
  "values": [[46294]],
  "numberFormat": [["d-mmm-yy"]]
}
```

To work out the number for a date, count the days since 30 December 1899.

**Session use:** open one session (`createSession`, `persistChanges: true`), do the reads and both writes inside it, then close it. `closeSession` sends no body, but some HTTP clients need `Content-Length: 0` set explicitly, or it fails with `411 Length Required` instead of `204`.

**Turning formulas into numbers** (step 8) — read the rows with `?$select=formulas,values,valueTypes`. A row has formulas if any company cell's formula starts with `=`. Start at the fifth row from the bottom and go up until you reach a row with no formulas. Then write each of those rows' `values` back:

```
PATCH .../worksheets('Diff_FOxMktCap')/range(address='B3884:OH3884')
{
  "values": [[1234.56, "...one per column...", null, "..."]]
}
```

Put `null` for the divider column. A `null` leaves that cell unchanged, so it stays blank. Don't use an empty string there, because next to numbers it becomes `0`. This write doesn't touch number formats, so they stay as they are. A cell with `valueTypes` of `Error` means that row is skipped.

Requests can time out; try again rather than treating the first failure as final.
