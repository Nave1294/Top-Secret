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

   ("Anyone" lets the page submit without guests needing to log in. It only
   grants access to the script's `doPost` — **not** to the spreadsheet.)

4. Copy the Web app URL (it ends in `/exec`) and paste it into
   `CONFIG.rsvpEndpoint` in `invite.html`.

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
