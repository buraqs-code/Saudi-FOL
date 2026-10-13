# 8c_weekly: put the new weekly report into the workbook

The Task: write the new weekly report into the market flows workbook's weekly (`W.`) sheets, then check that the weekly charts show it.

## When this runs

Only when step 8b saved `../../../run/8b_weekly.pdf`. If it did not, skip this folder.

## The steps

Do the steps in this order. Each step has its own folder.

| Step | What it does | Status |
|---|---|---|
| `8c1_read-report/` | Read the two Main Market tables from the PDF into JSON. Check them. Does not change the workbook | Tested on the 1 Oct report |
| `8c2_update-sheets/` | Add a row for the new week to the history, summary and email sheets. Freeze the old history rows. Write the `.dl` sheets. Write the date last | Written. Not run yet. Only its checks and the `.dl` write have been tried |
| `8c3_check-charts/` | Check that the four charts on `W.Email` show the new row. They follow the data by themselves. Does not change anything | Written. Not run yet |

Step 8c1 only reads. If it fails, the workbook is not changed. Step 8c3 only checks the charts. Step 8e exports them, on every run.

## If something goes wrong

Each step tells you what to do. A failure never stops the run. Add a line to `../../../run/issues.md` that starts with "Market flows:". Do not end it with "the run stopped".

## Next

After 8c3, go to `../8d_monthly/` if `../../../run/8b_monthly.pdf` exists. If it does not, go to `../8e_export-graphs/`.
