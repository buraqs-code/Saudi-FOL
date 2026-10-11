# 07_export_pdf: turn the report into PDFs

The Task: make two PDFs.

- **The report** — `EMAIL.1` and `EMAIL.CHARTS` together. This gets attached to the email.
- **The email body** — `Email.0` and `EMAIL.2` together. Its pages become the pictures in the body of the email.

The work is done on throwaway copies of the workbook, never on the workbook itself. Each PDF gets its own fresh copy. Don't make two PDFs from the same copy: OneDrive keeps handing back the first PDF it made from a file, even after the file changes.

## Inputs
- `../../run/connection.md`

## Process

If `../../run/issues.md` has a line that starts with "No report:", do not do this stage. Go to `stages/08_send-report/`.

1. **Note the report date.** Read the date cell in `EMAIL.1` of the real workbook that every figure works from (currently E3). Find it the way stage 06 does: take a figure in the `Mkt Cap` column and follow its date reference, through row 1, to the cell holding a plain date. This is read only. Step 3 uses it.

   Stage 06 has only just changed this cell, so a read straight after can still show the old date. If it shows a date older than the last date in `Mkt_Cap`, wait 30 seconds and read again.

2. **Clear out any old copy.** Look in the workbook's folder for a file named `FOL_pdf_export_temp.xlsx`. If one is there, a past run didn't finish cleaning up. Delete it, and add a line to `../../run/issues.md` saying an old copy was found and deleted. If it says the file is locked, wait and try again (see "Technical details").

3. **Make the report PDF**, following "Making one PDF" below with:
   - sheets to keep visible: `EMAIL.1` and `EMAIL.CHARTS`
   - save as: `../../run/7_report.pdf`

   The sheets come out in tab order, so `EMAIL.1` comes first and the charts follow.

4. **Make the email body PDF**, following "Making one PDF" again, with a new copy:
   - sheets to keep visible: `Email.0` and `EMAIL.2`
   - save as: `../../run/7_body.pdf`

   `Email.0` comes first because its tab sits before `EMAIL.2`'s. If someone ever moves the tabs the other way round, the two pictures in the email swap places too.

### Making one PDF

a. **Make the copy.** Copy the workbook into the same folder, named `FOL_pdf_export_temp.xlsx`. Wait until it's done, then note the copy's `id`. Before changing anything, check the `id` you're using is the copy's and not the one in `connection.md`.

b. **Check the copy is up to date.** Read the date cell in the copy. It must show the date from step 1. A new copy often isn't ready to open for 20–30 seconds, so keep trying for about two minutes. If it shows an older date, delete the copy, wait a minute and go back to (a). Do this once. If it still doesn't match, go to "If something fails".

c. **Hide every other sheet.** Open a session on the copy, hide every sheet except the two to keep, check those two are the only ones still visible, then close the session. Closing saves the change.

d. **Ask for the copy as a PDF** and save it.

