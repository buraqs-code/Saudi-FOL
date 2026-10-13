# 8c3_check-charts: check that the weekly charts show the new week

The Task: check that the four charts on `W.Email` include the row that step 8c2 added. This step does not change the charts. Step 8e exports them, on every run.

The charts follow the data by themselves. They read from named ranges, which grow when a row is added. You do not need to update them. This step only checks that it worked.

## Inputs
- `../../../../run/connection.md`: the market flows workbook's `itemId`

## Process

Open a session on the market flows workbook. This step only reads.

1. **Count the dates.** On `W.Email`, count the dates in column A, from row 7 down. Note the last date, and the last row that has one.

2. **Check each chart.** List the charts on `W.Email`. For each chart:
   - Read the values of the chart's series.
   - The number of values must equal the number of dates from step 1.
   - The last value must equal one of the values on the last row that has a date. Allow a difference of 0.000001. You do not need to know which column the chart shows. If the chart did not get the new row, its last value is last week's, and it does not agree.

3. **If a chart is one value short, wait 30 seconds and check it once more.** OneDrive can take that long to save the new row.

4. **If every chart passes,** add a line to `../../../../run/issues.md` that starts with "Market flows updated:". For example: "Market flows updated: the weekly charts now go to 8 Oct 2026." Use the last date from step 1. Stage 09 shows this line in the "Run Succeeded" email. It is not an issue. Then go to "Next".

## If something fails

Do not change the chart. Add a line to `../../../../run/issues.md` that starts with "Market flows:". Name each chart that failed and say what was wrong. Use the chart's title. For example: "Market flows: the weekly chart '<title>' has 113 values but the sheet has 114 dates. It may not show the new week." Do not end it with "the run stopped".

Step 8e still exports the charts as they are.

## Outputs
- `../../../../run/issues.md`: a "Market flows updated:" line if every chart passed, or a "Market flows:" line for each chart that failed

## Next

Go to `../../8d_monthly/` if `../../../../run/8b_monthly.pdf` exists. If it does not, go to `../../8e_export-graphs/`.

## Technical details

*Required to run this step.*

**Listing the charts.** `GET /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets('W.Email')/charts`. Each chart has a `name` (for example `Chart 2`). Read its title with `GET .../charts('{name}')/title`. The title is in `text`.

**Reading the values.** `GET .../charts('{name}')/series/itemAt(index=0)/points`. Each item in `value` has a `value`. Each chart has one series.

**Reading the sheet.** `GET .../worksheets('W.Email')/usedRange?$select=address,values`. Dates are the numbers in column A, from row 7 down.

Graph can show the values of a chart. It cannot show the cell range that a chart reads from, so this step compares values.
