CGEN Project State — June 18, 2026
BUILD SESSION COMPLETE
iOS — Build 14 (1.1.0) submitted to Apple, Waiting for Review
Three rejection fixes included: 2.1b (Upgrade Plan visible in "+" menu), 2.3.2 (unique promo images done prior), 3.1.2c (EULA links done prior). Expecting approval in 1-3 days.
Note: iOS build number jumped to 14 (not 9) due to failed authentication attempts consuming EAS builds. Version number remains 1.1.0. Apple only requires build number to increment — 14 is valid.
Android — Build 8 (1.1.0) live on closed testing track
12 Bitrupt testers opted in on correct track. 14-day clock started June 18. Daily activity confirmed — 10 reports from 6 testers on day 1. Do NOT push new Android build until 14 days complete (around July 2).
EAS builds used this month: 11 of 30 (6 Android, 5 iOS). 19 remaining.
Apple 2FA note: SMS to Israeli number unreliable. Use real Apple ID password + wait for device push notification or Android SMS when prompted. App-specific password is NOT what EAS asks for at login — that's for a later step.
PENDING — next session
FAQ page on c93n.com (currently 404)
Delete draft from My Lab
index.tsx Concept Generator modal — update text to reflect new tiers
Backend fixes: missing post title + partial dedup bug
Google Play Billing — wire RevenueCat Android IAP after production access granted
Audio + PDF Briefing (future)

CGEN Project State — June 17, 2026 (Evening)
SESSION COMPLETE — Build 9 ready, not yet submitted
DONE THIS SESSION
"+" Menu — both screens (mylab.tsx + index.tsx)

Replaced all bottom linked buttons with a single lemon-green "+" bar
Tapping "+" opens CGEN-styled bottom sheet: Sign Out, FAQ, Upgrade (green), Browse CGEN, Support (green), divider, Delete Account (red)
Visitor screen menu: FAQ, Upgrade (→ login), Browse CGEN, Support — no Sign Out or Delete
Sign Out and Delete Account both have Alert confirmations
Menu animationType="none" (prevents animation crash on return from browser)
Both "+" bars are inline (not floating) — avoids tab bar conflicts and positioning issues

Pro tier recognition (_layout.tsx + login.tsx)

/check-pro endpoint called on every launch (refreshTierStatus) and on login/register
Tier priority: premium > pro > free
Manual WordPress role cgen_pro → app reads it on next launch, sets cgen_tier = 'pro'
Founders and manual grants now recognized by the app automatically

Upgrade modal — iOS RevenueCat IAP (mylab.tsx)

Tapping Upgrade in "+" menu calls Purchases.getOfferings() → shows native plan picker
Packages displayed dynamically from RevenueCat (title, price, /mo /yr suffix)
On purchase: sets cgen_tier + cgen_is_premium in AsyncStorage
Android fallback: links to c93n.com/upgrade
Satisfies Apple 2.1b rejection (visible IAP entry point)

Apple 2.1b — also fixed:

Visible Upgrade in My Lab "+" menu (logged-in path)
Visible Upgrade in Visitor "+" menu (visitor path → login → IAP)

UI changes

My Lab subtitle: "Know Before You Build" → "Business Intelligence"
Visitor screen: removed "EXPLORE CGEN" button, "ENGINE GUIDE →" now full-width standalone
useEffect replaces useFocusEffect in mylab.tsx (loads once on mount, not on every focus return)

Pagination — My Lab Saved Intel

WordPress /my-posts endpoint updated: accepts page param, returns { reports, has_more, page }
20 reports per page, fetches 21 to detect has_more (no extra DB query)
"LOAD MORE" button appears when has_more: true
Appends next page to existing list, button disappears on last page

NEXT BUILD QUEUE (build 9 → iOS resubmission)

Build EAS iOS build 9
Resubmit to Apple with three fixes: 2.3.2 (unique promo images ✓ done prior), 3.1.2(c) (EULA links ✓ done prior), 2.1b (Upgrade visible ✓ done this session)
Android: closed test restarting on correct track — monitor opted-in count in Play Console, clock starts at 12+

PENDING (next sessions)

FAQ page — build on c93n.com (WordPress), FAQ menu item links to it (currently 404)
Delete draft from My Lab (mylab.tsx + backend endpoint) — confirmation Alert, Archives untouched
report.tsx title-decode fix — already saved locally (stripHtml on report.title)
index.tsx — align Concept Generator modal with new tiers
WordPress Pro role endpoint already done and tested (June 17 morning session)
Backend fixes (main.py): missing post title bug, partial-report dedup bug
Google Play Billing for Android (future build — not blocking)
Audio + PDF Briefing (future build)

