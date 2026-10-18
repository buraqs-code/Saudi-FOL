# 09_send-report: send the emails

The Task: at the end of every run, send the right emails.

- **A good run** sends two emails: the report, to the people who read it, and then a short "Run Succeeded" email, to whoever looks after the automation.
- **A stopped run** sends only a "Run Stopped" email, to whoever looks after the automation. The report doesn't go out.
- **A no-report run** sends only a "No report" email, to whoever looks after the automation. This occurs on Friday and Saturday, and when there is no new trading day. Stage 06 decides this.

Every run ends here, including one that stopped early.

## Inputs
- `../../config.yaml` — the `email` section: who sends, and who gets which email
- `../../run/connection.md` — to read the report date from the workbook
- `../../run/issues.md` — may not exist. If it doesn't, there were no issues.
- `../../run/7_table.pdf`, `../../run/7_charts.pdf`, `../../run/7_body.pdf` — only there on a good run
- `../../run/8e_charts/` — the market flows charts, one picture each. Or `../../run/8e_graphs.pdf`, if step 8e used its fallback. Only there if stage 08 ran and its step 8e worked

## Process

**Which date to use.** Every email on a good run uses the **report date**: the day the report covers, not the day the email is sent. Stage 06 put it in `EMAIL.1`, in the one date cell every figure works from (currently E3). Find it the way stage 06 does: take a figure in the `Mkt Cap` column and follow its date reference, through row 1, to the cell holding a plain date. The examples below use 30 Sep 2026, sent on 1 Oct 2026.

The "Run Stopped" email is the exception. The run may have stopped before the report date was known, so it uses **today's date**, in Pakistan time.

1. **Did the run stop?** Read `issues.md`. If any line says the run stopped, it stopped. Go to step 5.

   **Is it a no-report run?** If a line starts with "No report:", go to step 7.

   A line that starts with "Market closed:" does not stop the report. It only tells you that a row was copied.

   If no line says that but `7_table.pdf`, `7_charts.pdf` or `7_body.pdf` is missing, the report can't be sent. Treat it as stopped, with "the report PDFs weren't made" as the issue.

2. **Read the report date** from `EMAIL.1` (read only).

3. **Send the report email** to `email.report_recipients`.

   - **Subject:** `Saudi Foreign Ownership - 30 Sep 2026`
   - **Body:** a greeting and one line, then two pictures, `Email.0` first and `EMAIL.2` second, then the market flows charts, then the sign-off:

     > Dear Team, 
     > Please find the top foreign buying and selling on the Saudi Exchange for 30 Sep 2026. The full report is attached.
     >
     > *[picture of Email.0]*
     >
     > *[picture of EMAIL.2]*
     >
     > *[pictures of the market flows charts]*
     >
     > Regards,
     > Buraq

   - **If `issues.md` has a line starting "FO page behind:"**, add one more line to the body, right after "The full report is attached.":

     > Note: the foreign ownership figures for 30 Sep 2026 weren't available, so the figures from 29 Sep 2026 are used. The 1-day buying and selling tables are empty for this reason.

     The first date is the report date. The second is the last day the FO figures were updated. Take it from the "FO page behind:" line. This tells readers the FO figures are old.

     On these days the 1-day tables in the `EMAIL.2` picture show filler rows (e.g. "TASI" with `#N/A`, or one company repeated with "-"), because no company's FO changed. That's expected. Don't treat it as a wrong picture in the check below.

   - **Attached:** two files.
     - `7_table.pdf` (`EMAIL.1`), renamed to `Saudi FO-30 Sep 2026.pdf`
     - `7_charts.pdf` (`EMAIL.CHARTS`), renamed to `Saudi-FO-Charts-30 Sep 2026.pdf`

   The pictures are made from `7_body.pdf`: each page becomes one image, in the PDF's order, which puts `Email.0` first. Trim the white margins around each.

   Both pictures are shown **at the same width** in the email, so the two tables line up edge to edge and the second one isn't smaller than the first.

   **Look at each picture before sending.** Open each image and check:
   - It shows the right table: the first has the TASI summary rows, the second has the top foreign buying and selling tables.
   - Nothing is cut off. All four edges of the table, with their borders, are fully in the picture.
   - There's no wide strip of white left on any side, only a thin gap around the table.
   - It isn't blank or mostly empty.

   If a picture fails, redo the cutting for that page and look again. If it still fails after one more try:
   - **Part of the table is missing, or the picture is blank:** don't send the report. Add a line to `issues.md` saying which picture was wrong and how, ending with "the run stopped". Then go to step 5.
   - **Only the spacing is off** (the table is all there but has extra white around it): send the report anyway. Add a line to `issues.md` saying which picture had extra white around it.

   **The market flows charts.** Put them after the `EMAIL.2` picture, with no heading, in order. The monthly charts come first, then the weekly charts.
   - **If `8e_charts/` is there,** use its pictures as they are, in file order (`01.png`, `02.png`, …). Do not cut them.
   - **If only `8e_graphs.pdf` is there,** make one picture from each of its pages, in order, the same way as `7_body.pdf`.

   Look at each one: the charts must be drawn, nothing cut off, and not blank.

   These pictures never stop the report. If one still fails after one more try, leave out **all** the market flows pictures, not only the bad one. Add a line to `issues.md` that starts with "Market flows:" and says which picture was wrong. Send the report without them.

   If neither is there, send the report without the charts. Stage 08 has already logged the reason, or it did not run.

