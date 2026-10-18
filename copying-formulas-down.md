# Process: Copy the Previous Row's Formulas Down to a New Row (Microsoft Graph)

**Base URL**

```
https://graph.microsoft.com/v1.0/{workbook}/workbook/worksheets('{sheet}')
```

Pick `{workbook}` by where the file lives. `{path}` is relative to the drive's root, so include folders, e.g. `Reports/2026/flows.xlsx`.

| File location | `{workbook}` |
|---|---|
| Your own OneDrive | `me/drive/root:/{path}:` |
| Another user's OneDrive (app-only token) | `users/{user-id or email}/drive/root:/{path}:` |
| SharePoint site (default library) | `sites/{site-id}/drive/root:/{path}:` |
| Any drive, by ID (incl. other SharePoint libraries) | `drives/{drive-id}/root:/{path}:` |
| File shared with you, or path unknown | `drives/{drive-id}/items/{item-id}` |

- App-only tokens have no signed-in user, so `me/...` doesn't work with them.
- Addressing by item ID is the most robust option, because it keeps working if the file is renamed or moved.

Replace `{sheet}` and the column span (`A:Q` in the examples) with the target workbook's values.

## Steps

1. **Find the last row.**
   `GET {base}/usedRange(valuesOnly=true)?$select=address`
   The end of the address gives the last row **N** and the column span. Confirm row N is real data, not a totals row, notes, or pre-filled blank formulas.

2. **Check the target row is empty.**
   `GET {base}/range(address='A{N+1}:Q{N+1}')?$select=formulas`
   If any cell has content, stop.

3. **Check the source data exists.**
   If the formulas read from another sheet, confirm that sheet's row N+1 has the new period's data. If it doesn't, stop.

4. **Read the last row's pattern.**
   `GET {base}/range(address='A{N}:Q{N}')?$select=formulasR1C1,numberFormat,valueTypes`
   - If any `valueTypes` entry is `"Error"`, stop.
   - If any `formulasR1C1` entry doesn't start with `=`, it's a typed value (e.g. a date). Replace it with the new value; write dates as Excel serial numbers. Don't copy the old value.

5. **Write the new row.**
   `PATCH {base}/range(address='A{N+1}:Q{N+1}')`

   ```json
   {
     "formulasR1C1": [[ ... ]],
     "numberFormat": [[ ... ]]
   }
   ```

   Use the arrays from step 4, unchanged except for the replaced typed values. Never write A1-style formulas or edit the R1C1 strings.

6. **Confirm the write.**
   The response must be `200` and its `address` must end in row N+1. If it shows row N, the PATCH didn't run.

7. **Verify.**
   `GET {base}/range(address='A{N+1}:Q{N+1}')?$select=formulasR1C1,valueTypes,text`
   - `formulasR1C1` must match what was sent.
   - No `valueTypes` entry may be `"Error"`.
   - `text` must show real values, not blanks or dashes.
   - If any check fails, clear the row and report it.

8. **Report** the new row number and its date in one line.

## Don't Use This Process When

- the data is an Excel table (use `rows/add` instead);
- the row has spilling dynamic array formulas;
- the sheet keeps only the newest row as live formulas. In that case, convert the previous row to values before the source data is overwritten.
