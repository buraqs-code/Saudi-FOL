# 8c2_update-sheets: add the new week to the workbook

The Task: add a row for the new week to the history, summary and email sheets. Then write the new week to `W.Flows.dl` and `W.Own.dl`. Write the date last.

Keep the steps in this order. The newest history row reads the `.dl` sheets through formulas. You must freeze it before the `.dl` sheets change. If you do not, last week's row shows this week's figures.

## Inputs
- `../../../../run/8c1_weekly.json`: the report date and the checked figures
- `../../../../run/connection.md`: the market flows workbook's `itemId`

## Process

Open a session on the market flows workbook. Do all the steps in this session.

1. **Check before you write.** Do not write anything in this step. If a check fails, stop. Go to "If something fails".
   - **Find the state.** Read the date in `W.Flows.dl`, in the cell after `Date:`. This is the ".dl date". Read the last date in column A of `W.Flows.history`. Only two states are correct:
     - **Normal:** the last history date is the .dl date. The new row is the row under it.
     - **Resume:** the last history date is the report date, and the row above it has the .dl date. An earlier run added rows and then failed. The new row is the last row.

     Any other state means that the sheets are out of step. Stop. A person must look at them.
   - **Check that the sheets line up.** Summary and Email read history by row number, so all four sheets must agree. In `W.Flows.history`, `W.Own.history`, `W.Flows.Summary` and `W.Email`, the row above the new row must show the same date. The new row must be empty or show the report date.
   - **Check the history formulas.** In both history sheets, every formula cell in the newest formula row must hold the expected formula. See "Technical details".
   - **Check the history labels.** In both history sheets, each column's label must agree with the `.dl` cell that the column reads. See "Technical details".

2. **Add the new row to each sheet.** Use the new row number from step 1 in all four sheets. Do the sheets in this order: `W.Flows.history`, `W.Own.history`, `W.Flows.Summary`, `W.Email`. For each sheet:
   - If the new row already shows the report date, skip this sheet. An earlier run added it.
   - If not, copy the row above down into the new row. Where column A holds a typed date, write the report date. See "Copying a row down".

   The new rows show last week's figures for now. Step 4 corrects them.

3. **Freeze the old history rows.** In `W.Flows.history` and `W.Own.history`, change the row above the new row to values. Do this only if that row still has formulas. Do not freeze rows in `W.Flows.Summary` or `W.Email`. They keep formulas on every row.

4. **Write the values to the `.dl` sheets.** Find each row by its label, not by a fixed cell.
   - **`W.Flows.dl`:** on each of the 16 rows, write Buy, % of Total Buys, Sell, % of Total Sells, Net and Difference. On the Grand Total row, write the total Buy and the total Sell in riyals. Write 100% in both percentage columns. Write 0 in Net and Difference. Values replace any old formulas. This is correct.
   - **`W.Own.dl`:** on each of the 16 rows and on the Grand Total row, write Holding Value, Issued (%) and Free Float (%).
   - Write each percentage as a fraction. For example, 22.18% becomes 0.2218.
   - Do not change the label columns (the key, Nationality and Investor Type). Do not change the Checks row.

5. **Check the workbook.** If a check fails, stop. Go to "If something fails".
   - The `.dl` cells agree with the JSON.
   - On each `.dl` sheet, the Checks row is near 0: 2 or less for riyals, 0.0005 or less for percentages.
   - Each new history row now shows the new week. Each value equals the `.dl` value that rows 1 and 2 of its column point to.
   - The frozen history rows have no formulas.
   - The new Summary and Email rows show the report date and have no errors.

6. **Write the date last.** Write the report date in `W.Flows.dl`, in the cell after `Date:`. Then close the session. Step 8a reads this date next week.

## If something fails

Do not write the date. Add a line to `../../../../run/issues.md` that starts with "Market flows:" and says what failed. Do not end it with "the run stopped". If step 2 added rows, say so: until the next run, they can show last week's figures under the new date.

The old date stays, so the next run tries again. Step 1 then finds the resume state, and step 2 skips the rows that are already there.

Skip 8c3. Go to `../../8d_monthly/` if `../../../../run/8b_monthly.pdf` exists. If it does not, go to `../../8e_export-graphs/`.

## Outputs
- A new row in `W.Flows.history`, `W.Own.history`, `W.Flows.Summary` and `W.Email`
- `W.Flows.dl` and `W.Own.dl`, with the new week's values and date
- `../../../../run/issues.md`, added to only if something failed

## Next

Go to `../8c3_check-charts/`.

## Technical details

*Required to run this step.*