FILES CHANGED THIS SESSION

mylab.tsx — complete rewrite
index.tsx — complete rewrite
_layout.tsx — Pro check added to refreshTierStatus
login.tsx — Pro check added to handleLogin + handleRegister
WordPress "CGEN Custom Functions" snippet — /my-posts pagination added

Future Architecture — New Pricing & Tier Model (proposed June 23, 2026)
Replaces: Current Visitor/Subscriber/Pro/Premium four-tier system with daily/monthly generation resets.
Core principle: If you don't use it, you don't pay. What you paid for never expires.

VISITOR — unchanged

1 free report on Intelligence Engine
1 free report on Idea Validator
No Concept Generator access
Reports published to public Archives automatically


SUBSCRIBER — registered, always free

My Lab private workspace
All 3 engines including Concept Generator (registration gift)
3 one-time welcome reports on registration (any engine, one-time only — not monthly)
After welcome reports: $1.50 per report, any engine, pay as you go
No subscription, no monthly reset, no expiry
Natural upgrade pressure: after 5 individual purchases ($7.50 spent) the bundle becomes obviously better value


PRO — bundle buyer

Everything Subscriber has
Pre-paid report bundles (consumable IAP):

11 reports = $15 ($1.36/report)
22 reports = $30 ($1.36/report)
55 reports = $75 ($1.36/report)


Credits never expire
No earn mechanic, no token counting — pure credit inventory


Why this works economically:

Zero recurring free API cost after welcome reports
3 welcome reports = $1.20 one-time acquisition cost per user (cheaper than any ad)
Every report after welcome is pre-funded
Free tier eliminated entirely — no monthly $3,600 liability at scale
Pro bundle margin: $15 bundle costs $4.40 API = $10.60 margin per bundle


What this eliminates:

cgen_premium role and all Premium logic
Monthly/daily period-stamped counters in all three engine files
The AsyncStorage tier keys beyond basic logged-in status
RevenueCat subscription products (Monthly/Annual) — replaced by consumable bundles
All 5 legal docs need updating to reflect new model
WordPress Pro/Premium roles simplified to credit balance system


Build scope — this is a major architectural rebuild:

WordPress: new credit balance meta field per user, welcome report tracking field, deduct on generation, check balance before allowing generation
main.py: check credit balance before generating, deduct after success, return balance to app
All 3 engine files: replace period counter logic with credit balance check
New RevenueCat consumable IAP products (11/22/55 bundles + single $1.50)
mylab.tsx: show credit balance somewhere visible
Pricing page c93n.com/upgrade: full redesign
All 5 legal docs updated
App Store Connect: new IAP products submitted for review

Timing: After Android 14-day test completes (~July 2) and production access applied for. Dedicated build session, not bundled with quick fixes.
Status: Architecture confirmed, no code written yet.

Note for next pricing rebuild session:
Internal code value 'free' in getTier() and all AsyncStorage tier logic refers to what is publicly called Subscriber in the UI and documentation. These are the same thing. When rewriting the tier/credit system, 'free' stays as the code value — do NOT rename it to 'subscriber' in code. The word "Subscriber" appears only in user-facing text, FAQ, and marketing copy. Any session that sees tier === 'free' in the code should read it as "this user is a Subscriber who has not paid yet." Never confuse the two or introduce 'subscriber' as a code value — it will break the gate logic across all three engine files.

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

- --

Store Status
iOS App Store

Build 1.0.1 (7) submitted — under review
Rejected 3 times on Guideline 3.1.1 (payments)
Promo code field removed, upgrade links removed, member wall updated to "OK GOT IT"
Decision made to implement Apple In-App Purchase (IAP) as the permanent fix before next submission
RevenueCat identified as the implementation path

Google Play

AAB Build 1.0.1 (7) live on internal testing track
18 testers (Bitrupt team) — Day 8 of 14
Active days confirmed: ~5-6 out of 14 needed
Bitrupt paid $125 upfront, $125 + tip on completion
Communication via Fiverr + Slack


App — Current State (Build 7)
login.tsx

