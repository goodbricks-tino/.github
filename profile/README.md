# Goodbricks — Reference Codebase for Tino

This organization contains reference repositories shared by **Goodbricks** with **Tino** to support the terminal and widget refactor engagement.

---

## What is Goodbricks?

**Goodbricks** is a SaaS platform built specifically for Islamic organizations — mosques (masjids), Islamic centers, and Muslim nonprofits. It serves two audiences:

- **Organization staff** — admins, finance managers, event coordinators, and front-desk volunteers who use Goodbricks to run their operations
- **Community members** — Muslim donors and attendees who discover and give through MuslimSpaces

The platform is built on **AWS Lambda + DynamoDB**, deployed via **AWS CDK**, with payments handled by **Stripe Connect** (one Stripe account per fund per org).

---

## The full platform — all repos

### 🔧 Backend

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-api` | Hono, Node.js 22, AWS Lambda | Single monorepo containing three Lambda functions: `api-admin` (org staff endpoints), `api-muslimspaces` (consumer endpoints), and `events` (async event handlers — emails, push notifications, webhooks, analytics). All routing is done with Hono inside each function. |
| `goodbricks-lib` | TypeScript, npm package | Shared library (`@goodbricks/lib`) published to GitHub npm registry. Contains entity ID generators for every DynamoDB entity and shared TypeScript types used across all backend and frontend repos. |

### 🖥️ Frontend — Staff

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-admin-web` | React 19, Vite, TypeScript | The main org staff portal at `admin.goodbricks.com`. Used by admins, finance staff, and event coordinators to manage donations, campaigns, events, team members, banking, and analytics. Includes the in-browser Virtual Terminal (Cmd+T shortcut). |
| `goodbricks-admin-web-demo` | React 19, Vite | Demo variant of the admin portal with seed data, used for sales and onboarding. |

### 🌐 Frontend — Consumer

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-muslimspaces-web` | Next.js 15, TypeScript | The MuslimSpaces consumer web app at `muslimspaces.com`. Used by community members to discover nearby masjids, view prayer times, donate, register for events, and follow organizations. Also hosts public campaign/donation/event landing pages. |
| `goodbricks-website` | Next.js | Marketing and landing site for `goodbricks.com` — targeted at org admins. |
| `muslimspaces-website` | Next.js | Marketing site for `muslimspaces.com` — targeted at community members. |

### 📱 Mobile & Native

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-muslimspaces-ios` | React Native / Expo | iOS app for MuslimSpaces. Masjid discovery, prayer times with push notification reminders, giving flow, event registration, community feed. |
| `goodbricks-muslimspaces-android` | React Native / Expo | Android equivalent of the iOS app. |

### 💳 Virtual Terminal ← *shared in this org*

The in-person donation terminal. Staff use it on a dedicated iPad or Android tablet to accept card-present donations via a Stripe Bluetooth reader. Split into two tightly coupled repos:

| Repo | Stack | What it does |
|------|-------|-------------|
| [`gb2-terminal-expo`](https://github.com/goodbricks-tino/gb2-terminal-expo) | React Native, Expo 52, Stripe Terminal SDK | Native wrapper. Owns the Stripe Terminal SDK — reader discovery, connection, payment intent lifecycle, offline store-and-forward, 2-hour session management. Embeds the web UI in a WebView and bridges it via PostMessage. Also contains all system-level docs. |
| [`gb2-terminal-web`](https://github.com/goodbricks-tino/gb2-terminal-web) | React 18, TypeScript, Vite, XState 5, Zustand | Web UI that runs inside the WebView. XState drives all screen transitions (org select → connect reader → select layout → payment flow). Zustand stores runtime state. Radix UI + Tailwind for components. |

```
gb2-terminal-expo  (native — Stripe hardware layer)
       ↕  PostMessage bridge
gb2-terminal-web   (web UI — XState screen machine)
       ↕  HTTP
goodbricks-api     (backend — payment intents, org data)
```

**Payment layouts:** Single Page, Multi-Page wizard, Tap-to-Pay (zero-touch auto-loop), Minimal.

**Key docs in `gb2-terminal-expo`:** `POSTMESSAGE_API.md`, `RECOVERY_AND_POLLING_MECHANISM.md`, `OFFLINE_PAYMENT_LOCAL_STORAGE_BACKUP.md`, `UI_SCREENS_GUIDE.md`

---

### 🧩 Embeddable Widgets ← *shared in this org*

Standalone React bundles deployed to a CDN and embedded on org websites and Goodbricks-hosted donation pages.

| Repo | Stack | What it does |
|------|-------|-------------|
| [`goodbricks-donate-widget`](https://github.com/goodbricks-tino/goodbricks-donate-widget) | React (CRA), Material UI, Stripe Elements | The core donation form widget. Handles one-time and recurring giving, card / Apple Pay / Google Pay / ACH, fee coverage, reCAPTCHA, and tribute gifts. Configured entirely via a `cause` prop passed at mount time. Built as a standalone JS bundle deployed to `cdn.goodbricks.org`. |
| [`goodbricks-org`](https://github.com/goodbricks-tino/goodbricks-org) | Next.js (Pages Router) | The `goodbricks.org` host site. Each page (`/campaign/[orgId]/[slug]`, `/cause/...`, `/pledge/...`, `/donate/...`) does a server-side fetch for the data, then dynamically loads the appropriate widget bundle from the CDN and calls `window.<Widget>.mount(data)`. |

```
Donor visits goodbricks.org/campaign/[orgId]/[slug]/donate  (goodbricks-org)
       └── getServerSideProps → goodbricks-api  (fetch campaign data)
       └── injects <script> from cdn.goodbricks.org          (widget CDN)
             └── window.DonateWidget.mount({ cause })        (goodbricks-donate-widget)
                   └── Stripe Elements / Apple Pay / ACH
                   └── POST → goodbricks-api                 (process donation)
                         └── EventBridge → events lambda     (receipt email, analytics)
```

Other widget types (CampaignWidget, CauseWidget, StatsWidget, PledgeWidget) follow the same CDN-load + mount pattern but are in separate repos not included here.

---

### 📺 Displays

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-displays` | React Native (Android) | In-masjid TV/screen display app. Shows real-time donor walls, campaign progress, prayer times, and announcements. Deployed to dedicated Android screens. |

### ⚙️ Infrastructure

| Repo | Stack | What it does |
|------|-------|-------------|
| `goodbricks-infra` | AWS CDK v2, TypeScript | All cloud infrastructure as code. Stacks: DynamoDB (single-table), Cognito (admin + consumer user pools), API Gateway v2, Lambda functions, EventBridge (bus + pipes + rules + scheduler), S3 + CloudFront, SES, SNS (push notifications). One config file per environment (`sandbox.yml`, `prod.yml`). |

---

## How all the pieces connect

```
                    ┌─────────────────────────────────────────┐
                    │          goodbricks-infra (CDK)         │
                    │  DynamoDB · Cognito · APIGw · Lambda    │
                    │  EventBridge · S3 · SES · SNS · CF      │
                    └──────────────┬──────────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────────┐
                    │           goodbricks-api                 │
                    │  api-admin │ api-muslimspaces │ events   │
                    └───┬────────┬───────────┬────────────────┘
                        │        │           │
          ┌─────────────▼─┐  ┌───▼──────┐  ┌▼──────────────────────────┐
          │ Admin Portal  │  │MuslimSpc │  │  Async Event Handlers      │
          │ admin-web     │  │-web / app│  │  emails · push · analytics │
          └───────────────┘  └──────────┘  └───────────────────────────┘
                │
    ┌───────────┴──────────────────────────────┐
    │                                          │
┌───▼──────────────┐              ┌────────────▼─────────────┐
│  Virtual Terminal│              │  Embeddable Widgets       │
│  gb2-terminal-   │              │  goodbricks-org           │
│  expo + web      │              │  + goodbricks-donate-     │
│  (this org)      │              │    widget  (this org)     │
└──────────────────┘              └──────────────────────────┘
```

---

## Shared infrastructure patterns

Every frontend app follows the same auth model: **AWS Cognito JWT tokens** sent on every API request. Two user pools — one for org staff (admin), one for community members (consumer).

Every async side-effect (send receipt email, update analytics, trigger push notification) flows through **EventBridge** — API writes to DynamoDB, DynamoDB Streams pipe into EventBridge, rules route events to the appropriate Lambda handler. No direct calls between services.

Payments use **Stripe Connect** — each org fund has its own Stripe account, platform fees are collected at the Goodbricks level. In-person payments additionally use the **Stripe Terminal SDK** in `gb2-terminal-expo`.
