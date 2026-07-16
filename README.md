# Live Translate — Browser Extension

A Chrome/Edge extension that translates any web page into **Mexican Spanish** on
demand, live in the browser. Click a page, click "Translate page," and the
visible text flips to natural es-MX — with a "Restore page" button to flip it
back.

The extension is a thin client: it talks to a backend translation service over
HTTP. It doesn't run any translation logic itself.

## What's in this repo

```
extension/
├── manifest.json            Chrome MV3 manifest
├── content.js                Reads the saved backend URL, relays popup commands
├── translation-widget.js     The on-page widget (floating button + panel)
├── popup.html / popup.js     Extension toolbar popup (set backend URL, controls)
└── icons/                    Toolbar icons
```

## Prerequisites

- Google Chrome or Microsoft Edge (any Chromium-based browser with MV3 support)
- A running backend that implements the translation API contract (see below).
  This extension does not include a backend — point it at one you already have
  running, either locally or deployed.

## Install

1. Open `chrome://extensions` (or `edge://extensions`)
2. Enable **Developer mode** (toggle, usually top-right)
3. Click **Load unpacked**
4. Select the `extension/` folder from this repo

The extension icon should now appear in your browser toolbar.

## Configure the backend URL

1. Click the extension's toolbar icon to open the popup
2. In the **"Backend (Node gateway) URL"** field, enter your backend's base URL
   (e.g. `http://localhost:8787` for a local backend, or your own deployed URL)
3. Click **Save**
4. Click **Check backend** to confirm it's reachable — you should see
   `Backend OK · {...}`
5. **Reload any already-open tabs** you want to translate — the saved URL only
   takes effect on pages loaded after you save it

## Use it

1. Go to any web page
2. Click the floating round button in the bottom-right corner
3. Click **Translate page** — the visible text on the page translates in place
4. Click **Restore page** to revert to the original text
5. On a second translate of text you've already seen, you'll notice a cache-hit
   badge and a much faster response — that's the backend's caching at work, not
   this extension

You can also trigger **Translate this page** / **Restore page** directly from
the popup instead of the on-page button — both do the same thing.

## Backend API contract

The extension expects the configured backend to implement:

```
POST /translate
  { "text": "Good morning", "target": "es-MX" }
  → { "translated": "Buenos días", "cached": false, "latencyMs": 812, "model": "..." }

POST /translate/batch
  { "texts": ["Home", "Add to cart"], "target": "es-MX" }
  → { "results": [{ "translated": "...", "cached": false }, ...], "latencyMs": 40 }

GET /health
  → { "status": "ok", ... }
```

Any backend that speaks this contract works with this extension unmodified.

## Troubleshooting

- **"Can't reach backend at ..."** — the backend isn't running, or the saved
  URL doesn't match where it's actually listening. Re-check the popup's
  "Check backend" button.
- **Nothing happens after reconfiguring the URL** — reload the page. The
  extension reads its saved config once per page load, not live.
- **Widget doesn't appear on a page** — some sites block extensions with a
  strict Content Security Policy. This extension's content-script injection
  is generally unaffected by page CSP, but if you see console errors
  mentioning CSP, that's the site restricting script execution, not a bug in
  this extension.
- **Some text stays untranslated** — the extension translates the page's text
  as it exists at the moment you click. Content that loads in *after* that
  (lazy-loaded sections, dynamic ads, infinite-scroll content) won't be
  caught until you translate again.

## Privacy note

This extension sends the visible text of pages you choose to translate to
whatever backend URL you configure. Don't point it at a backend you don't
trust, and don't use it on pages containing sensitive information you don't
want sent to your translation backend.
