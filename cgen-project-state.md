# CGEN App — Project State File
**Last updated:** May 6, 2026  
**Updated by:** Claude (AI & I Build Session)  
**Version:** 1.0.0 (1) — Preview Build

---

## PROJECT OVERVIEW
- **App name:** CGEN
- **Slug:** cgen-app
- **Platform:** React Native / Expo Router
- **Build system:** EAS Build
- **Backend:** FastAPI / Python on Render.com
- **WordPress:** c93n.com (Hostinger, Astra Pro)
- **API:** Anthropic Claude Sonnet via main.py
- **Current APK:** https://expo.dev/artifacts/eas/45ZfNMCce74LepgEo6EGJQ.apk
- **Status:** Soft launched — APK live on c93n.com homepage

---

## FILE STRUCTURE
```
app/
  (tabs)/
    _layout.tsx
    index.tsx         — Home screen (visitor + My Lab)
    intelligence.tsx  — Intelligence Engine screen
    validator.tsx     — Idea Validator screen
    concepts.tsx      — Concept Generator screen (Premium gated)
    login.tsx         — Login / Register screen
    report.tsx        — Single report reader screen
    modal.tsx         — Paywall modal (replaces default)
  _layout.tsx
assets/
components/
constants/
hooks/
scripts/
app.json
eas.json
package.json
```

---

## SCREENS — CURRENT STATE

### index.tsx (Home)
- **Visitor view:** CGEN eyebrow, "Know Before You Build" headline (28px, one line), subtitle, LOGIN·REGISTER button, EXPLORE CGEN → (links to c93n.com), ENGINE GUIDE → (links to c93n.com/directories)
- **Three engine cards:** Intelligence, Validator, Concept Generator
- **Concept Generator card:** Premium gated — tapping opens Premium modal, PREMIUM badge centered below "Discover Opportunities"
- **Premium modal:** Dark CGEN design, explains Premium, Register button → login screen, "Maybe later" dismiss
- **Green dividers:** Above Saved Intel section, above Sign Out/Support buttons
- **My Lab view (logged in):** MY LAB title, Know Before You Build subtitle, ↓ Saved Intel scroll link, engine cards, Saved Intel section with reports, Sign Out + Support buttons
- **Visitor notice:** "All generated Intel will be automatically saved..." shown to visitors
- **Footer:** "Powered by c93n.com"

### intelligence.tsx (Intelligence Engine)
- **Fields:** Business name (required), Industry (required), Focus (optional)
- **Terms checkbox:** Required before submit
- **Archive notice:** Members private, Visitors public
- **Paywall logic:** Visitor limit = 1, Member limit = 3 (cgen_intel_count)
- **Rating gate:** Global — checks all 3 counters, fires if total >= 1
- **Loading steps:** 5 steps, 4000ms interval
- **Delay message:** "The engine is scanning the web for live data. Deep searches may take up to 2 minutes."
- **Result display:** Report badge, report number, full text, Reset button, privacy note
- **PaywallModal:** Imported from ../modal

### validator.tsx (Idea Validator)
- **Fields:** Idea description (required, textarea), Industry (required), Audience (optional)
- **Terms checkbox:** Required
- **Paywall logic:** Visitor limit = 1, Member limit = 3 (cgen_valid_count)
- **Rating gate:** Global — checks all 3 counters
- **Loading steps:** 5 steps, 4000ms interval
- **Delay message:** Same as intelligence
- **PaywallModal:** Imported from ../modal

### concepts.tsx (Concept Generator)
- **Status:** Accessible to logged-in users but gated at home screen level by Premium modal
- **Fields:** 4 inputs (Industry, Target Segment, Core Frustration, Desired Outcome)
- **Flow:** Form → 3 concept cards → Full landing page
- **Paywall logic:** Visitor limit = 1, Member limit = 3 (cgen_concept_count)
- **Rating gate:** Global — checks all 3 counters
- **Loading steps:** 6 steps, 3500ms interval
- **Delay message:** Same pattern

### modal.tsx (Paywall Modal)
- **Replaces:** Default Expo template
- **Design:** Dark CGEN, lemon-green accents
- **Options:** Register free (→ login screen) | Pay $6 (→ PayPal URL in browser)
- **PayPal URL:** https://www.paypal.com/ncp/payment/LAUXW9YHQFCP4
- **Props:** visible (bool), onClose (function)

### login.tsx (Login / Register)
- **Modes:** Login / Register toggle
- **Login:** Email + Password → JWT via Simple JWT Login plugin
- **Register:** Email + Password + Auth Code (CGEN2026)
- **After login:** Redirects to /(tabs), stores cgen_token, cgen_user_id, cgen_email
- **Visitor option:** "Continue as visitor" button

### report.tsx (Report Reader)
- **Loads:** Single post via /wp-json/cgen/v1/my-post/{id}
- **Displays:** Type badge, title, date, full content (HTML stripped to plain text)
- **Actions:** Make Public button (draft only) → calls /make-public on backend
- **Published:** Shows "Published to CGEN Archives" confirmation

---

## BACKEND — main.py (Render.com)

