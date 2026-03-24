# Google Sheets Order Setup

Follow these steps once to connect the landing page forms to Google Sheets.

## 1) Create the target sheet

1. Open [Google Sheets](https://sheets.google.com) and create a new spreadsheet.
2. Rename the first tab to: `Orders`
3. Put this header row in row 1:

`submitted_at | product_name | price_mad | name | phone | address | form_id | page_url | user_agent`

## 2) Create Apps Script webhook

1. In the sheet, click `Extensions` -> `Apps Script`.
2. Delete default code and paste this:

```javascript
const SHEET_NAME = "Orders";

function doPost(e) {
  try {
    const payload = JSON.parse(e.postData.contents || "{}");
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
    if (!sheet) {
      return jsonResponse({ status: "error", message: "Sheet not found" }, 500);
    }

    sheet.appendRow([
      payload.submitted_at || new Date().toISOString(),
      payload.product_name || "",
      payload.price_mad || "",
      payload.name || "",
      payload.phone || "",
      payload.address || "",
      payload.form_id || "",
      payload.page_url || "",
      payload.user_agent || ""
    ]);

    return jsonResponse({ status: "success" }, 200);
  } catch (err) {
    return jsonResponse({ status: "error", message: String(err) }, 500);
  }
}

function doOptions() {
  return jsonResponse({ status: "ok" }, 200);
}

function jsonResponse(obj, statusCode) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## 3) Deploy as Web App

1. Click `Deploy` -> `New deployment`.
2. Type: `Web app`.
3. Execute as: `Me`.
4. Who has access: `Anyone`.
5. Click `Deploy`.
6. Copy the Web App URL.

## 4) Add URL to landing page

1. Open `index.html`.
2. Find:

```javascript
var SHEETS_WEB_APP_URL = "";
```

3. Replace with your URL:

```javascript
var SHEETS_WEB_APP_URL = "https://script.google.com/macros/s/XXXXXXXX/exec";
```

## 5) Test end-to-end

1. Submit a test order from the landing page.
2. Confirm new row appears in `Orders`.
3. Confirm both forms (`form-1` and `form-2`) write rows.

## Important note about future updates

If you edit Apps Script code later, you must deploy a **new version** and keep the latest Web App URL in `index.html`.
