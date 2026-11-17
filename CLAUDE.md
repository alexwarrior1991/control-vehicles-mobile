# CLAUDE.md

## Project

`control-vehicles-mobile`: the drivers' phone app of
[control-vehicles](https://github.com/alexwarrior1991/control-vehicles) (a JavaFX desktop app, separate
repository). `README.md` explains the setup; `CONTRACT.md` is the API between the phone, the Google Sheet and the
desktop. Code, comments in the app and docs: English for docs and code names; the phone app's texts are Spanish and
Portuguese (`app/i18n.js`).

## Commands

```bash
npm test        # backend: the real apps-script/Code.gs in Node over an in-memory Sheet (test/appsScript.mjs)
npm run serve   # local app + fake backend on :8765, prints a demo driver link
npm run e2e     # Playwright, phone viewport, fake GPS, against the fake server
```

## Rules

- **No build step and no runtime dependencies.** `app/` is served as it is by GitHub Pages
  (`.github/workflows/pages.yml`); `apps-script/Code.gs` is pasted into Apps Script as it is (ES5-style `var`
  and functions, no modules).
- **`CONTRACT.md` is shared with the desktop repository** (`MobileClient`, `MobileService` there): a change to
  a call or a field is made in both repositories in the same change, and fields are only added.
- The phone never loses a press: every action goes to the queue in `localStorage` first and is sent in order;
  the backend is idempotent by the trip id the phone generates, so resending is always safe.
- The backend answers JSON `{ok:false, error}` instead of throwing; the Sheet stores everything as text (dates are
  the phone's local time `yyyy-MM-ddTHH:mm`).
- The look is a dark terminal style in JetBrains Mono, bundled in `app/fonts` (woff2, SIL Open Font License,
  `OFL.txt` next to it) and cached by the service worker, so it works offline and on any phone. Bump the cache
  name in `app/sw.js` whenever a cached file changes.
- Screens are shown and hidden with the `hidden` attribute (`[hidden]{display:none!important}` in the CSS): the
  e2e test checks only one screen is visible at a time.
- Tests: `test/backend.test.mjs` (backend rules), `test/e2e.mjs` (the app end to end, Spanish and Portuguese,
  offline queue, cancel, revoked link). The fake server (`test/fakeServer.mjs`) runs the real Code.gs.
