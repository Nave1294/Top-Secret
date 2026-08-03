# Engagement Party RSVP

A single-file, self-contained RSVP site for an engagement party. No build step,
no framework, no backend to run — just an `index.html` you host on GitHub Pages
(or anywhere static).

Once GitHub Pages is enabled, it's live at:

```
https://<your-github-username>.github.io/Top-Secret/
```

## 1. Fill in your details

Everything you need to edit is in the **`CONFIG` block at the top of
`index.html`** — the couple's names, date/time, venue, dress code, the note,
registry link, RSVP deadline. Nothing else in the file needs touching.

The field that drives the countdown clock is `dateTimeISO` — use a full ISO
timestamp with the correct timezone offset (`-04:00` = Eastern Daylight Time,
`-05:00` = Eastern Standard Time).

## 2. Choose how RSVPs reach you

### Option A — Google Sheet (recommended)

RSVPs append to a Google Sheet you own. One-time setup:

1. Create a new Google Sheet. In the first row add these headers (order matters):
   `submittedAt | name | email | attending | guests | dietary | message`
2. **Extensions → Apps Script**, delete the placeholder, and paste:

   ```javascript
   function doPost(e) {
     const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
     const d = JSON.parse(e.postData.contents);
     sheet.appendRow([
       d.submittedAt, d.name, d.email,
       d.attending, d.guests, d.dietary, d.message
     ]);
     return ContentService
       .createTextOutput(JSON.stringify({ ok: true }))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

3. **Deploy → New deployment → Web app.**
   - *Execute as:* **Me**
   - *Who has access:* **Anyone**
4. Copy the Web app URL (ends in `/exec`) and paste it into
   `CONFIG.rsvpEndpoint` in `index.html`.

Submissions then land in the sheet in real time. (The page posts with
`mode: "no-cors"`, so there's no CORS to fight; Apps Script records the row
regardless.)

### Option B — Email fallback (zero setup)

Leave `CONFIG.rsvpEndpoint` as `""` and set `CONFIG.rsvpEmail` to your address.
The **Send RSVP** button opens the guest's email app pre-filled with their
answers — they just hit send, and you tally replies in your inbox.

## 3. Go live on GitHub Pages

In the repo: **Settings → Pages → Build and deployment**, set *Source* to
**Deploy from a branch**, branch **main** / **/(root)**, and save. Give it a
minute and your RSVP site is live at the URL above. (`.nojekyll` is included so
Pages serves the files as-is.)
