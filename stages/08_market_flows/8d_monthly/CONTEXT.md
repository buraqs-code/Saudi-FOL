# 8d_monthly: put the new monthly report into the workbook

The Task: write the new monthly report into the market flows workbook's monthly (`M.`) sheets, then check that the monthly charts show it.

## When this runs

Only when step 8b saved `../../../run/8b_monthly.pdf`. If it did not, skip this folder.

## The steps

Do the steps in this order. Each step has its own folder. The steps match 8c, with the monthly sheets.

| Step | What it does | Status |
|---|---|---|
| `8d1_read-report/` | Read the two Main Market tables from the PDF into JSON. Check them. Does not change the workbook | Written. Not run yet |
| `8d2_update-sheets/` | Add a row for the new month to the history, summary and email sheets. Freeze the old history rows. Write the `.dl` sheets. Write the date last | Written. Not run yet. Its checks pass on the live workbook |
| `8d3_check-charts/` | Check that the four charts on `M.Email` show the new row. They follow the data by themselves. Does not change anything | Written. Not run yet |

Step 8d1 only reads. If it fails, the workbook is not changed. Step 8d3 only checks the charts. Step 8e exports them, on every run.

## If something goes wrong

Each step tells you what to do. A failure never stops the run. Add a line to `../../../run/issues.md` that starts with "Market flows:". Do not end it with "the run stopped".

## Next

After 8d3, go to `../8e_export-graphs/`.