4. **If the report email fails to send,** the run counts as stopped after all. Add a line to `issues.md` saying the report email couldn't be sent and what came back, ending with "the run stopped". Then go to step 5.

   If it sent, go to step 6.

5. **Stopped: send the "Run Stopped" email** to `email.status_recipients`. Then you're done.

   - **Subject:** `FOL: Run Stopped - 1 Oct 2026`
   - **Body:** one line, then the issues table:

     > FOL stopped on 1 Oct 2026. The report was not sent.

     If the run stopped only because the report email failed (step 4), the sheets were already updated. Say so instead, using the report date:

     > The sheets were updated for 30 Sep 2026, but the report email couldn't be sent.

6. **Succeeded: send the "Run Succeeded" email** to `email.status_recipients`.

   - **No issues:**
     - Subject: `FOL: Run Succeeded - 30 Sep 2026`
     - Body: `FOL ran successfully for 30 Sep 2026.`
   - **Some issues:**
     - Subject: `FOL: Run Succeeded, 2 issues to review - 30 Sep 2026` (or `1 issue`)
     - Body: the same line, then the issues table.
   - **Market flows updated:** if `issues.md` has lines that start with "Market flows updated:", put each one in the body, under the first line, as written. For example: "Market flows updated: the weekly charts now go to 8 Oct 2026." These lines are not issues. On a day with no new report, there is no such line, and the email says nothing about market flows.

7. **No report: send the "No report" email** to `email.status_recipients`. Then stop.

   - **Subject:** `FOL: No report - 10 Oct 2026`. Use today's date in Pakistan time.
   - The examples below are for runs on Friday 9 Oct and Saturday 10 Oct. The date after "Sheets updated for" is the date of the row that this run added.
   - **Body, weekend run** (the "No report:" line starts with "No report: weekend run"):

     > Weekend run. Sheets updated for 9 Oct 2026. Report not sent. The next report goes out on Sunday.

   - **Body, no new trading day** (the "No report:" line starts with "No report: no new trading day"):

     > No new trading day. The FO page shows 8 Oct 2026. This day was in the last report. Sheets updated for 9 Oct 2026. Report not sent. Possible causes: a market holiday, or the FO page did not update.

   - If `issues.md` has other lines, add the issues table below.

### The issues table

Two columns, **Issue** and **Likely fix**, one row for each line in `issues.md`.

Do not include lines that start with "No report:", "Market closed:" or "Market flows updated:". These lines are normal. Do not count them as issues in the "Run Succeeded" subject. Lines that start with "Market flows:" are issues. Include them.

