# DOCKBOARD — Warehouse Picking App

A single-file web app for warehouse picking: inventory + pick orders stored in a
Google Sheet, scanned with a phone/tablet camera **or** a USB/Bluetooth barcode
scanner (which just types like a keyboard).

Everything runs in the browser and talks directly to Google's Sheets API — there's
no backend server to run or maintain.

---

## 1. One-time Google setup (you do this once)

Google requires any app that touches your Drive/Sheets to be registered.

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a
   new project (or use an existing one).
2. **Enable APIs**: search for and enable the **Google Sheets API**.
3. **Configure OAuth consent screen**: choose "External" (or "Internal" if you're on
   Google Workspace), fill in an app name, your email, and add your own account as a
   test user if the app stays in "Testing" mode.
4. **Create credentials → OAuth client ID → Web application**.
   - Under "Authorized JavaScript origins", add the URL you'll host the app at
     (e.g. `https://yourname.github.io` or `http://localhost:8080` while testing).
   - Copy the generated **Client ID** (looks like `xxxx.apps.googleusercontent.com`).

You'll paste that Client ID into the app's Settings tab — it's stored only in your
browser's local storage, never sent anywhere else.

## 2. Hosting the app

The app is a single `index.html` file. Any static host works — pick whichever is
easiest for you:

- **GitHub Pages** — push this folder to a repo, enable Pages. Free, gives you a
  stable HTTPS URL (needed for OAuth and for camera access, which browsers block on
  plain HTTP except `localhost`).
- **Netlify / Vercel (drag-and-drop)** — drop the folder in, get a URL instantly.
- **Your own web server / intranet** — copy `index.html` anywhere Apache/Nginx/IIS
  serves static files. Good if the app should stay inside your company network.
- **Just testing locally** — run `python3 -m http.server 8080` in this folder and
  open `http://localhost:8080`.

Whichever URL you land on, make sure it's added under "Authorized JavaScript
origins" in step 1.4 above.

## 3. First run

1. Open the hosted app, go to **Settings**, paste your Client ID, save.
2. Click **Sign in with Google**.
3. In Settings, either:
   - **Create New Sheet** — the app creates a new Google Sheet in your Drive with
     the right tabs and headers, or
   - **Connect Sheet** — paste the ID of a sheet you already made (the long string
     in its URL between `/d/` and `/edit`).

## 4. Sheet structure

The app expects two tabs:

**Inventory**
| SKU | Description | Location | Quantity |
|-----|-------------|----------|----------|

**Orders** (one row per line item — an order with 3 SKUs is 3 rows sharing the same OrderID)
| OrderID | SKU | Description | QtyOrdered | QtyPicked | Status |
|---------|-----|--------------|------------|-----------|--------|

You can bulk-load inventory via **Settings → Import inventory from CSV**
(columns: SKU, Description, Location, Quantity). Pick orders are simplest to create
by editing the Orders sheet directly in Google Sheets (or export orders there from
whatever system generates them) — leave QtyPicked and Status blank for new orders.

**Important:** the SKU column value is what gets matched against a scanned barcode,
so your barcode labels need to encode the same value that's in the SKU column.

## 5. Using it in the warehouse

- **Orders tab** lists every order with progress and status (Open / In progress /
  Complete).
- Tap an order to open the **picking screen**, which lists every line item still to
  pick.
- **USB/Bluetooth scanner**: the scan box at the bottom stays focused — just scan,
  the scanner "types" the code and hits Enter automatically.
- **Phone/tablet camera**: tap the 📷 button to open the camera scanner.
- Each successful scan flashes green and beeps, decrements the order line and the
  Inventory quantity for that SKU, and marks the order Complete once every line is
  fully picked. A scan that doesn't match anything in the order flashes red.

## Notes / limitations

- This is a lightweight, single-warehouse tool built directly on Sheets — fine for
  small-to-mid volume. If you outgrow Sheets (very high order volume, many
  simultaneous pickers), a proper database backend would be the next step.
- Camera scanning needs HTTPS (or `localhost`) and camera permission.
- Google's OAuth token expires periodically; if writes start failing, sign out and
  back in.