Email + password registration — no promo code field
Display Name field added (optional) — saves to WordPress display_name on registration
Email validation added
KeyboardAvoidingView behavior='height' for Android
Registration JWT fix — checks both jwt and userId as success signals
AUTH_KEY hardcoded to 'CGEN2026' silently

intelligence.tsx, validator.tsx, concepts.tsx

KeyboardAvoidingView wrapping with behavior='height' for Android
Terms & Conditions tappable link to c93n.com/terms-and-conditions
Field hint text added below Industry and Target Audience in validator
Member Limit Wall — "OK, GOT IT" button, no upgrade link, no Premium reference
MEMBER_LIMIT set to 3 for iOS build

mylab.tsx

Duplicate publish alert — shows Alert if backend returns duplicate:true
Instant UI publish update on Make Public confirmation

report.tsx

Duplicate publish alert added
MAKE PUBLIC outlined green style

index.tsx

Green dividers, visitor privacy box, empty state with lemon-green CTA
Sign Out grey/muted below Support
Members Only modal for Concept Generator (register free flow)


Backend (main.py) — Current State

All 3 engines live with web search, PDF generation, SVG thumbnails
Active request lock on all 3 engines (prevents duplicate generation)
max_tokens reduced to 4000 on Intelligence and Validator engines
No-repetition instruction added to Intelligence and Validator prompts
INPUT_INVALID protection for visitor path on all 3 engines — blocks publishing gibberish to Archives
Duplicate check at generation time — Intelligence Engine only
Duplicate check at publish time — all 3 engines
Author display name fetched from WordPress and appended to published report content as "Published by [Name]" in lemon-green style
Audio disconnected (501 error) — functions kept dormant for future reactivation
Counter loading from WordPress on startup


functions.php (Snippets) — Current State

SVG upload enabled
Featured image saves correctly on new posts
All custom endpoints live: my-posts, my-post, get-user-id, rate-engine, check-premium, paypal-webhook, claim-premium, upload-media, create-post, update-post, delete-account, request-delete
cgen_ratings table created and active
cgen_premium role registered
ratings-summary endpoint added (for homepage widget)


WordPress Site

Homepage: "AI Business Intelligence Platform" headline
Star rating widget live on homepage — shows live total from database
357+ star ratings accumulated during test period
20+ registered members
104+ published reports in Archives
Upgrade page updated — PayPal completely removed, new pricing displayed:

Pro: $24.99 one-time
Premium: $6/month or $60/year
Coming Soon messaging — no payment processing active


Pending — Post-Test Fix List

Apple IAP implementation — RevenueCat, StoreKit, subscription products in App Store Connect. Required before paid tiers go live on iOS. This is the next major session.
Duplicate publish alert — backend blocks duplicate publishing, app shows Alert for duplicate:true response in mylab.tsx and report.tsx ✅ (added build 7)
Expired token UX — no message shown when token expires
Monthly counter reset — AsyncStorage counters never reset
Audio briefing reconnection — TTS removed due to 501 error, reconnect after launch
PDF download monetization — paywall before download button ($1 per download)
HTML entity decoder — for Read screen title/content display in app
Draft duplicate detection — same user running same report twice creates duplicate drafts. Plan: detect existing draft with same title/user, append new content as "ADDITIONAL INTELLIGENCE" section instead of creating new draft


Community / Membership Features

Display Name collected at registration — shown on published reports as "Published by [Name]"
Founding Member badge — planned digital badge for first 100 members (design + email campaign pending)
First 100 registered members get Premium free


Pricing Strategy

Phase 1: First 100 members — free Premium access
Phase 2: Pro $24.99 one-time, Premium $6/month or $60/year — until 500-1000 users
Phase 3: Raise Premium to $15.99/month after 500-1000 users
PDF download monetization: $1 per download from Archives (post-launch)


AI & I Book

Chapter 1, 2, 3, 4 published on ZenGate ✅
Chapter 5 "The First Fruit" — written, covers 293-rating discovery, QA report, 11 bug fixes ✅
Chapter 6 — planned for launch period
Total planned: 6 chapters


LinkedIn Activity

Slack post — 565+ impressions, strong engagement
Architects/CGEN post — 580+ impressions
HeyGen post — published, visual split composition
All posts reference c93n.com/archives and app launch


Key Working Preferences

Non-technical explanations, no jargon
Surgical edits to specific code blocks preferred
Show exact block to delete before showing replacement
Never rewrite large files without confirmation first
Full function rewrites required for structural Python changes
Plain markdown format for session-end summaries
GitHub state file updated manually by David


