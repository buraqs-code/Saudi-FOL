# 06_build_report: set the report date

The Task: set the report date in the `EMAIL.1` sheet. The report date is the last trading day. All figures in the report use this date. Do not write other cells.

The report goes out Sunday to Thursday. Each report shows the previous trading day:

| Run day | Report date |
|---|---|
| Sunday | Thursday |
| Monday | Sunday |
| Tuesday | Monday |
| Wednesday | Tuesday |
| Thursday | Wednesday |
| Friday, Saturday | No report |

## Inputs
- `../../run/connection.md`
- `../../run/3a_fo.json` — the `as_of_date`. This is the date on the foreign ownership (FO) page.

## Process

If a step tells you to stop, the run stops. Add a line to `../../run/issues.md`. End the line with "the run stopped". Go to `stages/08_send-report/`.

1. **Find the day of the week.** Use today's date in Pakistan time.

   If today is Friday or Saturday, do not change the report date. Add this line to `../../run/issues.md`:

   > No report: weekend run. The next report goes out on Sunday.

   Do not do stage 07. Go to `stages/08_send-report/`.

2. **Open the workbook.** Start a session. Do all steps below in this session.

3. **Get the new report date.** Read `as_of_date` from `../../run/3a_fo.json`. This date is the new report date.

   If `as_of_date` is empty, stop. Log that the FO date is missing.

4. **Find the date cell.** All figures in `EMAIL.1` use one date cell. Currently this cell is E3. Rows and columns can move. Thus, follow the formulas to find the cell:

   - Find the `Mkt Cap` column by its heading. Read the formula of the first company in it.
   - The formula uses a cell in row 1 of its column. Read that cell.
   - That cell points to a different cell. Follow the links until you find a cell with a plain date. That cell is the date cell.
   - Do the same from the `Current FO` column. It must go to the same cell.

   If the links do not end at a plain date, or the two columns go to different cells, stop. Log where the links ended and what was in that cell.

5. **Compare the new report date with the date cell.**

   - **New report date is later.** Go to step 6.
   - **Same date.** There is no new trading day. That day was in the last report. The cause can be a market holiday, or the FO page did not update. Do not write. Add this line to `../../run/issues.md`:

     > No report: no new trading day. The FO page shows 8 Oct 2026. This day was in the last report.

     Close the session. Do not do stage 07. Go to `stages/08_send-report/`.
   - **New report date is earlier.** Stop. Log the two dates.

6. **Make sure that the new report date has a row.** Read column A of `Mkt_Cap`, `FO_Actual` and `Diff_FOxMktCap`. Each sheet must have a row with the new report date. If a sheet does not have this row, stop. Log the sheet name and the date.

7. **Write the new report date** into the date cell.

8. **Do a check before you close the session.**

   - The date cell shows the new report date as a date, not as a number.
   - Some figures in the `Mkt Cap` and `Current FO` columns show numbers, not errors. An error tells you that a sheet does not have the date.

9. **Close the session.**

## Outputs
- The `EMAIL.1` sheet, with the new report date in the date cell
- `../../run/issues.md` — a "No report:" line when no report goes out, or a line when the run stops

## Technical details

**Today's day of the week** — use Pakistan time (PKT). PKT is UTC+5 all year, with no daylight saving. Get the time in UTC and add 5 hours. Do not use the time zone of the server.

**The FO date** — `as_of_date` has this format: `Oct 8, 2026`. Change it to a real date before you compare it.

**How the figures link to the date cell.** A figure in the `Mkt Cap` column currently has this formula:

```
=XLOOKUP($B7,Mkt_Cap!$B$2:$OH$2,INDEX(Mkt_Cap!$B$6:$OH$4999,XMATCH(E$1,Mkt_Cap!$A$6:$A$4999,0,-1),0),"")
```

The formula uses `E$1`. `E1` holds `=E3`. `E3` holds the plain date. These addresses are examples. Follow the formulas in the sheet.

Read formulas with `GET .../worksheets('EMAIL.1')/range(address='E7')?$select=formulas,values,valueTypes`. A plain date has a number in `formulas` (no `=`) and `valueTypes` of `Double`.

**The date cell holds a plain date, not a formula.** It is an Excel date serial number with the format `d.mmm.yy` (for example, `16.Sep.26`).

**Writing the date cell** — write the serial number and the format in one request:

```
PATCH .../worksheets('EMAIL.1')/range(address='E3')
{
  "values": [[46303]],
  "numberFormat": [["d.mmm.yy"]]
}
```

`46303` is 8 October 2026. To get the number for a date, count the days from 30 December 1899.

**Why all three sheets need the date.** The `Mkt Cap` and `Current FO` columns find the report date in `Mkt_Cap` and `FO_Actual`. They need an exact match. If there is no match, they show an error. The change columns add values from `Diff_FOxMktCap` up to the report date. If a day is missing there, the number is wrong but no error shows.

**Why the report date is not the last row.** Each run adds a row for every calendar day. Fridays, Saturdays and holidays get a copy of the last trading day. On Sunday, the last row is Saturday. But the FO page shows Thursday. The report must show Thursday.

**Do not change the other dates in row 1.** They are fixed year-end dates for comparisons.

**The sheet calculates automatically.** If a check in step 8 shows old figures, read again.

**Session use:** open one session (`createSession`, `persistChanges: true`). Do the reads and the write in it. Then close it. `closeSession` sends no body. Some HTTP clients must set `Content-Length: 0`. If not, the result is `411 Length Required`, not `204`.

Requests can time out. Try again. Do not think that the first failure is final.
