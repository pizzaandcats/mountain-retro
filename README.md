# Mountain Retro landing page

Static site. No build step. Upload the whole folder to any static host (Netlify, Vercel, Cloudflare Pages, GitHub Pages, Shopify theme assets, etc.).

```
index.html            Landing page (Kit form embedded in the "First Dibs on the Trail" coupon)
thank-you/index.html  Post-signup page with the "What should we make next?" poll
assets/site.css       Shared styles (brand tokens from the Mountain Retro design system)
assets/fonts/         Jost (self-hosted, SIL Open Font License, see OFL-LICENSE.txt)
assets/img/           Approved logo SVGs (unchanged) + web-sized versions of the porch photo
```

## Before going live

1. **Kit redirect.** In Kit, set the form's success action to redirect to `https://YOURDOMAIN/thank-you/`.
2. **Kit form type.** The embed is placed inline inside the coupon. This only renders in place if form `367724b386` is an *inline* form in Kit. If it's a modal, slide-in or sticky bar, switch it to inline (or make a new inline form and swap the `data-uid`).
3. **Kit form styling.** `index.html` restyles Kit's standard classes (`.formkit-input`, `.formkit-submit`, etc.) so the form matches the page. Check it once live; if Kit's template uses different markup, tweak the `#kit-form` rules.
4. **Poll storage (required for votes to be saved).** See below. Until it's set up, the thank-you page shows a red "Setup needed" bar and tells voters their vote was not saved.
5. **Social preview image.** `og:image` is a relative path. Change it to the full URL once the domain is set.

## Poll storage: Google Sheets in about 5 minutes

1. Create a Google Sheet. Add headers in row 1: `submitted_at | choice | email | page`.
2. Extensions > Apps Script. Replace the code with:

```js
const ALLOWED = ['ringer-tee', 'retro-trail-shorts', 'more-sock-designs', 'trailhead-cap', 'basecamp-beanie'];

function doPost(e) {
  let data = {};
  try { data = JSON.parse(e.postData.contents); } catch (err) {}
  if (!ALLOWED.includes(data.choice)) {
    return ContentService.createTextOutput(JSON.stringify({ ok: false }))
      .setMimeType(ContentService.MimeType.JSON);
  }
  SpreadsheetApp.getActiveSpreadsheet().getSheets()[0]
    .appendRow([new Date(), data.choice, String(data.email || '').slice(0, 254), String(data.page || '').slice(0, 100)]);
  return ContentService.createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Deploy > New deployment > Web app. Execute as: Me. Who has access: Anyone. Copy the `/exec` URL.
4. In `thank-you/index.html`, set `window.MR_POLL_ENDPOINT = "https://script.google.com/macros/s/.../exec";`
5. Vote once on the live page and confirm a row lands in the sheet.

Any endpoint that accepts a POST with a JSON body (sent as `text/plain`) and returns HTTP 200 works the same way.

Notes:
- A vote only shows "Vote counted" after the endpoint confirms the save. Failures say the vote wasn't saved.
- Each vote is saved with the voter's email. The landing page remembers the address typed into the Kit form, so the poll's email field is usually filled in already; voters who arrive another way type it in.
- The browser remembers a confirmed vote so the same visitor isn't asked twice. This is a soft limit, not fraud protection.
- When you change the script, save it, then Deploy > Manage deployments > edit (pencil) > Version: New version > Deploy. This keeps the same `/exec` URL.