- **Issue** — what happened, in plain words. Name the company, sheet or date involved.
- **Likely fix** — what would most likely sort it out, in one or two lines. Use what you know about this project to make it specific, but write it for someone who doesn't code. Write just the fix

Example:

| Issue | Likely fix |
|---|---|
| A new company (code 4325) appeared on the website but has no column in `FO_Actual`, so its data wasn't recorded today. | Add a column for code 4325 in `FO_Actual` and the other company sheets. |

If a line is just for information and nothing needs doing, say so in the fix column, e.g. "Nothing to fix."

Don't suggest re-running to replace copied figures. A re-run never overwrites values already in the sheet, so it changes nothing. For copied figures, say what to check instead, e.g. "Check the website is reachable", or "Nothing to fix" if the copy was expected.

### For every email

- Short, plain words, straight to the point.
- Dates written like `30 Sep 2026`.
- The status emails have no sign-off.

## Outputs
- The emails, sent
- `../../run/issues.md` — added to only if the report email fails to send

## Technical details

**Reading the report date** — once found, read it, e.g. `GET /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets('EMAIL.1')/range(address='E3')?$select=values,text`. The value is an Excel date serial number (days since 30 December 1899), e.g. `46295` is 30 Sep 2026.

**Sending an email** — every email is sent from the `email.sender` mailbox:

```
POST /v1.0/users/{email.sender}/sendMail
{
  "message": {
    "subject": "Saudi Foreign Ownership - 30 Sep 2026",
    "body": { "contentType": "HTML", "content": "..." },
    "toRecipients": [ { "emailAddress": { "address": "someone@example.com" } } ],
    "attachments": [ ... ]
  },
  "saveToSentItems": true
}
```

A reply of `202` means it was accepted for sending. Anything else means it failed.

**The pictures in the body** — each goes in as an attachment marked as inline, and the body points to it by its `contentId`:

```
{
  "@odata.type": "#microsoft.graph.fileAttachment",
  "name": "body_1.png",
  "contentType": "image/png",
  "contentBytes": "<the image, base64-encoded>",
  "isInline": true,
  "contentId": "body_1"
}
```

and in the body: `<img src="cid:body_1" width="700">`. Number them by page, `body_1`, `body_2` and so on, and put them in the body in that order. The market flows pictures carry on the numbering after the `7_body.pdf` pictures, e.g. `body_3`, `body_4`.

**The `8e_charts/` pictures** are PNGs, about 1400 pixels wide. Attach each one as it is, with `"contentType": "image/png"`. Show it at `width="700"`, the same as the other pictures.

**Turning the PDF into a picture** — render each page with a PDF library, e.g. `pypdfium2`, at about twice normal size (`scale=2`) so it stays sharp. Then cut off the white margins.

**Picture width** — give every `<img>` the same fixed `width="700"`, no matter how wide the image itself is. Don't size each one from its own pixel width, such as "half the real width". The pages trim to different widths, so the pictures would come out at different sizes in the email. Set only `width`, not `height`, so each picture keeps its shape.

**The PDF attachments** — each the same as the picture, but `"contentType": "application/pdf"`, `"isInline": false`, and no `contentId`.

**The issues table** — a plain HTML `<table>`, with borders and padding written on each cell (`style="border:1px solid #ccc; padding:6px"`). Outlook ignores most other styling, but this much it keeps.

**Size** — one `sendMail` request can be about 4 MB at most, attachments included. Base64 makes files about a third bigger. The two attached PDFs, the two pictures and the market flows pictures should still be within it. If a send fails because it is too large, send it again without the market flows pictures, and log a "Market flows:" line saying so. If a send fails because it's too large, log that as the issue.

**Today's date** — use Pakistan time (PKT), not the server's clock, which may be on a different day. PKT is UTC+5 all year, with no daylight saving, so take the time in UTC and add 5 hours. Don't rely on the server's time-zone settings to do this.

Requests can time out. Try again rather than treating the first failure as final. But a `sendMail` that timed out may still have gone through, so check the sender's Sent Items for that subject before sending it again.