# CGEN Project State — June 12, 2026

## SESSION SUMMARY — Apple IAP Build (the big one)

iOS 1.1.0 (8) SUBMITTED to App Review — full In-App Purchase implementation via RevenueCat, answering all three 3.1.1 rejections. Apple business side 100% complete (agreement + bank + tax forms Active).

## Store Status

### iOS App Store
- **1.1.0 (8) submitted June 12 — Waiting for Review**
- Contains full IAP implementation + new tier system
- All 3 IAP products attached to the version review: CGEN Pro, Premium Monthly, Premium Annual
- Review notes explain the 3.1.1 resolution; demo credentials provided
- Paid Apps Agreement: bank account connected (Mizrahi Tefahot), W-8BEN Active, Certificate of Foreign Status Active

### Google Play
- Bitrupt testing ongoing — keep 12 testers active daily until day 14
- **DO NOT rebuild Android until Bitrupt finishes** — new tier limits would disrupt testers
- Reminder message sent re: daily active tester count (12×1 beats 10×3)

## RevenueCat — Fully Configured
- Account created, email verified, project "CGEN - Concept General"
- App Store app connected: bundle com.cgen.app, In-App Purchase Key (.p8) uploaded with Key ID + Issuer ID
- Products: com.cgen.app.pro (non-consumable), com.cgen.app.premium.monthly, com.cgen.app.premium.annual
- Entitlements: `pro` (1 product), `premium` (2 products)
- Offering `default`: Lifetime/Monthly/Annual packages mapped to real App Store products (Test Store remnants harmless)
- Public API key (iOS): appl_QhBYffCpsXeRhtMyWzJvUJfOkFg
- Skipped (optional, later): App Store Connect API key, Apple Server Notification URL

## NEW — CGEN Access Tiers (FINAL SPEC)
- **Visitor** (no account): 1 generation per engine on Intelligence + Validator. Concept Generator locked.
- **Registered unpaid**: 1 additional generation per engine on the 2 engines (separate counter), private Lab. Then wall offers Pro/Premium.
- **Pro** ($24.99 one-time): 3 generations per month per engine on ALL 3 engines (up to 9/month).
- **Premium** ($6.99/mo or $69.99/yr): 3 generations per day per engine on all 3 engines (up to 9/day).
- First 100 registered: auto-granted Premium server-side (existing WordPress auto-claim).
- Limits are exact (3 = 3): per-tier separate counters eliminated the old shared-counter off-by-one.
- Counters bump only on successful generation; errors don't consume.

