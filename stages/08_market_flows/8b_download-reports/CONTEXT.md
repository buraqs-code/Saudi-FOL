# 8b_download-reports: download the new reports

The Task: download the PDF of each report that step 8a marked as new. Skip the others.

## Inputs
- `../../../run/8a_check.json`: which reports are new, and their PDF links

## Process

1. **For each report with `is_new: true`**, download the PDF from its `url`. Check that what comes back really is a PDF. A blocked download often comes back as a web page instead.

2. **Save it:**
   - Weekly → `../../../run/8b_weekly.pdf`
   - Monthly → `../../../run/8b_monthly.pdf`

3. **If a download fails,** try to work out why and fix it first. If it still fails, add a line to `../../../run/issues.md` starting "Market flows:", e.g. "Market flows: the weekly report couldn't be downloaded (blocked: 403 Access Denied)." Carry on with the other report if there is one.

4. **Move on:**
   - Weekly PDF saved → `../8c_weekly/`. It goes on to the monthly afterwards if needed.
   - Only the monthly PDF saved → `../8d_monthly/`
   - Nothing saved → `../8e_export-graphs/`

## Outputs
- `../../../run/8b_weekly.pdf` and/or `../../../run/8b_monthly.pdf`, only for reports that were new and downloaded
- `../../../run/issues.md`, added to only if a download failed

## Technical details

*Required to run this step.*

**Downloading.** Use `curl_cffi`, the same as the page fetch in 8a: try `impersonate="safari184"` first, then `impersonate="firefox133"`. A `403` reply means the site blocked this connection.

**Checking it's a PDF.** The reply's content type is `application/pdf`, and the file starts with `%PDF`. The weekly report is currently about 10 pages and 1.8 MB.

**What's inside.** The tables in these PDFs are pictures, not text. The next step will have to read them from the page images. That's not this step's concern. It's noted here so nobody is surprised when the file has almost no text in it.
