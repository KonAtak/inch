# inch.

**Every action gets you 1% closer.**

Inch intercepts the void that drives bad habits and redirects you toward your personal goals with one specific, physical micro-action.

📱 Coming to the App Store

---

## What is Inch?

Most habit apps try to block, shame, or gamify guilt. Inch does something different.

When you're bored at 11pm and about to open Instagram for the fourth time, Inch intercepts that moment and gives you one small, physical thing to do toward the goal you actually care about.

> *"Go grab a banana. That's 300 calories toward your goal."*

Not a motivational quote. Not a streak counter. One action, doable in 5 minutes, tied to your real life.

---

## Core Principles

**The journey never resets.** A relapse is one event on a longer road — not a wipe. Every action is permanent and cumulative.

**Goals must be specific.** "Be better" produces nothing. Inch enforces measurable targets at onboarding. The AI refuses vague goals.

**Nudges are always physical.** Never quotes. Never generic advice. Always one thing to do right now, connected to your exact goal.

**No shame. Ever.** Not in the UI, not in notifications, not in the relapse flow.

**The exit is always visible.** You can dismiss the intercept and continue to the app. Inch redirects — it never locks.

---

## Repo Layout

```
apps/mobile/          Expo / React Native (iOS first, TypeScript)
  ios/                Native iOS project (after prebuild)
  modules/            Local native modules (inch-family-controls)
  src/                Screens, components, navigation, context

supabase/
  migrations/         Postgres schema + RLS policies
  functions/          Edge Functions (AI calls, push, webhooks)
    generate-nudge/   Claude API → specific micro-action
    validate-goal/    Enforces goal specificity at onboarding
    send-nudge-push/  Scheduled vulnerability-window push
    update-portrait/  User AI portrait (mood + pattern memory)
    revenuecat-webhook/ Pro entitlement sync
    check-dormant-users/ Re-engagement for inactive users
    delete-account/   Full GDPR-compliant data wipe
    sync-subscription/ RevenueCat → profiles.is_pro

packages/shared/      Shared TypeScript types (goal/nudge DTOs)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile | Expo + React Native (iOS, TypeScript) |
| Backend | Supabase (Auth, Postgres + RLS, Edge Functions) |
| AI | Anthropic Claude API (via Edge Functions only — never client-side) |
| Notifications | Expo Push Notifications |
| Payments | RevenueCat (entitlement id: `pro`) |
| Analytics | PostHog |
| Intercept | iOS Family Controls + ManagedSettings + Shield Extension |

---

## Prerequisites

- Node 20+
- Supabase project + CLI
- Apple Developer Program membership
- **Family Controls entitlement** approved for `com.getinch.app` (submit at [developer.apple.com](https://developer.apple.com/contact/request/family-controls-distribution))
- EAS account: `cd apps/mobile && npx eas init`

---

## Mobile Setup

```bash
cd apps/mobile
cp .env.example .env   # fill in Supabase URL + anon key
npm install
npx expo start
```

iOS native build (required for Screen Time APIs):

```bash
npx expo run:ios
```

---

## Supabase Edge Function Secrets

Set these in the Supabase dashboard → Edge Functions → Secrets:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Claude API key |
| `REVENUECAT_SECRET_API_KEY` | RevenueCat secret key (not SDK key) |
| `REVENUECAT_WEBHOOK_SECRET` | Webhook signature verification |
| `CRON_SECRET` | Auth header for scheduled function calls |
| `DORMANT_REENGAGEMENT_PUSH` | Set `true` to enable re-engagement push |

Optional tuning:

| Secret | Default | Description |
|---|---|---|
| `PRO_DAILY_NUDGE_CAP` | `30` | Max nudges/day for Pro users |
| `GENERATE_NUDGE_IP_LIMIT` | `120` | Rate limit per IP per window |
| `GENERATE_NUDGE_IP_WINDOW_SECONDS` | `3600` | Window size in seconds |
| `VALIDATE_GOAL_IP_LIMIT` | `60` | Rate limit for goal validation |

---

## Deploy Edge Functions

```bash
supabase functions deploy generate-nudge --no-verify-jwt
supabase functions deploy validate-goal --no-verify-jwt
supabase functions deploy send-nudge-push --no-verify-jwt
supabase functions deploy update-portrait --no-verify-jwt
supabase functions deploy init-portrait --no-verify-jwt
supabase functions deploy complete-goal --no-verify-jwt
supabase functions deploy check-dormant-users --no-verify-jwt
supabase functions deploy revenuecat-webhook --no-verify-jwt
supabase functions deploy delete-account --no-verify-jwt
supabase functions deploy sync-subscription --no-verify-jwt
```

---

## Scheduled Jobs

Both functions require `x-cron-secret` header matching `CRON_SECRET`.

| Job | Schedule | Notes |
|---|---|---|
| `send-nudge-push` | Every 15 min | Fires nudges in vulnerability windows |
| `check-dormant-users` | Daily 10:00 UTC | Re-engagement for inactive users |

```bash
curl -X POST \
  -H "x-cron-secret: $CRON_SECRET" \
  "https://<PROJECT_REF>.supabase.co/functions/v1/send-nudge-push"
```

---

## Data Model

| Table | Purpose |
|---|---|
| `profiles` | 1:1 with auth user, stores `is_pro`, push token |
| `goals` | One active goal per user (v1) |
| `nudges` | Generated micro-actions, completion status |
| `intercept_events` | App bundle ID, shown/dismissed/acted |
| `relapse_events` | Append-only. Never deleted. Never resets progress. |
| `mood_checkins` | 3-second chip selection feeds nudge context |
| `progress_snapshots` | Monotonic journey points — only ever increases |

RLS enforced on all tables. Users read/write only their own rows.

---

## Pricing

| | Free | Pro ($9.99/mo · $59.99/yr) |
|---|---|---|
| Goals | 1 | Unlimited |
| Nudges/day | 3 | 30 (configurable) |
| App intercept | — | ✓ |
| Mood check-ins | — | ✓ |
| Pattern intelligence | — | ✓ |
| Full journey map | — | ✓ |
| Relapse flow | — | ✓ |

---

## License

Private · All rights reserved.

---

*Inch is a personal wellness tool, not a medical device. If you are struggling with addiction or mental health, please reach out to a qualified professional or a support helpline in your region.*