## App Code — Build 8 Changes
- **package**: react-native-purchases installed (requires EAS build — Expo Go won't work)
- **_layout.tsx**: Purchases.configure on startup (iOS only) + refreshTierStatus() on every app launch — checks WordPress /check-premium (source of truth for Premium incl. manual grants) AND RevenueCat getCustomerInfo (source of truth for purchases). Fixes the "manual Premium needs re-login" problem.
- **intelligence.tsx / validator.tsx / concepts.tsx**: full tier system rewrite
  - Counter keys per engine: visitor (cgen_intel_count etc.), free member (cgen_m_intel_count etc.), Pro month-stamped (cgen_pro_intel etc.), Premium day-stamped (cgen_prem_intel etc.)
  - Period-stamped counters self-reset (format "period|count") — kills the never-resetting counter bug for paid tiers
  - Purchase wall modes: upgrade (all 3 products), pro-limit (subscriptions only), premium-limit (no products, "refreshes tomorrow"), locked (concepts, for visitor/free)
  - Live Apple prices via offerings; Restore Purchases button; Terms+Privacy links in wall
  - Android always falls back to "OK GOT IT" wall — Play build behavior unchanged
- **login.tsx**: sets cgen_tier on login/register from both WordPress check-premium and RevenueCat entitlements
- **app.json**: version 1.1.0, buildNumber 8
- AsyncStorage new keys: cgen_tier, cgen_m_*_count, cgen_pro_*, cgen_prem_* (per engine)

## App Store Connect — Product Metadata
- All 3 product descriptions rewritten: consistent "Private Lab" naming, engine names (Market Research, Idea Validator, Concept Generator), exact numbers matching code, NO "unlimited", NO "triggered" language
- Subscription group localized as "CGEN Premium" (was the hidden Missing Metadata blocker)
- Review screenshots uploaded per subscription (iPad size)
- App description updated: Concept Generator no longer tagged "(Premium)", MEMBERSHIPS section added
- Promotional text updated with memberships mention

## Pricing Strategy (updated)
- Pro $24.99 one-time: 3/month per engine, all 3 engines
- Premium $6.99/month or $69.99/year: 3/day per engine, all 3 engines
- First 100 members: free Premium
- (replaces old $6/$60 PayPal-era pricing)

## Pending — Next Sessions
- [ ] Await Apple review verdict on 1.1.0 (8)
- [ ] index.tsx: align home-screen Concept Generator modal with new tiers (registration alone no longer unlocks it)
- [ ] Backend awareness of tiers (currently counters are app-side only — reinstall resets them; server-side enforcement later)
- [ ] Apple Server Notification URL → paste into App Store Connect (instant renewal/cancellation sync)
- [ ] Android build with tier system — ONLY after Bitrupt completes
- [ ] Expired token UX, HTML entity decoder, audio reconnection, draft duplicate detection (carried forward)
- [ ] AI & I Chapter 6 — this session is the obvious material
      
# CGEN Project State — June 13, 2026

## Store Status
- **Apple App Store:** Build 1.1.0 (8) submitted, in review. Full IAP (RevenueCat), tier system, bank + tax forms Active.
- **Google Play:** Bitrupt closed test, day 10 of 14 (started June 2, two Sundays off). 4 days left, then apply for production access. Android still on build 7 — rebuild to 8 AFTER test completes, together with Google Play billing connection.
- 522 star ratings (day 10) — passed the 500 prediction 4 days early.

## DECISION — Founding member reward changed (important)
- **First 100 members now get free PRO, not free Premium.** Reason: free Premium (3/day, all engines) could cost up to ~$19/user/month in API — 100 of them could sink CGEN pre-revenue. Free Pro (3/month per engine, 9/month total) caps founder cost to ~$1–2/user/month. Survivable, still a real lifetime gift.
- Coworkers confirmed 9 generations/month is strong value for real work.

## NEXT BUILD QUEUE (batch before building 8-9 iOS)
Hold all of these for ONE build to avoid wasted builds:
1. **Create WordPress `Pro` role** (like existing CGEN Premium role). David assigns manually: Subscriber → Pro, for the 100 founders and any edge cases. Same as how testers were switched to Premium.
2. **Backend + app must learn to read Pro from WordPress** — currently the app can ask /check-premium for Premium, but has NO way to hear "this user is Pro" from the site. Needs: WordPress endpoint returns tier (premium/pro/free), and app (_layout.tsx + login.tsx) sets cgen_tier from it. Without this, manual Pro switches won't be seen by the app. (Same class as the manual-Premium-refresh fix already done.)
3. **report.tsx title-decode fix** — already saved locally, rides in next build.
4. Subscriber = registered, didn't pay. Gets the small free taste, then hits the wall.
Pro = paid (or one of the 100 founders you manually switch). Gets 3/month per engine.
5. index.tsx Concept Generator modal — align with new tiers (carried from June 12).

## Cost / Billing notes
- June API spend: ~$83 of $500 limit at day 13 (~$6.40/day avg during testing surge). Healthy.
- Daily "$10 due" charges were AUTO-RELOAD top-ups, not daily spend — not a problem.
- Token volume 13.3M last 7 days (+61%) = Bitrupt testing surge, expected.
- Post-launch: monitor real (non-tester) per-Premium-user cost. If heavy users exceed $6.99 revenue, Phase 3 price ($15.99) 

## Other
- Bitrupt: respect Pakistan weekend (Sundays off) — no messages on quiet days. Relationship warm, future projects discussed.
- Asif UX review: countered his 40-hr package with scoped $70–100 website review. Awaiting reply.
- Chapter 5 "The First Fruit" published to ZenGate + Facebook. Closing line corrected to day 10 / 522 ratings.
- CGEN site About Us page fully rewritten — repositioned from single-engine Concept Generator identity to three-engine privacy-first platform; dropped personal-story lead, reframed founder bio (third person), updated Direction section to real roadmap (app, briefings, Archives).
- All 5 legal docs updated to four-tier model + correct pricing + Apple/Google/RevenueCat/PayPal payment structure. About Us page rewritten (three-engine, privacy-first identity).

# CGEN Project State — June 16, 2026

## CRITICAL FINDING — Android closed test was never counting
- Discovered the entire June 1–16 Bitrupt test ran on the WRONG Google Play track.
- Build 7 was on **Internal testing**, not **Closed testing**. The opt-in link shared with testers was an internal-test link (`/apps/internaltest/...`), so testers opted into internal — which does NOT count toward production access.
- Play Console "Apply for production" showed: **0 testers opted in**, closed-test requirement not started.
- Root cause: link error on my end (sent internal link, assumed it was the closed opt-in link). Not Bitrupt's fault.
- ~15 days lost on the Android production timeline (not 10 — June 1 to June 16). API cost ~$10/day during that period. Net gains: homepage star ratings (real), one bug (registration-failure) that wouldn't have been caught solo, and hard-won knowledge of how Google closed testing actually works.
- **Does NOT affect:** the app itself (built, stable), Apple submission (still in review), or any other work. Only the Android admin track was delayed.

## FIX DONE TODAY
- Created proper **Closed testing** release with build 7 (1.0.1).
- Added countries/regions to the closed track (was missing — caused rollout error).
- Cleared advertising-ID declaration (App content → Advertising ID → No, app has no ads).
- Correct closed opt-in link: **https://play.google.com/apps/testing/com.cgen.app** (note `/testing/` not `/internaltest/`).
- Invited Asif to Play Console (view access) for transparency on opt-in count.
- Sent Bitrupt fresh-start offer: $137.24 at restart tomorrow + $170.00 at end of 14-day test, both with tip + review. Requirement clarified: 12 testers minimum opted in via correct link (Google's hard minimum) + daily activity. Possible call ~12:00 noon tomorrow.

## NEXT — verify restart is counting
- Watch Play Console "testers opted in" count climb as Bitrupt team accepts the correct link. When it hits 12+, the real 14-day clock starts.
- Each tester must: open link → tap "Become a tester"/accept → then use app. The accept step is what was missing before.

## NEXT BUILD QUEUE (batch into one build — 8→9 iOS + Android closed build)
1. **Pro role recognition** — create WordPress `Pro` role (like CGEN Premium role). Backend endpoint returns tier (premium/pro/free); app (_layout.tsx + login.tsx) reads it and sets cgen_tier. Without this, manual Pro switches aren't seen by the app. Subscriber = registered unpaid; Pro = paid or one of first 100 manually promoted. First 100 = free Pro for life (changed from free Premium — cost protection).
2. **report.tsx title-decode fix** — already saved locally (stripHtml on report.title).
3. **Delete draft from My Lab** — private draft only, Archives untouched. Frees testers/users from duplicate-block when re-running a topic. MUST have confirmation pop-up (same Alert pattern as publish).
4. **FAQ link button in My Lab** — at bottom next to SUPPORT, Linking.openURL to new site FAQ page.
5. **index.tsx** — align Concept Generator modal with new tiers (registration alone no longer unlocks it).

## INDEPENDENT WEB WORK (no build needed)
- Build FAQ page on c93n.com (WordPress). Draft Q&As anytime — doesn't block the app build.

## STATUS SNAPSHOT
- **iOS:** 1.1.0 (8) in Apple review (submitted June 12, full IAP + tier system).
- **Android:** closed test restarting tomorrow on correct track, clock starts when 12+ opt in.
- **Site:** all 5 legal docs current (4-tier model, correct pricing, Apple/Google/RevenueCat/PayPal). About Us rebuilt (3-engine, privacy-first). Homepage star widget live (~522+).
- **Book:** AI & I Ch.5 published. Ch.6 ("The Quiet Work") drafted intro — but the real Ch.6 story is now this: the day 15 days turned out not to count, and the restart. Stronger chapter than planned.
## BACKEND FIXES — queue (main.py)
- Missing post title: some generated reports save with full content but "(no title)" — title step (business name + report number) not firing before save. Ensure title is set before the post is written. Same reliability family as HTML-entity/save issues.
- Note: unpublished duplicate drafts are NOT counted by the backend (only published/Archives reports are dedup-checked) — confirmed expected behavior, no fix needed.
## BACKEND FIXES — queue (main.py) [add to existing]
- Partial-report dedup bug: when a near-duplicate business name is detected (e.g. misspelling then correct spelling minutes apart), the section-skip logic suppresses the first half of the report and emits only Media & Press → Status → Verdict. Web search runs fine; section assembly drops Identity/Market Position/Traction/Financial. Fix: dedup should either (a) cleanly block with the duplicate message, or (b) generate a full report — never emit a half. Review the section-skip path in the generation flow.

- # CGEN Project State — June 17, 2026

## DONE TODAY — WordPress Pro tier (backend complete + tested)
- Created `cgen_pro` role via snippet "Create CGEN Pro Role": add_role('cgen_pro', 'CGEN Pro', array('read' => true)). Appears in Users role dropdown beside CGEN Premium.
- Created `/check-pro` endpoint (separate snippet "CGEN Check Pro Endpoint") — exact twin of /check-premium: JWT → user_id param → session fallback chain, checks in_array('cgen_pro', $user->roles), returns {user_id, is_pro}.
- TESTED both ways: user 118 (not Pro) → is_pro:false; user 11 / ali@gmail.com (CGEN Pro) → is_pro:true. Working.
- Endpoint URL: https://c93n.com/wp-json/cgen/v1/check-pro?user_id=X&JWT=xxx
- NOTE: a user has exactly ONE role/tier at a time (no double roles). Subscriber → manually promoted to CGEN Pro for the 100 founders, or via purchase.

## NEXT SESSION — APP SIDE (build 9). Do these together, ONE build:
All surgical edits — request current file before editing each (files may have changed).

1. **Pro recognition in app** (_layout.tsx + login.tsx):
   - On startup (refreshTierStatus) and on login/register, call BOTH /check-premium AND /check-pro.
   - Tier priority: premium > pro > free. Set cgen_tier accordingly. (Premium wins only as a safety default; in practice users hold one role.)
   - Mirror the existing /check-premium fetch pattern exactly. WordPress is source of truth for manual Pro/Premium grants; RevenueCat is source of truth for purchases (already wired).
   - This makes manually-assigned Pro founders recognized by the app on next launch (same fix pattern as the Premium-refresh fix already done).

2. **My Lab "+" menu redesign** (mylab.tsx):
   - Replace the row of linked buttons with ONE small lemon-green "+" square at the bottom.
   - Tapping "+" opens a popup/modal list: Sign Out, FAQ, Browse CGEN, Upgrade, Support, Delete Account.
   - Delete Account MUST be visually separated (different color, bottom of list) with its existing confirmation — never a mis-tap from Sign Out.
   - "Upgrade" item opens the purchase screen — this ALSO satisfies Apple's 2.1b rejection (reviewer couldn't find IAPs; a visible Upgrade entry point fixes it permanently).

3. **Visible Upgrade entry on Visitor screen too** (per Apple 2.1b) — bottom, reachable without hitting the wall.

4. **Delete draft from My Lab** (mylab.tsx + backend endpoint):
   - Private draft only, Archives untouched. Confirmation pop-up required (Alert pattern like publish).
   - Frees users from duplicate-block when re-running a topic.

5. **report.tsx title-decode fix** — already saved locally (stripHtml on report.title).

6. **FAQ page** — build on c93n.com (WordPress, web work, no build needed). FAQ menu item links to it.

7. **index.tsx** — align Concept Generator modal with new tiers.

## APPLE — build 8 (1.1.0) rejection status
- 3 issues from June 13 review:
  - 2.3.2 duplicate promo images → FIXED (3 unique images for Pro/Monthly/Annual).
  - 3.1.2(c) EULA link → FIXED (Terms + Privacy links added to App Description; Privacy URL in its field).
  - 2.1(b) reviewer couldn't find IAPs → needs build 9 with visible Upgrade button (item 2/3 above). This is the reason build 9 is needed before resubmitting.

## BACKEND FIXES — queue (main.py), separate future session:
- Missing post title: reports save with full content but "(no title)" — title step not firing before save.
- Partial-report dedup bug: near-duplicate business name (misspelling then correct) makes section-skip logic emit only Media&Press→Status→Verdict, dropping first half. Fix: dedup should hard-block OR generate full — never half.

## ANDROID / Bitrupt
- Closed test restarting on CORRECT track (build 7 on Closed testing, link play.google.com/apps/testing/com.cgen.app). 
- Asif (dev.bitrupt@gmail.com) invited to Play Console view access. Verifying all testers complete "Become a tester" before 14-day cycle starts.
- Channel rule: finance/payment/tip/review talk = Fiverr/direct only, NOT team channel.
- Payments: $137.24 at restart + $170.00 at completion, tip + review.
