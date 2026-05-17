# CGEN App — Project State File

**Last updated:** May 17, 2026
**Updated by:** Claude (AI & I Build Session)
**Version:** 1.1.0 — Post Audio/PDF Launch

---

## PROJECT OVERVIEW

- **App name:** CGEN
- **Slug:** cgen-app
- **Platform:** React Native / Expo Router
- **Build system:** EAS Build
- **Backend:** FastAPI / Python on Render.com
- **WordPress:** c93n.com (Hostinger, Astra Pro)
- **API:** Anthropic Claude Sonnet via main.py + OpenAI TTS for audio
- **Current APK:** https://expo.dev/artifacts/eas/45ZfNMCce74LepgEo6EGJQ.apk
- **Status:** Soft launched — APK live, iOS submitted (May 14, awaiting Apple review), Google Play pending address verification (bank statement from Mizrahi Tefahot requested)

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
    report.tsx        — Single report reader screen (updated May 17)
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
- **Premium modal:** Dark CGEN design, lemon-green accents, explains Premium, Register button → login screen, "Maybe later" dismiss
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

### report.tsx (Report Reader) — UPDATED May 17

- **Loads:** Single post via /wp-json/cgen/v1/my-post/{id}
- **Displays:** Type badge, title, date, full content (HTML stripped to plain text)
- **Make Public button:** Outlined green (not filled) — shows confirmation Alert before publishing
- **Publishing state:** ActivityIndicator inside button + status message below: "Generating audio briefing & PDF — this takes up to 30 seconds..."
- **Published:** Shows "✓ Published to CGEN Archives" confirmation
- **NOTE:** Make Public button styling (filled lime-green) still needs fix — pending next build

---

## BACKEND — main.py (Render.com) — UPDATED May 17

### Environment Variables (Render)

- ANTHROPIC_API_KEY
- OPENAI_API_KEY (added May 17 — for TTS audio)
- WORDPRESS_URL
- WORDPRESS_USER
- WORDPRESS_PASSWORD

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

### Dependencies (requirements.txt)

- fastapi
- uvicorn
- anthropic
- pydantic
- requests
- openai (added May 17)
- reportlab (added May 17)

### Endpoints

- POST /generate-concept
- POST /generate-landing
- POST /generate-intelligence (now includes duplicate detection)
- POST /generate-validator
- POST /make-public (now generates audio + PDF at publish time)
- POST /poll/vote
- GET /poll/results/{poll_id}

### NEW — Duplicate Detection (Intelligence Engine)

- Runs before every intelligence report generation
- Checks published posts only (drafts ignored entirely)
- Matches business_name against post titles + slugs
- If match found → returns duplicate: true + post_url, does NOT run the engine
- Frontend needs update to handle duplicate response and show link to existing report
- NOTE: Frontend HTML not yet updated for this — pending next web session

### NEW — Audio Briefing (OpenAI TTS)

- Voice: onyx (deep male)
- Model: tts-1
- Input: plain text report, prefixed with "CGEN Intelligence Briefing. [Title]."
- Current limit: 4000 characters (truncates) — known issue, full report cutoff mid-way
- TODO: implement chunking approach to read full report (3 chunks, ~$0.15/report)
- Triggered: at publish time for visitor flow AND Make Public (member flow)
- Upload: MP3 → WordPress media library, URL appended to post content as "▶ Listen to Briefing" button

### NEW — PDF Briefing (ReportLab)

- Style: dark #0a0a0a background, lemon-green #c8f55a accents
- Footer: "Generated by CGEN · Concept General · c93n.com · [year]"
- Sections: all report sections formatted with headers and body text
- Triggered: same as audio — at publish time
- Upload: PDF → WordPress media library, URL appended to post content as "↓ Download PDF" button

### make-public

- Normalizes report_type (strip, lowercase)
- Logs report_type for debugging
- Assigns correct category per type
- Generates slug with subject name
- NOW ALSO: generates audio + PDF at publish time, appends media buttons to post content

---

## WORDPRESS — functions.php

### Active features

