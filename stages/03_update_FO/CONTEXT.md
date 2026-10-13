# 03_update_FO: fetch foreign ownership data and write it into the workbook

This stage reads foreign ownership figures for every company off the Saudi Exchange's Foreign Ownership page and writes them into the workbook's `FO_Actual` sheet.

## The steps

This step is split into two stages. Each sub-step has its own folder and its own instructions.

| Step | What it does |
|---|---|
| `3a_fetch-data/` | Fetch the day's foreign ownership data from saudiexchange.sa |
| `3b_write-to-sheet/` | Write that data into the `FO_Actual` sheet |