**Row names.** The report and the `.dl` sheets use different names for some investor types. The 16 rows are in the same order in both. Before you write, make sure the Nationality and Investor Type of each row agree with this table. If a row does not agree, do not write. Go to "If something fails".

| # | Report (JSON) | Sheet: Nationality | Sheet: Investor Type |
|---|---|---|---|
| 1 | Saudi / Individuals / Retail | Saudi | Retail |
| 2 | Saudi / Individuals / HNWIs | Saudi | HNWIs |
| 3 | Saudi / Individuals / IPIs | Saudi | IPIs |
| 4 | Saudi / Individuals / DPMs | Saudi | Individual DPMs |
| 5 | Saudi / Institutions / Corporates | Saudi | Corporates |
| 6 | Saudi / Institutions / Mutual Funds | Saudi | Mutual Funds |
| 7 | Saudi / Institutions / GRE's | Saudi | GREs |
| 8 | Saudi / Institutions / DPMs | Saudi | Institutional DPMs |
| 9 | GCC / Individuals | GCC | Individuals |
| 10 | GCC / Institutions | GCC | Institutions |
| 11 | GCC / DPMs | GCC | GCC DPMs |
| 12 | Foreign / Individuals | Foreign | Individuals |
| 13 | Foreign / Institutions | Foreign | Institutions |
| 14 | Foreign / SWAP Holders | Foreign | SWAP Holders |
| 15 | Foreign / DPMs | Foreign | Foreign DPMs |
| 16 | Foreign / Strategic Investor | Foreign | Strategic Investor |

**Finding the cells in the `.dl` sheets.** Row 1 holds the column headings. Find each column by its full heading, e.g. "Holding Value (SAR)": column A of `W.Own.dl` is also headed "Holding Value".
- The first data row has `SaudiRetail` in column A (currently row 2).
- The Grand Total row has `Grand Total` in column B (currently row 18).
- The Checks row has `Checks` in column A or B (currently row 23).
- The date is in the cell after `Date: ` in row 1 of `W.Flows.dl` (currently L1). `W.Own.dl` has no date.

**The history sheets.** Row 1 ("Column") and row 2 ("Row") say which `.dl` cell each column reads: the column number and the investor row, counted inside the `.dl` data block from A2. Row 3 is a label. The weekly rows start under the "Date" row (currently row 7). The last row is the last one with a date in column A (currently row 119). Do not change rows 1 to 6.

**The expected history formulas.** Read the row with `?$select=formulasR1C1`. Every cell from column B on must hold:
- `W.Flows.history`: `=INDEX(W.Flows.dl!R2C1:R17C9,W.Flows.history!R2C,W.Flows.history!R1C)`
- `W.Own.history`: `=INDEX(W.Own.dl!R2C1:R18C6,W.Own.history!R2C,W.Own.history!R1C)`

**The label check.** For each history column, rows 1 and 2 point to one `.dl` cell. Take that cell's key (column A of its row) and heading (row 1 of its column).
- `W.Flows.history`: row 3 must be the key and the heading joined, e.g. `SaudiRetail` + `Buy (SAR)` = `SaudiRetailBuy (SAR)`.
- `W.Own.history`: row 3 must be the key, e.g. `SaudiRetail`.

If a label does not agree, someone moved a row or a column. The column would copy the wrong figure.

**Copying a row down.** `N` is the last row. Use the full width of the sheet.
1. Read row `N+1`. It must be empty. If it is not, stop.
2. Read row `N` with `?$select=formulasR1C1,numberFormat,valueTypes`. If any `valueTypes` entry is `"Error"`, stop.
3. Write the same `formulasR1C1` and `numberFormat` arrays to row `N+1` with `PATCH .../range(address='A{N+1}:{last column}{N+1}')`. Do not edit the formula strings. Replace a typed date (an entry that does not start with `=`) with the report date.
4. The reply's `address` must end in row `N+1`.

The full process is in `copying-formulas-down.md`, at the top of the project.

**Freezing a row.** Read the row's `values`. Write them back to the same range with `PATCH` and the body `{"values": [[...]]}`.

**Writing.** Use a session with `persistChanges: true`. Send the header `workbook-session-id` on every request. Write one block per sheet: `PATCH /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets('{sheet}')/range(address='<first cell>:<last cell>')` with the body `{"values": [[...], [...]]}`, one inner list per row. OneDrive can take about 20 seconds to save the changes. Do the checks inside the same session.

**The date.** Write it as an Excel date number: the count of days since 30 December 1899. For example, 1 Oct 2026 is `46296`.
