# 8a_check-for-new-reports: checks if the report on site is newer

The Task: compare the dates of the reports already in the market flows workbook with the reports on saudiexchange.sa. Note which, if any, are newer.

The weekly report comes out at the end of each week and the monthly one at the end of each month. So on most days neither is newer.

## Inputs
- `../../../run/connection.md`: the market flows workbook's `itemId`
- `../../../config.yaml`: `sources.market_flows`, the page the reports are listed on

## Process

1. **Read the dates in the workbook.** In both `W.Flows.dl` (weekly) and `M.Flows.dl` (monthly), the date is in row 1, in the cell right after the one that says `Date:`. Find it by that label, not by a fixed cell. This step only reads the workbook.

2. **Read the report dates on the site.** Fetch the page. Find every card of each kind by the date at the end of its title:
   - Weekly: `Weekly Trading and Ownership By Nationality Report 01-10-2026`
   - Monthly: `Monthly Trading and Ownership By Nationality Report 30-09-2026`

   The date is day-month-year. Ignore every other kind of report. Note each card's title, date and PDF link. The titles are typed by hand, so they aren't always tidy. See "Technical details".

3. **Pick one report of each kind.** Do the weekly and the monthly separately. A report is **new** if its date on the site is later than the date in its sheet.
   - **One new report.** Pick it.
   - **More than one new report.** Pick the **oldest** new one, not the newest. The workbook then gets every week and every month in order, one per run. Add a line to `../../../run/issues.md`, for example: "Market flows: 2 weekly reports are newer than the sheet. This run records 01-10-2026. The next run records the next one."
   - **No new report.** Note the newest card. `is_new` is `false`.

4. **Write `../../../run/8a_check.json`** and move on:
   - **Either or both are new** → go to `../8b_download-reports/`. Only the new ones get worked on. The other is skipped.
   - **Neither is new** → go straight to `../8e_export-graphs/`.

5. **If something fails** (the sheet's date can't be found, the page won't load, a card can't be found), try to work out why and fix it first. If it still fails, add a line to `../../../run/issues.md` starting "Market flows:" saying what failed. Treat a report you can't check as not new. Carry on with the other.

## Outputs
- `../../../run/8a_check.json`
- `../../../run/issues.md`, added to only if something failed

## Technical details

*Required to run this step.*

**Reading row 1.** In a workbook session, `GET /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets('W.Flows.dl')/usedRange?$select=values,text`, then look in the first row for `Date:`. Ignore spaces around it: in `W.Flows.dl` it's currently typed `Date: `, with a space at the end. The date is the next cell (currently K1 holds the label and L1 the date, in both sheets). It comes back as an Excel date serial (days since 30 December 1899), shown in Excel as e.g. `1-Oct-26`: `46296` is 1 Oct 2026.

**Fetching the page.** Plain requests are blocked because the site fingerprints the connection. Use `curl_cffi`: try `impersonate="safari184"` first, then `impersonate="firefox133"`.

**Finding the cards.** All reports, over a thousand, are in the page's HTML. No clicking or filtering is needed. Each card:

```html
<li class="reportTypeMnthYear">
  <h2>Weekly Trading and Ownership By Nationality Report 01-10-2026</h2>
  ...
  <a href="/wps/wcm/connect/2a094e94-.../Weekly+Trading+and+Ownership+By+Nationality+Report+01-10-2026.pdf?MOD=AJPERES&amp;CACHEID=..." class="btn"> View Report </a>
  <a href="(the same link)" download="..."> Download </a>
</li>
```

Both buttons have the same link. Put `https://www.saudiexchange.sa` in front of it, and turn `&amp;` into `&`.

**Untidy titles.** Match the start of the `<h2>`, ignoring case and spaces ("WeeklyTrading" has been seen). Dates have been seen with one-digit parts ("28-2-2025"), spaces around the dashes, a trailing `.`, and the month first ("03-30-2023": if the middle number is over 12, it's the day). Skip a card with no date. If a title has two dates run together, take the last one.

**The JSON shape:**
```json
{
  "weekly":  { "sheet_date": "2026-09-24", "site_date": "2026-10-01", "is_new": true,
               "title": "Weekly Trading and Ownership By Nationality Report 01-10-2026",
               "url": "https://www.saudiexchange.sa/wps/wcm/connect/...pdf?MOD=AJPERES&CACHEID=..." },
  "monthly": { "sheet_date": "2026-09-30", "site_date": "2026-09-30", "is_new": false,
               "title": "Monthly Trading and Ownership By Nationality Report 30-09-2026",
               "url": "https://www.saudiexchange.sa/wps/wcm/connect/...pdf?MOD=AJPERES&CACHEID=..." }
}
```

Dates are written year-month-day. If a date couldn't be read, it's `null` and `is_new` is `false`. `site_date`, `title` and `url` belong to the report picked in step 3.
