# 8e_export-graphs: get the market flows charts as pictures

The Task: get each chart on the `M.Email` and `W.Email` sheets of the market flows workbook as a picture. Stage 09 puts them in the email.

If the pictures cannot be made, use the fallback: make one PDF of the two sheets. The fallback is the older method. It works, but its pictures are less sharp.

## When this runs

**On every run that reaches stage 08.** It does not matter if the weekly or monthly report was new, or if 8c or 8d failed. The charts always show the latest figures in the workbook. When nothing is new, the charts are the same as last time. The email still needs them.

This step does not change the charts or the workbook. They follow the data by themselves.

## Inputs
- `../../../run/connection.md`: the `driveId` and the market flows workbook's `itemId`

## Process

1. **Remove old output.** If `../../../run/8e_charts/` or `../../../run/8e_graphs.pdf` is there, delete it. Stage 09 must only find this run's charts.

2. **List the charts.** Do `M.Email` first, then `W.Email`. The email shows the monthly charts first. On each sheet, put the charts in order from top to bottom. If two charts are at the same height, put the left one first. Note each chart's title.

3. **Get each chart as a picture.** Get them in the order from step 2. Save them in `../../../run/8e_charts/` as `01.png`, `02.png`, and so on.

4. **Check each picture.** Open it and look at it. It must be a real image. The chart must be drawn, not blank, and not cut off. The title must be the title from step 2.

5. **If every picture passes,** you are done. Go to "Next".

   If a picture fails, get it again, up to 3 times. If it still fails, delete `../../../run/8e_charts/`. Add a line to `../../../run/issues.md` that starts with "Market flows:" and says which chart failed and what came back. Say that the PDF fallback was used. Then do the fallback. Do not mix pictures from the two methods.

## Fallback: make a PDF

Do this only if step 5 sends you here.

The work is done on a throwaway copy of the market flows workbook, never on the workbook itself. Hiding sheets in the real workbook would change it for everyone who opens it.

1. **Note the latest dates.** In the real market flows workbook, read the last date in column A of `W.Email` and of `M.Email`. This is read only. Fallback step 4 uses these dates.

2. **Clear out any old copy.** Look in the workbook's folder for a file named `FOL_mf_export_temp.xlsx`. If one is there, an earlier run did not delete it. Delete it now. Add a line to `../../../run/issues.md` that starts with "Market flows:" and says that an old copy was found and deleted.

3. **Make the copy.** Copy the market flows workbook into the same folder, with the name `FOL_mf_export_temp.xlsx`. Wait until the copy is done. Note the copy's `id`. Make sure that this `id` is not the one in `connection.md`.

4. **Check that the copy is up to date.** In the copy, read the last date in column A of `W.Email` and `M.Email`. Both must be the dates from fallback step 1. A new copy can take 20 to 30 seconds before it opens, so try again for about two minutes. If a date is older, delete the copy, wait one minute and go back to fallback step 3. Do this one time only. If the date is still older, go to "If something fails".

5. **Hide all the other sheets, and put the monthly sheet first.** Open a session on the copy. Hide every sheet except `W.Email` and `M.Email`. The `M.Email` tab must come before `W.Email`. If it does not, move `M.Email` in front of `W.Email` in the copy. Then close the session. Closing saves the change.

6. **Get the copy as a PDF.** Save it as `../../../run/8e_graphs.pdf`. The sheets come out in tab order, so `M.Email` comes first. The print area and page breaks on each sheet set how the charts are split across pages.

7. **Check the PDF.** It must be a real PDF with at least one page. Look at each page as an image. Each page must show charts that are drawn, not empty boxes. If the PDF is wrong, wait 30 seconds and get it again, up to 3 times.

8. **Delete the copy.** Do this every time, also when an earlier step failed. Close any open session on the copy first.

## If something fails

This step never stops the run. If the fallback also fails, add a line to `../../../run/issues.md` that starts with "Market flows:" and says which step failed and what came back. Do not end it with "the run stopped". Do not save a PDF that failed the check in fallback step 7. Still delete the copy (fallback step 8).

The email still goes out, without the market flows charts.

## Outputs
- `../../../run/8e_charts/`: one picture for each chart, `M.Email` first, then `W.Email`, top to bottom
- or, only if the fallback was used, `../../../run/8e_graphs.pdf`: the `M.Email` pages, then the `W.Email` pages
- `../../../run/issues.md`, added to only if the fallback was used, something failed, or an old copy was found
- The real workbook is not changed. No copy stays in the folder.

