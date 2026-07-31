# Check-in-app
# Check In — a two-device alert app

A tiny app: one phone shows tap-to-send tiles ("Need Water," "Need Snack," "Need
Hugs," or custom ones you add), the other phone gets notified in real time. No
app store account, no Xcode, no cost.

It works as a **PWA** (Progressive Web App) — a website that installs to the
iPhone home screen and behaves like a real app (full screen, its own icon, no
browser bar).

Total setup time: about 15 minutes, once.

---

## What you need (both free)

1. A **GitHub** account — to host the app's files.
2. A **Firebase** account (Google) — to sync alerts between the two phones in
   real time. The free "Spark" plan is more than enough for this; no credit
   card required.

---

## Step 1 — Create the Firebase project (~5 min)

1. Go to https://console.firebase.google.com → **Add project** → give it any
   name (e.g. "check-in-app") → you can skip Google Analytics → **Create**.
2. In the left sidebar: **Build → Firestore Database → Create database** →
   choose **Start in test mode** → pick any region → **Enable**.
   - Test mode keeps this simple to set up. It means anyone with your
     Firestore URL could technically read/write it, which is fine for a
     private tool like this, but you can tighten the rules later (see
     "Optional: locking it down" below).
3. In the left sidebar gear icon → **Project settings** → scroll to
   **Your apps** → click the **`</>`** (web) icon → register an app (any
   nickname, no need for hosting) → **Register app**.
4. Firebase will show you a `firebaseConfig` object that looks like:
   ```js
   {
     apiKey: "AIza...",
     authDomain: "check-in-app.firebaseapp.com",
     projectId: "check-in-app",
     storageBucket: "check-in-app.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   }
   ```
   Copy this whole object — you'll paste it into the app on first launch on
   **each** phone. (This config is safe to be public; it's not a secret key.)

---

## Step 2 — Put the files on GitHub Pages (~5 min)

1. Go to https://github.com/new → create a new **public** repository (e.g.
   `check-in-app`).
2. Upload all 5 files from this project (`index.html`, `manifest.json`,
   `sw.js`, `icon-192.png`, `icon-512.png`) — you can drag-and-drop them on
   the repo's "Add file → Upload files" page.
3. Go to the repo's **Settings → Pages** → under "Build and deployment,"
   set **Source: Deploy from a branch**, **Branch: main / (root)** → **Save**.
4. GitHub will give you a live URL after a minute or two, like:
   `https://yourusername.github.io/check-in-app/`

---

## Step 3 — Install on both iPhones (~2 min each)

1. Open the GitHub Pages URL in **Safari** on the iPhone (must be Safari,
   not Chrome, for the install step to work on iOS).
2. Tap the **Share** icon → **Add to Home Screen** → **Add**.
3. Open the new "Check In" icon from the home screen.
4. On first launch it'll ask you to paste the Firebase config from Step 1 —
   paste the same object on **both** phones.

---

## Step 4 — Pair the two phones

- On the phone that will **receive** alerts (yours): choose
  **🔔 Receiving alerts**, then **This is my code, continue**. Note the
  6-character code shown.
- On the phone that will **send** alerts: choose **🙋 Sending alerts**, then
  type in that same code and tap **Pair this device**.

That's it — tapping a tile on the sender's phone now creates a Firestore
document, and the receiver's phone (if the app is open or recently
backgrounded) shows a toast, buzzes, and plays a soft chime.

---

## Adding your own alert tiles

On the sending phone: **Alerts tab → ＋** → type a name, pick an emoji, save.
Tiles can be removed from **Settings → Manage tiles**.

---

## Good to know / honest limitations

- **Alerts arrive instantly while the app is open, or recently used in the
  background.** True "wake the phone up even if the app's been closed for a
  while" push notifications on iOS require Apple's push service (APNs) or a
  small server that calls Firebase Cloud Messaging — that's a bigger lift
  involving Firebase's paid-but-still-effectively-free "Blaze" plan (it
  needs a card on file even though normal usage stays in the free tier). If
  you want, this can be added later — just ask.
- **Realistically**, keep the receiving phone's Check In app open (or
  glance at it) rather than expecting a locked, fully-closed phone to buzz
  like a text message, unless you set up the fuller push option above.
- Data is stored in your own free Firestore project — only visible to
  people with your project's config, which only you two have.

### Optional: locking it down further
Firestore's free "test mode" rules expire after 30 days and then block all
access, which will break the app. Before that happens, go to **Firestore →
Rules** and replace the contents with something like:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /pairs/{pairCode}/alerts/{alertId} {
      allow read, write: if true;
    }
  }
}
```
then **Publish**. (This keeps it open only to people who know a specific
pairing code, which is reasonably private for a two-person tool like this.)
