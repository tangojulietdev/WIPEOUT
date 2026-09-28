# Wipe Out

A single-page installable PWA for running a car wash and auto spa. Fully online: owner and employee logins sync live across every device through Firebase. Wash queue (waiting/washing/done), an editable price catalog by vehicle type and service, WhatsApp "wash done" messages, cloud number-plate scanning, UPI collection, daily/weekly/monthly/yearly insights, a searchable customer database, finance and payroll (owner only), and team/employee-login management.

**This build is multi-tenant.** One Firebase project can host any number of unrelated car wash shops at once, each one's data is fully isolated from every other's by Firestore security rules. Anyone who opens the app and taps **Create shop** gets their own brand-new, private shop, they can never see or touch another shop's washes, customers, finance or staff. You only set up the Firebase project once (below), then this exact same file (or its hosted link) is what you hand to every customer — no per-customer editing needed.

## What is inside

| File | Purpose |
|---|---|
| `index.html` | The whole app (HTML, CSS, JavaScript in one file) |
| `manifest.webmanifest` | PWA metadata so it installs to the home screen |
| `sw.js` | Service worker, caches the app shell only for fast loads and installability |
| `icons/` | App icons (192, 512, maskable, apple touch, favicon) |
| `firestore.rules` | Security rules to paste into your Firebase project |

## One-time setup (do this once, ever, not per customer)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project (any name).
2. In the project: **Build → Authentication → Get started → Email/Password → Enable → Save**.
3. **Build → Firestore Database → Create database**. Pick a region close to you (for India, `asia-south1`). Start in production mode.
4. Open **Firestore Database → Rules**, delete what's there, paste the contents of `firestore.rules` from this folder, and click **Publish**.
5. Go to **Project settings** (gear icon) → scroll to "Your apps" → click the `</>` web icon → register an app (any nickname, no hosting needed) → copy the `firebaseConfig` object it shows you.
6. Paste that config, and your Plate Recognizer API token, into the `BUILTIN_FIREBASE_CONFIG` and `BUILTIN_PR_TOKEN` constants near the top of the app's script in `index.html`, so every customer's copy connects automatically with no setup screen.

That's it, this project is now ready for unlimited shops.

## Onboarding a new customer

Just send them the app (this file, or its hosted link). They open it, tap **Create shop**, enter their name, shop name, email and a password, that's their fully isolated shop, live immediately. Nothing on your end changes per customer, no new Firebase project, no editing the file again.

For their staff: the shop owner opens **Team → Add employee login** inside the app themselves and creates logins for their own people. You don't need to be involved in that step at all.

## Run it locally

Open `index.html` in any modern browser. For the service worker and install prompt to work, it needs to be served over HTTP, not opened as a file. Quick local server:

```
cd wipeout
python3 -m http.server 8080
```

Then open `http://localhost:8080`.

## Deploy to GitHub Pages

1. Create a repo (for example `wipeout`) and push these files to the root.
2. In the repo: Settings, Pages, Source = `main` branch, `/root`.
3. Open the published URL on your phone, then use the browser menu, Add to Home screen / Install.

## Data, sync and roles

All data (washes, customers, finance, staff, catalog and settings) lives in Firestore, not on the device, so every device sees the same live data with no manual backup or export. An internet connection is required.

- **Owner account**: sees everything — Today, Insights, Customers, Finance (income, expenses, salary/advances), Team (add or remove employee logins), and Settings (catalog, prices, UPI ID, WhatsApp template).
- **Employee accounts** (created from Team → Add employee login): can only add and edit washes and view Customers. Finance and Team are hidden, both in the menu and on the server (Firestore rules block the request even if someone tries to bypass the app).
- **Every shop is isolated**: two different shops in this same project never see each other's data, enforced by `firestore.rules`, not just hidden in the app.
- **Shared quota**: on Firebase's free Spark plan this whole project gets 50K reads/day and 20K writes/day, shared across every shop combined, not per shop. Fine for a handful of small shops, worth watching (Firebase console → Usage) as you add more, and upgrading to the pay-as-you-go Blaze plan if you need headroom.

## Catalog and pricing

Settings → Vehicle types / Services lets you add, rename or remove entries as chips. The price list below groups by vehicle type; tap a type to expand it and set a price per service. On the Add wash screen, picking a type and service auto-fills the amount from this list — you can still overwrite it for one-off jobs.

## WhatsApp wash-done messages

Settings → WhatsApp wash update lets you edit the message template (placeholders: `{name}` `{vehicle}` `{plate}` `{amount}` `{shop}`) and toggle whether marking a wash "Done" prompts to send it. Sending opens WhatsApp with the message pre-filled — you tap Send. This uses the free `wa.me` link, not the paid WhatsApp Business API, so it can't be sent automatically without you tapping Send.

## Plate scanning

The Plate Recognizer token is baked into the app (`BUILTIN_PR_TOKEN`), not entered by shop owners, they never see it. Free tier is 2,500 scans/month, and because every shop in this project shares that one token, that quota is shared across every customer combined, not 2,500 each. Watch usage at platerecognizer.com if you're onboarding many shops, and swap in a paid tier or a second token if you outgrow it.

## UPI payments

Settings → Your UPI ID. The "Show UPI QR to collect" button on a wash, and "Collect" on a customer's dues, generate a UPI QR code and deep link. This is a free `upi://` link, not a payment gateway, so there is no automatic payment confirmation — you tap "Mark received" once the money is in.

## Notes

- Currency is Indian Rupee, formatted with the en-IN locale.
- A wash needs a vehicle number and an amount. Name and phone are optional but recommended, since the customer database and dues follow-ups are built from them.
- Payment options: Cash, UPI, Partial (records a part payment and a remaining due), or Due (full amount unpaid).
- Queue status (Waiting → Washing → Done) is separate from payment status, so you can track work in progress regardless of how or when it's paid.
- Outstanding dues appear automatically at the top of Customers, grouped by customer, with a one-tap call link.

Built under the Tango Juliet Dev brand.