## Next

Go to `stages/09_send-report/`.

## Technical details

*Required to run this step.*

### The pictures

**Listing the charts.** `GET /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets('M.Email')/charts?$select=name,top,left`. Sort by `top`, then by `left`. Read each title with `GET .../charts('{name}')/title`. The title is in `text`.

**Getting a picture.** `GET .../worksheets('M.Email')/charts('{name}')/image(width=1400)`. Give only the width. The height then follows the chart's shape. The reply is JSON: `{ "value": "<the PNG, base64-encoded>" }`. Decode it and save it. 1400 is twice the width the email shows, so the picture stays sharp. Each picture is about 30 to 85 KB.

Write a space in a chart name as `%20`, for example `charts('Chart%202')`. No session is needed. This only reads.

Microsoft's documentation says this request does not work with an app-only token. It does work with this project's token. It was tested on all the charts in October 2026. If it starts to fail with `401` or `403` on every chart, that may be the reason. The fallback then keeps the email going.

**Checking a picture.** The file starts with the bytes `\x89PNG`. A failure can also come back as a `500` or `504`, or a timeout. Try again.

### The fallback

**The name of the copy.** It must not contain the words in `workbook.filename_contains` or `market_flows_workbook.filename_contains` in `config.yaml`. If it did, stage 01 could take a copy that was not deleted for a real workbook. `FOL_mf_export_temp.xlsx` is correct while those settings stay as they are. It is also different from stage 07's copy, so the two copies cannot lock each other.

**Finding the folder.** The copy goes in the same folder as the workbook. Get the folder's `id` from the workbook's details: `GET /v1.0/drives/{driveId}/items/{itemId}?$select=parentReference`.

**Looking for an old copy.** `GET /v1.0/drives/{driveId}/items/{folderId}/children?$select=id,name`. Delete any item with the name `FOL_mf_export_temp.xlsx`.

**Making the copy.**

```
POST /v1.0/drives/{driveId}/items/{itemId}/copy
{
  "parentReference": { "driveId": "{driveId}", "id": "{folderId}" },
  "name": "FOL_mf_export_temp.xlsx"
}
```

The reply is `202` with a `Location` header. That is a link that shows the progress. Ask it every few seconds, **without** the `Authorization` header, until `status` is `completed`. The copy's `id` is the `resourceId` in that reply.

**Reading the dates.** `GET .../items/{id}/workbook/worksheets('W.Email')/usedRange?$select=values`. The dates are the numbers in column A, from row 7 down. On a new copy this can fail with `404` or `503` at first. That is the copy getting ready, not a real failure.

**Hiding a sheet.** Inside a session (`createSession`, `persistChanges: true`): `PATCH .../items/{copyId}/workbook/worksheets('W.Flows.dl')` with the body `{ "visibility": "Hidden" }`. Get the list of sheets from `GET .../workbook/worksheets?$select=name,visibility` on the copy. Always name a sheet in the address, like `worksheets('W.Flows.dl')`, not by its `id`. The `id`s change in a copy. Leave a sheet that is already `Hidden` or `VeryHidden` as it is.

**Moving a tab** (only if the order is wrong). In the same session: `PATCH .../items/{copyId}/workbook/worksheets('M.Email')` with the body `{ "position": <W.Email's position> }`. Read the positions from the sheet list (`?$select=name,position,visibility`). This changes only the copy.

`closeSession` sends no body. Some HTTP clients must set `Content-Length: 0`. If not, the reply is `411 Length Required` and not `204`.

**Making the PDF.** `GET /v1.0/drives/{driveId}/items/{copyId}/content?format=pdf`. The reply is a `302` redirect to the PDF. Follow it to download, without the `Authorization` header. Straight after the session closes, it can come back `406`. Wait a few seconds and ask again. The request can be slow. Allow up to 4 minutes. A timeout is normal. Ask again.

**Checking the PDF.** The file starts with `%PDF`. Count the pages and turn them into images with `pypdfium2`.

**Always close the session**, also when a step fails. A session that stays open locks the copy.

**A locked copy.** Deleting a locked copy gives `423 resourceLocked`. Making a new copy then gives `409 nameAlreadyExists`. The lock clears by itself after about 2 to 3 minutes. Try the delete again every 15 to 20 seconds until it works.

**Deleting the copy.** `DELETE /v1.0/drives/{driveId}/items/{copyId}`. It goes to the OneDrive recycle bin. That is correct.

Requests can time out. Try again. Do not take the first failure as final.
