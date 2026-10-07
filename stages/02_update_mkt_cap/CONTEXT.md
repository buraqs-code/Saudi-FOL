# 02_update_mkt_cap: fetch market cap data and write it into the workbook

This stage reads market cap figures for TASI and NOMU companies off the Saudi Exchange website and writes them into the workbook's `Mkt_Cap` sheet.

## The steps

This step is split into two stages. Each sub-step has its own folder and its own instructions.

| Step | What it does |
|---|---|
| `2a_fetch-data/` | Fetch the day's market cap data from saudiexchange.sa (TASI and NOMU) |
| `2b_write-to-sheet/` | Write that data into the `Mkt_Cap` sheet |

