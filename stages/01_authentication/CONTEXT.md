# 01_authentication: verify access and identify the workbook

The Task: verify the automation can access the workbook, find out exactly which file it is, and record both so later stages don't repeat the work. Do the same for the market flows workbook. This stage only reads and never changes either workbook.

## Inputs
- `../../config.yaml`
- Three settings available in your environment: `GRAPH_TENANT_ID`, `GRAPH_CLIENT_ID` and `GRAPH_CLIENT_SECRET`. Read them from there.

## Process

1. **Get a token for Microsoft's Graph API**. Every request in this run has to carry one.

   Get it once at the start of the run and reuse it for every request after. It's good for about an hour.

2. **Find the storage area the workbook lives in.** The file sits in the OneDrive of the person named as `workbook.owner_email` in the config. Look up that person's drive and note down its `id`. That's the `driveId` every later stage needs.

3. **Find the file itself.** It's in the folder the config gives as `workbook.folder`, inside that drive. List what's in that folder.

   Identify the file by the words the config gives as `workbook.filename_contains` appearing anywhere in the name.

   - **One Excel file.** That's the one. Note down its `id` (the `itemId`) and its name.
   - **Several Excel files, only one whose name contains those words.** Use that one and ignore the rest. An unrelated file / folder sitting in the folder isn't a problem.
   - **Several Excel files whose names all look like the workbook.** Stop. Instead of the usual file details, write down what you found in `../../run/connection.md`: each file's name, when it was last changed, and its size. Then add a line to `../../run/issues.md` saying what was found and that the run stopped.
   - **No Excel file at all.** Stop, same as above. Write a note that the folder held no Excel file into `../../run/connection.md`, and add a line to `../../run/issues.md` saying so and that the run stopped.

   **If this stage stops for either reason above:** don't continue on to stage 02 or 03. Skip straight to `stages/09_send-report/`, so the issue that was just logged to `../../run/issues.md` still gets sent out.

4. **Check it actually works.** Before anything else depends on these codes, use them to read the list of sheet names in the workbook. Note down what comes back. This is read only.

5. **Find the market flows workbook too.** It's a second Excel file in the same folder, used only by stage 08. Use the folder listing from step 3. Look for the words the config gives as `market_flows_workbook.filename_contains`. For example, `New_Saudi Market Flows.xlsx` matches "Saudi Market Flows".

   - **One file matches.** Note down its `id` (its own `itemId`) and its name. Then read its list of sheet names, the same way as step 4.
   - **No file matches, or more than one does.** Don't stop the run. The daily report doesn't depend on this file. Note what was found in `../../run/connection.md`, and add a line to `../../run/issues.md` starting "Market flows:" that says so. Stage 08 will see that it has no workbook and skip itself.

   The two workbooks are told apart by name, so the market flows file never counts as a second copy of the main workbook in step 3.

## Outputs
- `../../run/connection.md`: how it signed in, the `driveId`, and for each workbook its `itemId`, name and the sheet names seen when checking access, each under its own heading (`Workbook` and `Market flows workbook`). If a file couldn't be identified, what was found goes under its heading instead.
- `../../run/issues.md`: added to when the main workbook couldn't be identified and the run stopped, or when the market flows workbook couldn't be found (the run carries on).

## Technical details

**Getting the token** — `POST https://login.microsoftonline.com/${GRAPH_TENANT_ID}/oauth2/v2.0/token`, with a form body of:

- `client_id=${GRAPH_CLIENT_ID}`
- `client_secret=${GRAPH_CLIENT_SECRET}`
- `scope=https://graph.microsoft.com/.default`
- `grant_type=client_credentials`

The `access_token` in the reply is the token. Send it on every later request as the header `Authorization: Bearer ${TOKEN}`.

**Finding the drive** — `GET /v1.0/users/{workbook.owner_email}/drive`

**Listing the folder** — `GET /v1.0/drives/{driveId}/root:/{workbook.folder}:/children`. Don't add `Documents/` to the path — it's not needed here. Write a space in the folder path as `%20`.

**Checking access** — `GET /v1.0/drives/{driveId}/items/{itemId}/workbook/worksheets`. For the market flows workbook, use its own `itemId`. The `driveId` is the same for both.