### Counters
- concept_counter: starts at 16
- intelligence_counter: starts at 1
- validator_counter: starts at 2
- Loaded from WordPress on startup via load_counters_from_wordpress()
- HTML entity blocklist: {8216, 8217, 8218, 8220, 8221, 8222, 8211, 8212, 8230}

### Category IDs (WordPress)
- CAT_INTELLIGENCE = 19 (Market Research)
- CAT_VALIDATOR = 20 (Idea Validation)
- CAT_CONCEPT = 1 (Concept Draft)

### Endpoints
- POST /generate-concept
- POST /generate-landing
- POST /generate-intelligence
- POST /generate-validator
- POST /make-public
- POST /poll/vote
- GET /poll/results/{poll_id}

### make-public
- Normalizes report_type (strip, lowercase)
- Logs report_type for debugging
- Assigns correct category per type
- Generates slug with subject name

---

## WORDPRESS — functions.php

### Active features
- Admin paywall bypass (cgenAdminBypass)
- cgData nonce injection (user_id, display_name, nonce)
- Login/register redirect → /account
- Hide admin bar for non-admins
- REST API: members can read own drafts
- _cgen_type registered as REST-exposed meta field
- Custom endpoints: /cgen/v1/my-posts, /cgen/v1/my-post/{id}, /cgen/v1/get-user-id, /cgen/v1/rate-engine
- cgen_ratings table (id, rating, engine, user_id, created_at)

---

## ASYNC STORAGE KEYS
- cgen_token — JWT login token
- cgen_user_id — WordPress user ID
- cgen_email — user email
- cgen_intel_count — Intelligence Engine generation count
- cgen_valid_count — Validator generation count
- cgen_concept_count — Concept Generator generation count

---

## RATING GATE LOGIC
- Triggers globally after 1st generation on ANY engine
- Reads all 3 counters, adds them: if total >= 1 → show gate
- Gate shows before running (not after)
- Rating submitted to /wp-json/cgen/v1/rate-engine
- After submission → runs the engine

---

## PAYWALL LOGIC (APP)
- Visitor (no token): 1 free per engine → PaywallModal
- Member (has token): 3 free per engine per month → PaywallModal
- PaywallModal: Register free OR Pay $6 via PayPal web redirect
- Counters stored in AsyncStorage per engine

---

## PENDING / ROADMAP

### Bugs pending
- [ ] Google Play phone verification loop — awaiting Google support resolution
- [ ] My Lab box titles showing "CONCEPT" on mobile — debug /wp-json/cgen/v1/my-posts type field

### Features pending
- [ ] Audio briefing — OpenAI TTS → MP3 uploaded to WP media, attached to post, Listen button via functions.php (~$0.06/report)
- [ ] PDF download — generated at publish time, attached to post, Download button
- [ ] Build both Audio + PDF together in one session (both at publish time in main.py)
- [ ] Premium/paywall system — WordPress user roles, payment integration, app-side logic
- [ ] Concept Generator Premium gate — needs Premium role check in main.py + WordPress
- [ ] Secret VIP invitation codes — special AUTH_KEY codes for free Pro access
- [ ] Monthly generation reset for members (currently counters never reset)
- [ ] Google Play submission — pending identity verification approval
- [ ] App Store (iOS) — after Google Play is stable

### Certified Concepts
- CGEN-2026-0001: Pro.M
- CGEN-2026-0002: TrendTracker
- CGEN-2026-0003: The Blind Spot (AI product completeness auditor)

### LinkedIn posts published
- WIP.co — IR-0034
- Monday.com — IR-0035

---

## DESIGN TOKENS
- Background: #0a0a0a
- Accent: #c8f55a (lemon-green)
- Border dark: #1e1e1e / #2a2a2a
- Text primary: #ffffff
- Text muted: #999999 / #555555
- Font: Syne (headings), DM Sans (body), DM Mono (labels)
- Border radius: 4px (sharp), 8px (medium), 12px (cards)

---

## DEPLOYMENT
- Backend: Render.com (cgen-backend-udg1.onrender.com) — Pro plan, no cold starts
- WordPress: Hostinger
- App builds: EAS Build (expo.dev, infowolf account, cgen-app project)
- EAS profile: preview (Android APK)

- Updates to add to cgen-project-state.md:
SCREENS:

index.tsx — publishBtn style fixed: now outlined green (transparent background, #c8f55a border and text) to match readBtn visual weight. Was previously filled lemon-green.

DEPLOYMENT:

iOS App Store submission COMPLETE — build submitted via EAS CLI, all metadata, screenshots, privacy, tax, DSA compliance done. Submitted for Apple review May 14 2026. Bundle ID: com.cgen.app.

PENDING — update:

Google Play — still pending address verification. Bank statement requested from Mizrahi Tefahot, awaiting email. Once received, upload to Play Console and resubmit.
App Store (iOS) — remove from pending, mark complete.
NOTE TO CLAUDE — Next Session Suggestion (30 min build):
Build a small script that automatically pushes end-of-session state updates to this GitHub file via the GitHub API. The goal: at the end of every build session, Claude generates a structured update and the script commits it directly to cgen-project-state.md without manual copy-paste. This would make the file a live external memory that updates itself. Build this as a standalone tool — one input (the update text), one output (committed to GitHub). Priority: low, but high value long-term.
