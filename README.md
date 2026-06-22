# KAIM Calculator — Google Play Store Submission Guide
### (No dedicated dev machine required — using a cloud build service)

This folder contains everything needed to wrap the KAIM Calculator web app
into a real Android app and submit it to Google Play, using Capacitor +
Codemagic (same cloud build service used for the iOS version, if you did
that one too).

---

## STEP 1 — Google Play Console account ($25 one-time)
1. Go to https://play.google.com/console/signup
2. Sign in with any Google account
3. Pay the one-time $25 registration fee (no recurring cost, unlike Apple)
4. Verify your identity — usually same-day, can take up to 48 hours

## STEP 2 — Put this project on GitHub
(Skip this if you already created a GitHub repo for the iOS version — you
can add Android to that same repo instead of making a new one.)
1. Create a free GitHub account if you don't have one: https://github.com
2. Create a new repository (e.g. "kaim-calculator-android")
3. Upload everything in this folder to that repo:
   - package.json
   - capacitor.config.json
   - www/index.html
   - resources/ (icons, feature graphic, listing copy, privacy policy)

## STEP 3 — Edit the Bundle ID
Open `capacitor.config.json` and change:
   "appId": "com.yourcompany.kaimcalculator"
to something unique to you, e.g.:
   "appId": "com.johnsmith.kaimcalculator"
(If you're also publishing on iOS, using the same appId on both keeps
things tidy, though it's not required — they're separate stores.)

## STEP 4 — Codemagic Android build
1. Go to https://codemagic.io and sign in (or sign up with GitHub)
2. Add this repo as an application if you haven't already
3. Choose the "Capacitor" workflow, but select **Android** as the target
   platform (if you already set up iOS in Codemagic for this same repo,
   you can just add Android as a second build target)
4. In Codemagic's signing settings, choose **"Generate keystore automatically"**
   — Android signing is simpler than Apple's; Codemagke can create and
   manage this for you with one click
5. Set Codemagic to publish directly to **Google Play** — under
   Distribution, connect your Google Play Console account via a service
   account JSON key (Codemagic's docs walk through generating this in
   Play Console: Setup > API access)
6. Start a build — Codemagic will produce an `.aab` (Android App Bundle)
   file and can upload it straight to Play Console's internal testing
   track automatically

## STEP 5 — Play Console listing
1. Go to https://play.google.com/console
2. All apps → Create app
3. Fill in using `resources/play-store-listing.md` in this folder — name,
   short description, full description, and category are already drafted
4. Upload `resources/icon-512.png` as the app icon
5. Upload `resources/feature-graphic-1024x500.png` as the feature graphic
6. Add at least 2 phone screenshots (see list in play-store-listing.md)
7. Add your **Privacy Policy URL** — host `resources/privacy-policy.md`
   somewhere public (GitHub Pages works, same as the iOS setup)
8. Complete the Content Rating questionnaire (IARC) — straightforward
   given this app has no ads, no user content, no in-app purchases

## STEP 6 — Submit for Review
1. Once Codemagic uploads a build to Play Console, promote it from
   Internal Testing to Production (or submit straight to production if
   you skip internal testing)
2. Fill in remaining required fields (pricing — free or paid, countries
   of availability, data safety form)
3. Click **Submit for Review**
4. Google's review is typically faster than Apple's — often same-day to
   a few days

---

## In-App Book Reader (NEW)

The app now includes a full in-app reader for the KAIM ebook, reachable
via the "📖 Read the Full KAIM Guide" button on the calculator's home
screen. This is built entirely from the same EPUB content — no separate
e-reader app needed, no internet connection required to read it.

**Files involved:**
- `www/book.html` — the reader shell (top bar, chapter dropdown,
  prev/next, paywall overlay with the book cover displayed)
- `www/book/sections/*.xhtml` — the book's actual content, unpacked
  directly from the EPUB, including a dedicated `cover.xhtml` page
- `www/book/images/cover.png` — the book cover image, shown both as the
  first page of the book and on the paywall screen
- `www/book/styles/stylesheet.css` — the book's styling, shared with
  the EPUB version

**Free preview vs. locked content:**
Currently set so the Cover and Introduction (`section0001`,
`section0002`) are free, and the methodology + both appendices +
references (`section0003`–`section0006`) are locked behind a paywall.
Adjust the `free: true/false` flags in the `CHAPTERS` array at the top
of `book.html`'s script if you want a different split.

### ⚠️ IMPORTANT — The purchase flow is currently a PLACEHOLDER

The "Unlock Full Guide" button in `book.html` does **not** charge real
money right now. It just sets a flag in the device's local storage so
you can test the unlock flow end-to-end. Look for the comments marked
`PLACEHOLDER` in `book.html`'s `<script>` block — that's exactly where
real billing needs to be wired in before this goes live.

**To make it a real purchase, you have two main options:**

**Option A — RevenueCat (recommended, easiest cross-platform path)**
1. Sign up free at https://www.revenuecat.com
2. Create your "Unlock Full Guide" product in Google Play Console first
   (Console → Monetize → Products → In-app products), then link it in
   RevenueCat's dashboard
3. Install the Capacitor plugin: `npm install @revenuecat/purchases-capacitor`
4. Replace the `unlockBtn` click handler in `book.html` with a real
   call to `Purchases.purchasePackage(...)`, and call `setUnlocked()`
   only inside the success callback
5. Replace the `restoreBtn` handler similarly using
   `Purchases.restorePurchases()`

**Option B — Native Google Play Billing directly**
More setup, no RevenueCat dependency, but you handle receipt
validation yourself. Capacitor community plugin:
`@capacitor-community/in-app-purchases` (or similar — check current
maintenance status before using). More involved; Option A is
recommended unless you have a specific reason to avoid a third-party
billing layer.

Either way, this needs to be wired up and tested with a real Google
Play Console product **before** submitting the app — Google will
reject apps that reference purchases that don't actually work.


---

## Costs Summary
- Google Play Console: $25 one-time (no renewal, ever)
- Codemagic: free tier covers occasional builds
- GitHub: free
- GitHub Pages (privacy policy hosting): free

## Reusing With the iOS Project
If you set up both stores, you can actually combine these into a single
GitHub repo and a single Codemagic project with two build targets (iOS +
Android) pointed at the same www/index.html — no need to maintain two
separate copies of the calculator itself. This separate folder structure
is just to keep the Google Play-specific assets (feature graphic, Play
listing copy) cleanly organized on their own.
