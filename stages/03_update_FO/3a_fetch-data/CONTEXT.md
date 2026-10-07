# 3a_fetch-data: pull the day's foreign ownership data from saudiexchange.sa

The Task: fetch foreign ownership data from the Saudi Exchange website, convert it to the form the sheet needs, and save it as one JSON file the next step reads. 

## Inputs
- `../../../config.yaml`: The `sources.foreign_ownership` link is the page where you go to fetch the data.

## Process

1. **Fetch the page:** The link is given as `sources.foreign_ownership` in the config.

2. **Read the last updated date.** It's shown after the label "Last updated on", e.g. `Sep 30, 2026 3:00:00 AM`. Only the date matters here, so drop the time: `Sep 30, 2026`.

3. **Find the right table by its headers.** Currently there's only one table on the page, but the site may move things around in the future. Find the correct table the same way as the market cap pages: the table whose header row contains text like: **Symbol**, **Company** and **Total Foreign Ownership Actual**. Read those four columns by matching header text, not position. Ignore **Foreign Strategic Investors Ownership** and **Total Foreign Ownership Maxiumum Limit** entirely. We dont need them. 

   Each percentage cell reads like `"20.0%"` — strip the `%` to get the plain number (`20.0`), then also divide by 100 to get the decimal value the sheet actually stores (`0.20`). Keep both the plain number / percentage and the decimal in the JSON

   The exact header wording could change slightly. In that case use judgment to recognize the same four columns by meaning, not just an exact text match. Only treat it as "no table matches" if it's genuinely unclear which column is which.

   **If no table matches, or more than one does:** This means the site's layout has changed in a way these instructions don't account for. Don't guess. Diagnose first, then log it and continue (see step 5).

4. **Write one JSON file** — the date, then every row in the order the table lists them no need to split or reorder anything. See "Technical details" for the exact format.

5. **If the page fails to load, or the table can't be matched**, first try to work out why and fix it yourself. If it still doesn't work after that, don't stop the run — add a line to `../../../run/issues.md` saying what failed and why, and still write the JSON file with an empty company list so the next stage has something consistent to check for.

## Outputs
- `../../../run/3a_fo.json` 
- `../../../run/issues.md` only added to if the page or table couldn't be read

## Technical details

**Fetching the page.** Same method as the market cap pages — plain requests are blocked, the site fingerprints the connection itself:
- Try `impersonate="safari184"` first.
- If that fails, retry with `impersonate="firefox133"`.

**Coverage note, not something to code against:** every company in `run/2a_mkt_cap.json` appears on this page, but this page also lists roughly 17 additional symbols (ETFs and funds) that aren't in the market cap fetch at all. Fetch everything here regardless. What to do with symbols that don't match a column is not this stage's decision

**The JSON shape example:**
```json
{
  "as_of_date": "Sep 30, 2026",
  "companies": [
    {
      "symbol": "2030", "company": "SARCO",
      "fo_actual_percent": 3.72, "fo_actual_decimal": 0.0372
    }
  ]
}
```
On total failure, still write this file, with `"as_of_date": null` and `"companies": []` — the next stage checks for exactly this to know nothing was fetched.
