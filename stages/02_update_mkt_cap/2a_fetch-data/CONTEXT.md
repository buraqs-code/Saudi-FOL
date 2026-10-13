# 2a_fetch-data: pull the day's market cap data from saudiexchange.sa

The Task: fetch TASI and NOMU market cap data from the Saudi Exchange website and save it as one JSON file the next step reads.

## Inputs
- `../../../config.yaml`: The `sources.market_cap` section gives the TASI and NOMU page links where you go to fetch the data. 

## Process

1. **Fetch the TASI page:** The link is given as `sources.market_cap.tasi` in the config.

2. **Read the market date:** Shown near the top of the page, after the label "Market Date", e.g. `Market Date 2026/09/30`. Keep it exactly as shown.

3. **Find the right table:** The page has several tables. The table we're looking for contains an extensive list of the companies along with their Symbol (Numeric Tadawul Code), Company Names, and their Market Cap in SAR. The table has other columns too that we're not concerned with.

   Symbol, Company and Market Cap aren't next to each other in the table. Other columns sit in between. The header words can change a little, for example `Market Cap(SAR)` or `Market Cap (SAR)`. Use your judgement to find the three columns by their meaning.

   Keep the market cap exactly as the page shows it, with no rounding, but save it as a plain number: `825,000,000.00` becomes `825000000.00`.

   **If no table has these three columns, or more than one table has them:** the site's layout has changed. Do not choose a table at random. A wrong table gives a wrong market cap for every company. Treat it the same as a fetch failure (step 6): diagnose first, then log it and continue.

4. **Do the same for the NOMU page**. The link is given as `sources.market_cap.nomu` in the config. Same table, same three columns, same market-date location.

5. **Write one JSON file** — the market date, which page(s) were actually fetched successfully, then every company: all TASI companies first, in the order the table lists them, then all NOMU companies, same way. Refer to "Technical details" for the exact shape.

6. **If a page fails to load**, first try to work out why and fix it yourself. If it still won't load after that, don't stop the run. Add a line to `../../../run/issues.md` saying which page failed and why, and carry on with whatever data you do have (e.g. write the JSON with TASI only if NOMU is the one that failed). If both pages failed, still write the JSON, empty — see "Technical details".

## Outputs
- `../../../run/2a_mkt_cap.json` containing the market date and the combined company list
- `../../../run/issues.md` — only added to if a page couldn't be fetched

## Technical details

**Fetching the pages.** Plain requests always fail here as the site fingerprints the connection itself, not just headers. Use `curl_cffi`, impersonating a real browser:
- Try `impersonate="safari184"` first.
- If that fails, retry with `impersonate="firefox133"`.

**The company table** is identified by its header row, not its heading. An example of its header row (the words and spaces can change): `Symbol, Company, Open (SAR), High (SAR), Low (SAR), Close (SAR), % Change, Volume Traded, Value Traded (SAR), No. of Trades, Market Cap (SAR)`.

**The JSON shape example:**
```json
{
  "market_date": "2026/09/30",
  "pages_fetched": ["TASI", "NOMU"],
  "companies": [
    { "symbol": "2030", "company": "SARCO", "market_cap_sar": 825000000.00 }
  ]
}
```
`companies` is one flat list — TASI entries first, then NOMU entries, in the order each table lists them. `market_cap_sar` is the page's figure as a plain number, commas removed.

`pages_fetched` lists exactly which of `"TASI"` / `"NOMU"` were successfully read this run.

If both pages failed, still write this file, with `"market_date": null`, `"pages_fetched": []` and `"companies": []`. The next step checks for this to know nothing was fetched.
