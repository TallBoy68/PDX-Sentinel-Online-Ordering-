# Domaine Serene — In-Room Ordering App
## Setup & Deployment Guide

---

## Files in This Package

| File | Purpose |
|------|---------|
| `index.html` | Guest-facing ordering app (QR code destination) |
| `kitchen.html` | Staff/kitchen display screen |
| `menu.json` | Editable menu — add/remove items without touching code |
| `README.md` | This file |

---

## Step 1 — Set Up Firebase (one-time, ~10 minutes)

This app uses **Firebase Realtime Database** for live order sync between guest app and kitchen display.

1. Go to [https://console.firebase.google.com/](https://console.firebase.google.com/)
2. Click **Add project** → name it `ds-portland-lounge` (or similar)
3. Once created, click **Web** (</>) to register a web app
4. Copy the `firebaseConfig` object shown — you'll need it in Step 2
5. In the left sidebar, click **Build → Realtime Database**
6. Click **Create database** → choose **Start in test mode** (we'll lock it down later)
7. Select your region (us-central1 recommended)

### Firebase Security Rules

In the Realtime Database console, click the **Rules** tab and paste:

```json
{
  "rules": {
    "orders": {
      ".read":  true,
      ".write": true
    }
  }
}
```

> **Note:** These open rules are fine for internal use. For production, you can restrict writes by adding server-side validation or using Firebase App Check.

---

## Step 2 — Configure Both HTML Files

Open **both** `index.html` and `kitchen.html` in a text editor.

Find the `FIREBASE_CONFIG` block near the top of the `<script>` section in each file:

```javascript
const FIREBASE_CONFIG = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  databaseURL:       "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  ...
};
```

Replace all the placeholder values with your actual Firebase config from Step 1.

**This must be done in both files — they use the same database.**

---

## Step 3 — Edit the Menu

Open `menu.json` in any text editor (TextEdit, Notepad, VS Code, etc.).

### Structure

```json
{
  "categories": [
    {
      "id": "wine-glass",
      "label": "Wines by the Glass",
      "items": [
        {
          "id": "wg-01",
          "name": "Evenstad Reserve Pinot Noir",
          "vintage": "2021",
          "description": "...",
          "price": 28,
          "image": ""
        }
      ]
    }
  ]
}
```

### Rules
- `id` values must be **unique** across all items (use `wg-`, `wb-`, `fd-` prefixes for glass/bottle/food)
- `vintage` can be left as `""` for food items
- `price` is a number (no dollar sign)
- `image` is reserved for future use — leave as `""`
- Add as many categories and items as you need

---

## Step 4 — Deploy to GitHub Pages

1. Create a new repository at [github.com](https://github.com) (e.g., `ds-inroom-ordering`)
2. Upload all four files: `index.html`, `kitchen.html`, `menu.json`, `README.md`
3. Go to **Settings → Pages** → Source: **Deploy from a branch** → Branch: `main`, folder: `/ (root)`
4. Click Save. Your site will be live at:
   - Guest app: `https://YOUR-USERNAME.github.io/ds-inroom-ordering/`
   - Kitchen display: `https://YOUR-USERNAME.github.io/ds-inroom-ordering/kitchen.html`

---

## Step 5 — Generate QR Codes

Once deployed, generate a QR code for each room (or one universal code):

**Universal QR (one code, guests enter room number):**
- Point to: `https://YOUR-USERNAME.github.io/ds-inroom-ordering/`
- Generate free QR at: [qr-code-generator.com](https://www.qr-code-generator.com)

**Per-room QR (future):** Can append `?room=412` to pre-fill the room number field.

---

## Kitchen Display Setup

- Open `kitchen.html` on the kitchen tablet in Chrome or Safari
- Bookmark the page for quick access
- The display auto-refreshes in real time — no manual refresh needed
- Orders appear instantly when submitted by guests
- Staff use the **Start** and **Mark Complete** buttons on each order card
- Orders older than 15 minutes show in red — urgent indicator

---

## Demo / Testing Mode

If Firebase is **not yet configured**, both apps run in demo mode:
- The guest app logs orders to the browser console instead of Firebase
- The kitchen display shows a **"+ Inject Demo Order"** button to test the card layout
- No data is saved in demo mode

---

## Customization Notes

### Changing Colors / Branding
In `index.html`, the `:root` block at the top of `<style>` contains all color tokens:

```css
--navy:     #1F3864;   /* Primary brand color */
--burgundy: #7B2D00;   /* Accent / submit button */
--gold:     #C9A84C;   /* Cart badge / accents */
```

### Removing the Demo Button (Kitchen Display)
In `kitchen.html`, delete the `<button class="demo-btn" ...>` element before going to production.

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Menu not loading | Ensure `menu.json` is in the same folder as `index.html` |
| Orders not appearing on kitchen display | Check Firebase config is identical in both files |
| Firebase permission denied | Verify database rules are set to allow read/write |
| QR code not working | Test the URL in a browser first; ensure GitHub Pages is published |

---

## Phase 2 Ideas (for future development)
- Pre-fill room number via QR code URL parameter (`?room=412`)
- Menu management UI (edit `menu.json` through a browser interface)
- Order history export (CSV download for end-of-day reporting)
- HMS/Infor C7 integration for automatic room folio posting
- Special requests / allergy notes field (already wired, just needs enabling)
