# Wiring drawings into a Google Drive folder

The page is ready to POST PNGs to a Google Apps Script endpoint. Set it up once and every drawing the user sends will land in a Drive folder you control.

## 1. Create the Drive folder

1. Go to [drive.google.com](https://drive.google.com) and create a folder, e.g. **Hackathon 2026 — Drawings**.
2. Open the folder and copy the folder ID from the URL:
   `https://drive.google.com/drive/folders/`**`THIS_PART_IS_THE_ID`**

## 2. Create the Apps Script

1. Go to [script.google.com](https://script.google.com) → **New project**.
2. Replace the default `Code.gs` with the script below.
3. Paste your folder ID into `FOLDER_ID`.

```javascript
const FOLDER_ID = "PASTE_YOUR_FOLDER_ID_HERE";

function doPost(e) {
  try {
    const body = JSON.parse(e.postData.contents);
    const base64 = body.image.replace(/^data:image\/png;base64,/, "");
    const blob = Utilities.newBlob(
      Utilities.base64Decode(base64),
      "image/png",
      `${body.filename}-${new Date().toISOString().replace(/[:.]/g, "-")}.png`
    );
    DriveApp.getFolderById(FOLDER_ID).createFile(blob);
    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## 3. Deploy as a Web App

1. **Deploy → New deployment**.
2. Type: **Web app**.
3. Execute as: **Me**.
4. Who has access: **Anyone** (required so the static site can POST without auth).
5. Click **Deploy**, authorize when prompted.
6. Copy the **Web app URL** — it looks like `https://script.google.com/macros/s/AKfy.../exec`.

## 4. Paste the URL into the page

Open [paint89/index.html](index.html), find this line near the top of the `<script>` block:

```js
const DRIVE_ENDPOINT = "";
```

Paste your Web app URL between the quotes. Save. Done.

## Test it

1. Open the page, draw something.
2. Click the **upload** tool (the arrow-up icon, between Clear and Save).
3. Type a name → submit.
4. Refresh your Drive folder — the PNG should appear, timestamped.

## Notes

- The "Anyone" access scope means anyone with the script URL can POST. The script only accepts a filename and image, and only writes to the folder you set — so the blast radius is "the folder fills with junk", not "data leaks". For a hackathon this is fine. If you want stronger control, add a shared secret in the POST body and check it in `doPost`.
- If you change the script, redeploy (**Deploy → Manage deployments → ✏️ → New version**). The URL stays the same.
- CORS: Apps Script handles cross-origin POSTs from any page automatically as long as the content-type is `text/plain` (which the page already uses).