e. **Check the PDF before deleting the copy.** It must be a real PDF with at least one page, and show the right sheets:
   - **Report PDF:** has text from `EMAIL.1` (e.g. "SAR Mn"), and no text from `EMAIL.2` (e.g. "TOP 1-DAY FOREIGN BUYING"). For `EMAIL.CHARTS`, look at the last page: it must show the charts, drawn, not blank boxes. (Text alone can't prove the charts are there: the chart titles are company names, which also appear on `EMAIL.1`.)
   - **Body PDF:** has text from `Email.0` (e.g. "TASI - EX ARAMCO") and from `EMAIL.2`, and no text found only on `EMAIL.1` (e.g. "SAR Mn") or `EMAIL.CHARTS`.

   If it's wrong, wait 30 seconds and ask for the PDF again, up to 3 times. If it's still wrong, go to "If something fails". Note how many pages it has.

f. **Delete the copy.** Do this every time, even if a step above failed. Close any open session on the copy first. If it says the file is locked, wait and try again (see "Technical details").

### If something fails

If either PDF can't be made, the whole run stops. Add a line to `../../run/issues.md` saying which step failed and what came back, ending with "the run stopped". Still delete the copy (step f). Then go on to `stages/08_send-report/`, so the issue still gets sent.

## Outputs
- `../../run/7_report.pdf` — the report, `EMAIL.1` then `EMAIL.CHARTS`
- `../../run/7_body.pdf` — `Email.0` then `EMAIL.2`, for the pictures in the email body
- `../../run/issues.md` — added to if something fails, or if an old copy was found and deleted
- The real workbook is not changed. No copy is left behind in the folder.

## Technical details

**The copy's name must not contain the words in `workbook.filename_contains`** in the config. Otherwise stage 01 could mistake a leftover copy for the workbook. `FOL_pdf_export_temp.xlsx` is fine as long as that setting stays as it is.

**Finding the folder** — the copy goes in the same folder as the workbook. Get that folder's `id` from the workbook's details:

```
GET /v1.0/drives/{driveId}/items/{itemId}?$select=parentReference
```

**Looking for an old copy** — `GET /v1.0/drives/{driveId}/items/{folderId}/children?$select=id,name`, then delete any with the name `FOL_pdf_export_temp.xlsx`.

**Making the copy:**

```
POST /v1.0/drives/{driveId}/items/{itemId}/copy
{
  "parentReference": { "driveId": "{driveId}", "id": "{folderId}" },
  "name": "FOL_pdf_export_temp.xlsx"
}
```

The reply is `202` with a `Location` header. That's a link for checking progress. Ask it every few seconds, **without** the `Authorization` header, until `status` is `completed`. The copy's `id` is the `resourceId` in that reply.

**Reading the date cell** — once found, read it, e.g. `GET .../items/{id}/workbook/worksheets('EMAIL.1')/range(address='E3')?$select=values`. On a fresh copy this may fail with a `404` or `503` at first. That's the copy still getting ready, not a real failure.

**Hiding a sheet** — inside a session (`createSession`, `persistChanges: true`):

```
PATCH .../items/{copyId}/workbook/worksheets('Mkt_Cap')
{ "visibility": "Hidden" }
```

Get the list of sheets from `GET .../workbook/worksheets?$select=name,visibility` on the copy, rather than from `connection.md`, in case a sheet was added. Always name sheets in the address like `worksheets('Mkt_Cap')`, not by their `id`. The `id`s change in a copy. Leave any sheet already `Hidden` or `VeryHidden` as it is.

`closeSession` sends no body, but some HTTP clients need `Content-Length: 0` set explicitly, or it fails with `411 Length Required` instead of `204`.

**Making the PDF:**

```
GET /v1.0/drives/{driveId}/items/{copyId}/content?format=pdf
```

The reply is a `302` redirect to the PDF itself. Follow it to download, without the `Authorization` header. Asked for straight after closing a session, it can come back `406` at first. Wait a few seconds and ask again. Both PDFs are made with this same request, each on its own copy.

**This request is slow.** `EMAIL.CHARTS` has about 76 charts, so the report PDF can take over a minute. Allow up to 4 minutes per request. A timeout is normal. Ask again, don't treat it as a failure.

**Always close the session** — even if a step fails. A session left open locks the copy.

**A locked copy** — an open session makes the copy locked. Deleting it gives `423 resourceLocked`, and making a new copy then gives `409 nameAlreadyExists`. The lock clears by itself after about 2–3 minutes. Try the delete again every 15–20 seconds until it works.

**Sheet names** — `Email.0` is currently written with a lowercase "mail", unlike `EMAIL.1`, `EMAIL.2` and `EMAIL.CHARTS`. Use each name exactly as the sheet list gives it.

**Checking it's a PDF** — the file starts with `%PDF`. Count the pages with `pypdfium2`, the same library stage 08 uses. Install it if it isn't there.

**Deleting the copy** — `DELETE /v1.0/drives/{driveId}/items/{copyId}`. It goes to the OneDrive recycle bin, which is fine.

Requests can time out. Try again rather than treating the first failure as final.