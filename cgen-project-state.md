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
