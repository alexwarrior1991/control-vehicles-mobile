# Control Vehicles · phone app for drivers

Two big buttons on the driver's phone, **EMPIEZO** and **TERMINO** (Portuguese: **COMEÇO** / **TERMINO**): each
press takes the time and the GPS position by itself. The driver only types the odometer at the start (and, if
they want, at the end and the tolls). It is saved at once, and without coverage it waits on the phone and is sent
when the network comes back. The trips reach the desktop application
[control-vehicles](https://github.com/alexwarrior1991/control-vehicles) with *Trips → Bring trips from mobile*,
where Google computes the route and the costs and divergences as for any other trip.

```
phone app (GitHub Pages)  ──start / end──▶  Google Sheet + Apps Script  ◀──publish / export / ack──  desktop app
```

- `app/`: the phone app, plain HTML/CSS/JavaScript, installable (PWA), Spanish or Portuguese by the phone's
  language. No build step.
- `apps-script/Code.gs`: the backend, a Google Apps Script over a Google Sheet in your own Google account. Free,
  no server to look after; the Sheet is also a readable log of every trip.
- `test/`: the backend run in Node over an in-memory Sheet, a local server with a fake backend, and the end-to-end
  test of the app in a phone-sized browser with a fake GPS.
- [`CONTRACT.md`](CONTRACT.md): what the three parts send each other.

## Setting it up (once)

### 1. The Google Sheet and its script
1. In Google Drive create a new **Google Sheet**, e.g. «Control Vehicles · trips».
2. **Extensions → Apps Script**. Delete what is there, paste the whole of `apps-script/Code.gs` and save.
3. In the function list at the top choose **setup** and press **Run**. Google asks for permission (it is your own
   script): accept. The Sheet now has the tabs `Trips`, `Drivers` and `Vehicles`.
4. Open **View → Logs** (or *Execution log*) and copy the **admin secret** shown there. It is also in *Project
   settings → Script properties → ADMIN_SECRET*.
5. **Deploy → New deployment** → type **Web app** → *Execute as*: **Me**, *Who has access*: **Anyone** →
   **Deploy**. Copy the **Web app URL** (`https://script.google.com/macros/s/…/exec`).

«Anyone» is needed because the drivers' phones do not sign in to Google; nobody can do anything without a
driver's personal link or the admin secret. When you change `Code.gs` later, use **Deploy → Manage deployments →
Edit → New version**, so the URL stays the same.

### 2. Publish the phone app
In this repository on GitHub: **Settings → Pages → Source: GitHub Actions**. Every push to `master` then
publishes `app/` to **https://alexwarrior1991.github.io/control-vehicles-mobile/** (after the tests pass).

### 3. Connect the desktop app
In Control Vehicles: **Settings → Mobile**: paste the **Web app URL** and the **admin secret** (they stay on that
computer, never in the exported database), press **Test**, then **Publish drivers and vehicles**. Publish again
whenever a driver or a vehicle is added or changed.

### 4. Give each driver their link
Still in **Settings → Mobile**, choose the driver and press **Copy link**; send it to them (WhatsApp, e-mail…).
The link is personal: whoever has it can register trips as that driver. **New link** revokes the old one.

## For the driver

1. Open the link on the phone.
2. Add it to the home screen, so it opens like an app:
   - **Android (Chrome)**: menu ⋮ → **Add to home screen** / **Install app**.
   - **iPhone (Safari)**: share button → **Add to Home Screen**. (Open the link in Safari, not inside WhatsApp.)
3. The first time, allow the **location** when asked.
4. Starting: choose the vehicle (it is remembered), type the odometer, press **EMPIEZO**.
5. Arriving: optionally the odometer and the tolls, press **TERMINO**. A trip started by mistake can be cancelled.
6. «Todo guardado ✓» at the bottom means the office has it; «Pendiente de enviar» means it is kept on the phone and
   will be sent by itself when there is coverage.

## Development

Node 20 or newer; nothing to install to run the backend tests or the local server.

```bash
npm test          # the real Code.gs over an in-memory Sheet
npm run serve     # http://localhost:8765 with a demo driver (the link is printed)
npm install && npx playwright install chromium   # once, for the end-to-end test
npm run e2e       # the app in a phone-sized browser with a fake GPS; optional folder for screenshots
```

WebStorm has the three as run configurations (`npm serve`, `npm test`, `npm e2e`).
