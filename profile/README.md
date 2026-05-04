# Goodbricks — Reference Codebase for Tino

This organization contains reference repositories shared by **Goodbricks** with **Tino** to support the terminal and widget refactor engagement.

**Goodbricks** is a SaaS platform for Islamic organizations — mosques, Islamic centers, and Muslim nonprofits. It handles online fundraising, in-person giving, events, subscriptions, community discovery, and more.

---

## Repositories

### Virtual Terminal

The Goodbricks in-person payment terminal runs on a dedicated iPad or Android tablet at masjid front desks. It is split into two tightly coupled repos:

| Repo | What it is |
|------|------------|
| [gb2-terminal-expo](https://github.com/goodbricks-tino/gb2-terminal-expo) | React Native / Expo native app — owns the Stripe Terminal SDK, reader lifecycle, offline payments, and session management. Embeds the web UI in a WebView. Also contains all system-level docs (PostMessage API, recovery mechanisms, offline mode, deployment guides). |
| [gb2-terminal-web](https://github.com/goodbricks-tino/gb2-terminal-web) | React + TypeScript + Vite web UI — runs inside the WebView. XState drives all screen transitions. Zustand manages shared state. Radix UI + Tailwind for components. |

**How they fit together:**
```
gb2-terminal-expo (native)
  └── Stripe Terminal SDK (reader / hardware)
  └── WebView
        └── gb2-terminal-web (UI)
              └── XState machine (screen flow)
              └── PostMessage bridge → native
```

---

### Embeddable Widgets

Goodbricks widgets are standalone React bundles deployed to a CDN and embedded on org websites and hosted donation pages.

| Repo | What it is |
|------|------------|
| [goodbricks-donate-widget](https://github.com/goodbricks-tino/goodbricks-donate-widget) | The core donate widget — full donation flow with Stripe, Apple Pay, ACH, recurring giving, fee coverage, and reCAPTCHA. Built as a Create React App bundle. |
| [goodbricks-org](https://github.com/goodbricks-tino/goodbricks-org) | Next.js host site (`goodbricks.org`) — public pages for campaigns, causes, pledges, and donation flows. Each page loads the appropriate widget from the CDN and mounts it with server-fetched data. |

**How they fit together:**
```
goodbricks.org/campaign/[orgId]/[slug]/donate   (goodbricks-org)
  └── loads widget JS from CDN
        └── window.DonateWidget.mount({ cause }) (goodbricks-donate-widget)
              └── Stripe Elements / Apple Pay / ACH
              └── POST → Goodbricks API
```

---

## Key docs to read first

Before diving into code, these files in [gb2-terminal-expo](https://github.com/goodbricks-tino/gb2-terminal-expo) give the full system picture:

- **`POSTMESSAGE_API.md`** — Complete event contract between the web UI and the native layer
- **`README.md`** — Architecture overview, payment layouts, offline mode
- **`RECOVERY_AND_POLLING_MECHANISM.md`** — Autonomous recovery system
- **`UI_SCREENS_GUIDE.md`** — All screens, flows, and health monitoring

---

## Platform context

These repos represent two product surfaces within a larger platform:

```
Goodbricks Platform
├── Admin Portal          — org staff web app (React 19 + Vite)
├── MuslimSpaces          — consumer app (Next.js 15, iOS, Android)
├── Virtual Terminal  ←   these repos
├── Embeddable Widgets ←  these repos
├── Displays              — in-masjid TV screens (React Native)
├── Backend API           — Hono on AWS Lambda + DynamoDB
└── Infrastructure        — AWS CDK (Cognito, EventBridge, SES, SNS, S3)
```
