# 8c1_read-report: read the weekly tables from the report

The Task: read two Main Market tables from the weekly report into a JSON file. Check the figures. This step does not change the workbook.

## Inputs
- `../../../../run/8b_weekly.pdf`: the new weekly report
- `../../../../run/8a_check.json`: the report date (`weekly.site_date`)

## Process

Use the method in "Reading the PDFs", in `../../CONTEXT.md`.

1. **Read the value traded table.** Currently, its title is "Main Market Value Traded Breakdown". Write all its rows and totals to `../../../../run/8c1_weekly.json`, under `value_traded`.

2. **Read the ownership table.** Currently, its title is "Main Market Ownership Breakdown". The table can show one week or several. Find the column group headed "As of" + the report date. Use only its three columns:
   - Holding Value
   - Issued (%)
   - Free Float (%)

   Do not use any other week or the "Weekly Change" columns. Write the figures to the same JSON, under `ownership`.

   If no heading shows the report date, stop. Go to "If something fails".

3. **Check the JSON.** Do the sum checks, then the picture check. If a check still fails after one re-read, stop. Go to "If something fails".

## If something fails

Add a line to `../../../../run/issues.md` that starts with "Market flows:" and says what failed. Do not end it with "the run stopped". The workbook is not changed.

Skip 8c2 and 8c3. Go to `../../8d_monthly/` if `../../../../run/8b_monthly.pdf` exists. If it does not, go to `../../8e_export-graphs/`.

## Outputs
- `../../../../run/8c1_weekly.json`
- `../../../../run/issues.md`, added to only if something failed

## Next

When both checks pass, go to `../8c2_update-sheets/`.

## Technical details

*Required to run this step.*

**The JSON shape.** Keep names and figures as the report shows them. Percentages stay as shown (22.18, not 0.2218). `group` is `Individuals` or `Institutions` for Saudi rows, and `null` for the others. The Grand Total has `null` for Net and Difference, because the report leaves them blank.

```json
{
  "report_date": "2026-10-01",
  "value_traded": {
    "rows": [
      {"nationality": "Saudi", "group": "Individuals", "investor_type": "Retail",
       "buy_sar": 4109817324, "buy_pct": 22.18, "sell_sar": 3585874472, "sell_pct": 19.35,
       "net_sar": 523942852, "diff_pct": 2.83}
    ],
    "totals": [
      {"label": "Sub Total (Individuals)", "buy_sar": 7728612693, "buy_pct": 41.72,
       "sell_sar": 6553623317, "sell_pct": 35.37, "net_sar": 1174989376, "diff_pct": 6.34}
    ]
  },
  "ownership": {
    "as_of": "2026-10-01",
    "rows": [
      {"nationality": "Saudi", "group": "Individuals", "investor_type": "Retail",
       "holding_sar": 137702619818, "issued_pct": 1.51, "free_float_pct": 4.28}
    ],
    "totals": [
      {"label": "Grand Total", "holding_sar": 9123063858728, "issued_pct": 100.00, "free_float_pct": 100.00}
    ]
  }
}
```

Each table has 16 rows and 8 totals: Sub Total (Individuals), Sub Total (Institutions), Total Saudi Investors, Total GCC Investors, Total Foreign Investors, Grand Total, Institutional, Non-Institutional.