- Admin paywall bypass (cgenAdminBypass)
- cgData nonce injection (user_id, display_name, nonce, is_premium)
- Login/register redirect → /account
- Hide admin bar for non-admins
- REST API: members can read own drafts
- _cgen_type registered as REST-exposed meta field
- cgen_premium role registered
- PayPal webhook → assigns cgen_premium role on subscription
- Auto-claim pending Premium on registration
- Custom endpoints:
  - GET /cgen/v1/my-posts
  - GET /cgen/v1/my-post/{id} — returns type field from _cgen_type meta
  - POST /cgen/v1/get-user-id
  - POST /cgen/v1/rate-engine
  - GET /cgen/v1/check-premium
  - POST /cgen/v1/paypal-webhook
  - POST /cgen/v1/claim-premium
- cgen_ratings table (id, rating, engine, user_id, created_at)

---

## WEB PLATFORM — Custom HTML Blocks

### Intelligence Engine page (updated May 17)

- Rating gate text: "Rate your last experience to unlock the button below"
- After rating: "Thank you · Now hit the button below to run your report."
- Rating gate text color: rgba(255,255,255,0.6) — clearly readable
- On reset: text resets back to instruction
- Applies to both Mode A and Mode B

### Concept Generator page (updated May 17)

- Same rating gate text and color fixes as Intelligence Engine
- On restart: rating message resets correctly

### Read page — /read (updated May 17)

- Confirmation modal before Make Public: "Make This Report Public? This will publish your report to the CGEN Archives and make it visible to everyone. This action cannot be undone."
- Publishing status bar: spinner + "Publishing your report — generating audio briefing & PDF. This takes up to 30 seconds, please don't close this page."
- On success: status changes to "✓ Report published — redirecting to Archives..."
- report_type now read from data.type (API response) — NOT guessed from title
- Category bug fixed: Intelligence reports now land in Market Research correctly

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
- BUG: Star gate being skipped on incognito (non-first generation) — suspected counter reset issue, pending investigation

---

## PAYWALL LOGIC (APP)

- Visitor (no token): 1 free per engine → PaywallModal
- Member (has token): 3 free per engine per month → PaywallModal
- PaywallModal: Register free OR Pay $6 via PayPal web redirect
- Counters stored in AsyncStorage per engine

---

## LINKEDIN POSTS PUBLISHED

- WIP.co — IR-0034
- Monday.com — IR-0035
- Sticklight — IR-0038 (Elementor AI tool)
- Base44 — IR-0039 (published May 17, 2026)
- Wiz — IR-0041 (published May 17, 2026)

---

## PENDING / ROADMAP

### Bugs pending

- [ ] Star gate skipped on incognito non-first generation — suspected counter reset trigger, investigate
- [ ] Make Public button in report.tsx still lime-green filled (should be outlined green) — pending next build
- [ ] No "Return to My Lab" button on first generated report in app — missing on first generation only
- [ ] Audio TTS cuts off mid-report — current 4000 char limit too short, implement chunking (3 chunks, full read)
- [ ] Duplicate detection frontend — Intelligence Engine HTML needs update to handle duplicate:true response and show link to existing report
- [ ] Google Play phone verification loop — awaiting Google support / address verification

### Features pending

- [ ] Audio chunking — split report into 3 chunks, merge MP3s for full report read (~$0.15/report)
- [ ] Blind Spot Auditor — separate lightweight product on c93n.com infrastructure, domain TBD, built after CGEN complete
- [ ] Secret VIP invitation codes — special AUTH_KEY codes for free Pro access
- [ ] Monthly generation reset for members (currently counters never reset)
- [ ] Premium/paywall system — full check in app for cgen_premium role

### Planned — Next main.py upgrade (Audio + PDF full session)

- [ ] Full TTS chunking for complete report audio
- [ ] Summary mode option (2-minute briefing vs full read)

---

## CERTIFIED CONCEPTS

- CGEN-2026-0001: Pro.M
- CGEN-2026-0002: TrendTracker
- CGEN-2026-0003: The Blind Spot (AI product completeness auditor)

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
- iOS: Submitted to App Store May 14, 2026 — awaiting Apple review
- Google Play: Pending address verification (bank statement requested)
