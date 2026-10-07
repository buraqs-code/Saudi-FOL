# 06_build_report: point the report at the day just added

The Task: change the report's date — the one date cell in the `EMAIL.1` sheet that every figure works from, currently E3 — to the day the earlier stages just added. Nothing else in the sheet is written.

## Inputs
- `../../run/connection.md`

## Process

When a step below says to stop because something is wrong, the whole run stops. End the line in `../../run/issues.md` with "the run stopped", and skip straight to `stages/08_send-report/`. The one exception is step 3's "same date" case. Nothing is wrong there, so carry on to stage 07.

1. **Open the workbook** and start a session. Everything below happens inside it.

2. **Work out the report date.** Read down column A of `Mkt_Cap`, `FO_Actual` and `Diff_FOxMktCap` to the last row that has a date, on each sheet.

   All three must show the same date. That is the report date. If they don't match, an earlier stage didn't finish: write nothing, add a line to `../../run/issues.md` saying what each sheet showed, and stop.

3. **Find the report's date cell by following the links.** Every figure in `EMAIL.1` works from one date cell, currently E3. Rows and columns get added to this sheet, so don't go by where the cell sits. Follow the formulas to it instead:

   - Find the `Mkt Cap` column by its heading, and take the first company's figure in it.
   - Read its formula. It looks the date up from a cell in row 1 of its column.
   - Read that cell. It's a formula pointing at another cell. Keep following until you reach a cell holding a plain date, not a formula or text. That's the date cell.
   - Do the same from the `Current FO` column. It must lead to the same cell.

   If the trail doesn't end at a plain date (it ends at text, a blank cell or another sheet), or the two columns lead to different cells, the stage can't tell which cell drives the report. Write nothing, add a line to `../../run/issues.md` saying where the trail ended and what was there, and stop.

   Then compare the report date with the date in that cell:

   - **Report date is later** — carry on.
   - **Same** — the report is already on that day. Nothing to write. Close the session and go on to stage 07.
   - **Earlier** — something is wrong. Write nothing, add a line to `../../run/issues.md` saying what the cell held and what the report date was, and stop.

4. **Write the report date into that cell.**

5. **Check it worked, before closing the session.**

   - The date cell shows the report date, written as a date rather than a plain number.
   - A few figures lower down in the `Mkt Cap` and `Current FO` columns show numbers, not errors. An error means one of the three sheets doesn't have that date.

6. **Close the session.**

## Outputs
- The `EMAIL.1` sheet, with the date cell set to the new day
- `../../run/issues.md` — added to wherever a step above says to

## Technical details

**How the figures link to the date cell.** A figure in the `Mkt Cap` column currently looks like this:

```
=XLOOKUP($B7,Mkt_Cap!$B$2:$OH$2,INDEX(Mkt_Cap!$B$6:$OH$4999,XMATCH(E$1,Mkt_Cap!$A$6:$A$4999,0,-1),0),"")
```

The date it uses is `E$1`: the cell matched against the dates in column A of `Mkt_Cap`. `E1` holds `=E3`, and `E3` holds the plain date. The addresses here are examples; follow whatever the formulas actually say.

Read formulas with `GET .../worksheets('EMAIL.1')/range(address='E7')?$select=formulas,values,valueTypes`. A plain date has a number in `formulas` (no `=`) and `valueTypes` of `Double`.

**The date cell holds a plain date, not a formula** — an Excel date serial number, shown with the format `d.mmm.yy` (e.g. `16.Sep.26`).

**Writing the date cell** — write the serial number and set the format in the same request:

```
PATCH .../worksheets('EMAIL.1')/range(address='E3')
{
  "values": [[46294]],
  "numberFormat": [["d.mmm.yy"]]
}
```

`46294` is 29 September 2026. To work out the number for a date, count the days since 30 December 1899. The dates in column A of the three sheets are already stored this way, so the report date can be copied across as it is.

**Why all three sheets need the date.** The `Mkt Cap` and `Current FO` columns look the report date up in `Mkt_Cap` and `FO_Actual` and need an exact match, or they show an error. The change columns add up values from `Diff_FOxMktCap` up to that date, so a missing day there gives a wrong number rather than an error. That's why step 2 checks all three.

**Leave the other dates in row 1 alone.** They're fixed year-end dates used for comparisons. They aren't part of the daily update.

**The sheet recalculates by itself.** The workbook's calculation mode is automatic, so nothing needs triggering after the date cell changes. If a check in step 5 reads the old figures, read again.

**Session use:** open one session (`createSession`, `persistChanges: true`), do the reads and the write inside it, then close it. `closeSession` sends no body, but some HTTP clients need `Content-Length: 0` set explicitly, or it fails with `411 Length Required` instead of `204`.

Requests can time out; try again rather than treating the first failure as final.
