# 08_market_flows: update the market flows workbook and make its graphs

This stage checks saudiexchange.sa for a newer weekly or monthly "Trading and Ownership By Nationality" report, writes any new one's figures into the **market flows workbook**, and gets the graphs on its `W.Email` and `M.Email` sheets ready for the report email. It works on the market flows workbook, never the main workbook.

**Status: still being designed.** Steps 8a and 8b are written and tested. Step 8c1 is tested. Steps 8c2 and 8c3 are written, but not run yet. Steps 8d1 to 8d3 are written, but not run yet. Step 8e is written, but not run yet.

## Before you start

Skip this whole stage and go straight to `stages/09_send-report/` if `../../run/issues.md` has a line starting:
- **"No report:"**. Stage 06 decided no report goes out that day.
- **"Market flows:"** from stage 01, meaning the market flows workbook couldn't be found.

## If something goes wrong

**This stage never stops the run.** The daily report doesn't depend on it. A failure gets a line in `../../run/issues.md` starting **"Market flows:"**, never ending with "the run stopped". Each step says where to go next.

## The steps

This stage is split into steps. Each one has its own folder and its own instructions. The weekly and monthly reports go into different sheets with different formulas, so each has its own folder. Only the ones whose report is new are run.

| Step | What it does | Runs |
|---|---|---|
| `8a_check-for-new-reports/` | Compare the dates in `W.Flows.dl` and `M.Flows.dl` with the newest weekly and monthly reports on the site | every time |
| `8b_download-reports/` | Download only the reports that are newer | only if one is new |
| `8c_weekly/` | Write the new weekly report into the `W.` sheets, then update the weekly charts. Split into 8c1–8c3, see its own `CONTEXT.md` | only if the weekly is new |
| `8d_monthly/` | Write the new monthly report into the `M.` sheets, then check the monthly charts. Split into 8d1–8d3, the same as 8c, see its own `CONTEXT.md` | only if the monthly is new |
| `8e_export-graphs/` | Get each chart on `M.Email` and `W.Email` as a picture, for the email. If that fails, make one PDF of the two sheets from a throwaway copy. Uses the charts as they are | every time, even when nothing was new |

When both reports are new, do 8c, then 8d, in the same run.

## Reading the PDFs

8c and 8d both read figures from the report PDFs. Both use this method.

The tables in the PDFs are pictures, not text. Turn each page into an image and read the image.

1. **Find the table by its title, not by its page number.** Pages can move.
2. **Use only the Main Market tables.** The report also has Nomu-Parallel Market tables with the same layout. Do not use them.
3. **Write the figures as the table shows them.** Do not round them. Do not change the names.
4. **Do the sum checks, then the picture check.** Write to the workbook only when both pass.

**Sum checks.** The totals in the table let you find a misread digit. The report rounds its figures, so small differences are normal. When you add rows, the rounding adds up. Allow 1 riyal, or 0.01 for a percentage, for each row in the sum. For example, the Saudi total adds 8 rows, so it must agree within 8 riyals, or 0.08. A check on one row must agree within 2 riyals, or 0.02.
- The rows of each group add up to their subtotal or total.
- The Saudi, GCC and Foreign totals add up to the Grand Total.
- Institutional plus Non-Institutional equals the Grand Total.
- On each value traded row, Buy minus Sell equals Net.
- Each percentage equals its riyal figure divided by the Grand Total, times 100. Free Float (%) is the exception: you cannot calculate it from riyals, so only check that it adds up.

**Picture check.** A sum check cannot find two rows that changed places, or a figure from the wrong column. Look at the table image again. Compare each figure in the JSON with the image, row by row.

**If a check fails.** Read that row again from the image, and correct it. Do all the checks again. If a check still fails, do not write to the workbook. The step tells you what to do next.

### Technical details

*Required to read the PDFs.*

Turn the pages into images with `pypdfium2`, at a scale of 2 or more, so that small digits are clear. In the weekly report, the Main Market tables are currently on pages 5 (value traded) and 6 (ownership). The Nomu-Parallel tables are on pages 8 and 9. The monthly report has the same layout on the same pages. It also has Sukuk/Bonds Market tables, currently on pages 11 and 12. Do not use them.
