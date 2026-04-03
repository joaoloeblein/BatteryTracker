# ⚡ FRC Battery Tracker
### Built by Team 180 S.P.A.M. · Stuart, FL
### Free to use, modify, and share with the FRC community

A mobile-first web app for tracking 12V 18Ah SLA battery health during competition. Logs pre and post-match voltage, charge %, and internal resistance per battery per match. Syncs live across all devices via Firebase so your whole pit crew sees the same data in real time.

---

## What's Included

| File | Purpose |
|---|---|
| `index.html` | Full app — logging, fleet management, stats, QR codes |

---

## Features

- **Split pre/post logging** — one person saves pre-match readings before the robot goes on the field, a different person enters post-match data from any device after the match
- **Auto-filled match numbers** — the next match number per battery is pre-filled automatically
- **QR codes per battery** — print one per pack, scan it, the app opens with that battery pre-selected
- **Live sync** — all devices on the same pit network see data update in real time via Firebase Firestore
- **Stats & charts** — IR trend over time, fleet health ranking, voltage over time, charge drain per match, battery usage balance
- **Health alerts** — flags batteries with rising IR over 3 consecutive matches or consistently low post-match voltage
- **Read-only viewer** — a separate URL for anyone who needs to see the data but shouldn't be able to edit it
- **No install required** — works in any mobile browser, no app store needed

---

## Setup

You need a free Firebase account. This takes about 10 minutes.

### Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account
2. Click **Create a project**
3. Give it a name (e.g. `frc-batteries`) and click through the setup steps

### Step 2 — Register a web app

1. On the project home page, click the **Web icon** (`</>`)
2. Give it a nickname (anything, e.g. `battery-tracker`)
3. Click **Register App**
4. You will see a `firebaseConfig` block that looks like this:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

5. Copy these 6 values — you will need them in the next step

### Step 3 — Add your config to the HTML files

Open `index.html` in any text editor (Notepad, VS Code, TextEdit, etc.) and find this section near the top:

```js
var firebaseConfig = {
  apiKey:            "PASTE_API_KEY_HERE",
  authDomain:        "PASTE_AUTH_DOMAIN_HERE",
  projectId:         "PASTE_PROJECT_ID_HERE",
  storageBucket:     "PASTE_STORAGE_BUCKET_HERE",
  messagingSenderId: "PASTE_MESSAGING_SENDER_ID_HERE",
  appId:             "PASTE_APP_ID_HERE"
};
```

Replace each `PASTE_..._HERE` value with the matching value from Step 2. Save the file.

Repeat the same process for `index-readonly.html` using the exact same config values — both files point to the same database.

### Step 4 — Create the Firestore database

1. In the Firebase console, go to **Build → Firestore Database** in the left sidebar
2. Click **Create database**
3. Select **Start in test mode**
4. Choose a server location close to your region and click **Enable**

### Step 5 — Set Firestore rules

1. In Firestore, click the **Rules** tab
2. Replace the contents with the following and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

> **Note:** These rules allow anyone with your URL to read and write data. This is fine for a competition environment. If you want to restrict writes later, see the Advanced section below.

### Step 6 — Deploy

The easiest way to deploy is [Netlify Drop](https://app.netlify.com/drop):

1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Rename your `index.html` to `index.html` if it isn't already
3. Drag the file onto the page
4. Netlify gives you a free URL instantly (e.g. `random-name.netlify.app`)
5. You can rename the site under **Site settings → Change site name**

For the read-only viewer, repeat with `index-readonly.html` renamed to `index.html` on a **second** Netlify site so it gets its own URL.

That's it. Open the URL on your phone and start adding batteries.

---

## How to Use

### Registering batteries

1. Go to the **🔋 Fleet** tab
2. Type a battery ID (e.g. `BAT-01`) and tap **Add**
3. Repeat for each battery in your fleet

### Printing QR codes

1. In the Fleet tab, tap the **⬛ QR** button next to any battery to view and print its individual QR code
2. Tap **🖨 Print All QR Codes** to print a full sheet of all batteries at once (3 per row, ready to cut out)
3. Laminate the cards and attach them to your physical batteries

### Logging a match

**Pre-match (before the robot goes on the field):**
1. Scan the battery's QR code or open the app and go to the **📋 Log** tab
2. Select the battery — match number auto-fills to the next one
3. Measure voltage, charge %, and internal resistance with a Battery Beak or equivalent tool
4. Tap **▶ Save Pre-Match Data**

**Post-match (after the robot comes off the field):**
1. Go to the **⏳ Post** tab — batteries waiting for post-match data appear here
2. Tap **▼ Add Post-Match Data** on the correct battery
3. Enter post-match voltage, charge %, and IR
4. Tap **Save Post-Match ■**

The record is now complete and will appear in History and Stats.

### Reading the stats

- **Health Ranking** — batteries sorted by average internal resistance, lowest (best) to highest. GOOD is below 15mΩ, WATCH is 15–20mΩ, REPLACE is above 20mΩ
- **IR trend chart** — the most useful chart for spotting aging. A battery whose post-match IR climbs consistently over several matches is on its way out
- **Voltage chart** — pre vs. post voltage over time. Pre-match voltage shows charge quality; post-match shows how much the battery recovered
- **Drain chart** — voltage drop and charge % consumed per match. Spikes can indicate a hard match or a weaker battery

---

## Customizing for Your Team

Everything lives in the two HTML files. Common things teams change:

**Team name and branding** — search for `S.P.A.M.` and `Team 180` in the file and replace with your team name and number. The logo is embedded as a base64 image near the top — replace the `LOGO_SRC` variable with your own logo encoded the same way.

**IR thresholds** — the GOOD/WATCH/REPLACE thresholds are set at 15mΩ and 20mΩ. Search for those values in the `irColor` and `irLabel` functions and adjust for your batteries.

**Voltage floor** — the 12.0V warning line on the voltage chart is set in the `VoltageChart` component. Adjust to match your team's standards.

---

## Advanced — Restricting Writes

If you want the read-only viewer to be truly read-only at the database level (so even a technical user can't write through it), update your Firestore rules to this:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

This allows anyone to read but requires authentication to write. You would then need to add Firebase Authentication to the main app. Reply in the thread if you want help setting that up.

---

## Tech Stack

- Vanilla HTML, CSS, JavaScript — no build tools, no framework installs
- React 18 (loaded from CDN) with Babel standalone for JSX in the browser
- Firebase Firestore for real-time database sync
- Chart.js for the stats charts
- QRCode.js for QR code generation
- Google Fonts (Oswald, Barlow Condensed, Share Tech Mono)
- Deployed via Netlify Drop (free tier)

Total size: ~105KB for the full app including the embedded team logo.

---

## Credits

Built by **FRC Team 180 S.P.A.M.** — Stuart, FL  
*Inspiring STEM Leaders Through Robotics*

Free to use and modify. If you improve it, consider sharing back with the community.
