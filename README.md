# Top Secret — Engagement Party Invitation

A two-page, self-contained static site. No build step, no framework, no server.

| File | What it is |
|---|---|
| `index.html` | The **TOP SECRET** landing page — big wordmark, surprise warning, and an envelope that opens and links through |
| `invite.html` | The invitation itself — details, countdown, and the RSVP form |

Live at `https://<your-github-username>.github.io/Top-Secret/` once Pages is on.

## Editing the details

Everything is in the **`CONFIG` block at the top of `invite.html`** — names,
date, venue, times, food, parking, dress code, RSVP settings. Nothing else needs
touching.

`secretName` drives every piece of the "don't tell her" messaging, so changing
it once updates the whole site.

## RSVPs → your Google Sheet (guests never see the sheet)

This is the setup the site ships with. Guests fill in the form on the page; a
Google Apps Script appends each reply to your private sheet. **Guests never see
the spreadsheet, the guest list, or anyone else's RSVP** — they only ever see
the form.

1. Open your Google Sheet. In the first row add these headers, in this order:

   `submittedAt | name | email | attending | guests | dietary | message`

2. **Extensions → Apps Script** (opening it this way matters — see
   troubleshooting), delete the placeholder, and paste this. Put your own sheet
   ID in `SHEET_ID` (it's the long string in the sheet's URL between `/d/` and
   `/edit`):

   ```javascript
   const SHEET_ID = 'PASTE_YOUR_SHEET_ID_HERE';

   function sheet_() {
     // openById works whether or not the script is bound to the sheet.
     return SpreadsheetApp.openById(SHEET_ID).getSheets()[0];
   }

   function doPost(e) {
     try {
       const d = JSON.parse(e.postData.contents);
       sheet_().appendRow([
         d.submittedAt, d.name, d.email,
         d.attending, d.guests, d.dietary, d.message
       ]);
       return json_({ ok: true });
     } catch (err) {
       return json_({ ok: false, error: String(err) });
     }
   }

   // Visit the /exec URL in a browser to check the deployment is healthy.
   function doGet() {
     try {
       const rows = sheet_().getLastRow();
       return json_({ ok: true, status: 'reachable', rowsSoFar: rows });
     } catch (err) {
       return json_({ ok: false, error: String(err) });
     }
   }

   function json_(obj) {
     return ContentService
       .createTextOutput(JSON.stringify(obj))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

3. **Deploy → New deployment → Web app.**
   - *Execute as:* **Me**
   - *Who has access:* **Anyone**

   "Anyone" lets the page submit without guests logging in. It exposes only the
   script's `doPost`/`doGet` — **not** the spreadsheet.

   The first deploy will prompt you to authorize the script. You must complete
   that (including the "Advanced → Go to project (unsafe)" screen, which appears
   because the script is unpublished and yours). Until it's authorized, the
   script cannot write to the sheet.

4. Copy the Web app URL (it ends in `/exec`) and paste it into
   `CONFIG.rsvpEndpoint` in `invite.html`.

### If RSVPs aren't reaching the sheet

**Open the `/exec` URL directly in a browser tab.** What you see identifies the
problem immediately:

| What you see | What it means | Fix |
|---|---|---|
| `{"ok":true,"status":"reachable",…}` | Working correctly | Nothing — submit a test RSVP |
| A Google **sign-in page** | Access isn't public | Redeploy with *Who has access:* **Anyone** |
| `{"ok":false,"error":"…"}` | Script runs but can't reach the sheet | Check `SHEET_ID` is correct |
| **"Script function not found: doGet"** | An older version is deployed | **Deploy → Manage deployments → Edit → Version: New version** |

Two traps worth knowing:

- **`getActiveSpreadsheet()` returns `null`** in a standalone script (one made at
  script.google.com rather than from *Extensions → Apps Script* inside the
  sheet). The script above avoids this entirely by using `openById`.
- **Editing the code does not update the live URL.** Apps Script serves the
  deployed *version*, so after any edit you must go to
  **Deploy → Manage deployments → Edit (pencil) → Version: New version → Deploy.**

> **Keep the sheet itself private.** In the Sheet's **Share** dialog, General
> access should stay **Restricted**. Don't put an "anyone with the link" sheet
> URL on the site — that would expose the whole guest list, which for a surprise
> party is the one thing you can't afford to leak.

### Alternatives

- **A Google Form instead:** put the form's URL in `CONFIG.rsvpFormUrl` and the
  RSVP card becomes a button that opens it. Responses feed a sheet you own, and
  guests still can't see each other's answers.
- **Email only (zero setup):** leave `rsvpEndpoint` blank and set
  `CONFIG.rsvpEmail`. The button opens the guest's mail app pre-filled.

## Go live on GitHub Pages

**Settings → Pages → Build and deployment** → Source: **Deploy from a branch**,
branch **main**, folder **/ (root)**, Save. `.nojekyll` is already included.

Note that a Pages site is public to anyone with the link — there's no password.
Share the URL only with invited guests.
