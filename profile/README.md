# Goodbricks — Reference Codebase for Tino

This organization contains reference repositories shared by **Goodbricks** with **Tino** to support the terminal and widget refactor engagement.

---

## Repositories

### Virtual Terminal

The in-person donation terminal used by front-desk staff on a dedicated iPad or Android tablet. Two repos work together:

| Repo | Stack | Role |
|------|-------|------|
| [gb2-terminal-expo](https://github.com/goodbricks-tino/gb2-terminal-expo) | React Native / Expo | Native app. Owns the Stripe Terminal SDK — reader discovery, connection, payment lifecycle, offline mode. Embeds the web UI in a WebView and communicates via PostMessage. |
| [gb2-terminal-web](https://github.com/goodbricks-tino/gb2-terminal-web) | React + TypeScript + Vite + XState | Web UI that runs inside the WebView. XState drives all screen transitions. Zustand manages state. Radix UI + Tailwind for components. |

```
gb2-terminal-expo   native layer — Stripe hardware, reader management
       ↕  PostMessage bridge
gb2-terminal-web    UI layer — XState screens, payment flow, settings
```

Start with `POSTMESSAGE_API.md` and `README.md` inside `gb2-terminal-expo` — they document the full system.

---

### Embeddable Widgets

Donation widgets embedded on org websites and hosted donation pages. Two repos work together:

| Repo | Stack | Role |
|------|-------|------|
| [goodbricks-donate-widget](https://github.com/goodbricks-tino/goodbricks-donate-widget) | React (CRA) + Stripe Elements | The donation form widget — one-time and recurring giving, card / Apple Pay / ACH, fee coverage. Built as a standalone JS bundle deployed to a CDN. |
| [goodbricks-org](https://github.com/goodbricks-tino/goodbricks-org) | Next.js | Host site for public donation pages. Each page fetches data server-side then loads the widget bundle from the CDN and calls `window.DonateWidget.mount(data)`. |

```
goodbricks-org      host site — fetches data, provides the page shell
       └── loads widget JS from CDN
goodbricks-donate-widget    mounted into the page — handles the full donation flow
```

---

## Backend API endpoints

These are the Goodbricks API endpoints called across the four repos. Two base URLs are in use — `https://goodbricksapp.com` (older, used by the widget and terminal web) and `https://api.goodbricks.app` (newer, used by the native app). Both point to the same backend.

### Virtual Terminal — `gb2-terminal-expo`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/stripe/connection-token?organizationPublicId=[orgId]` | Fetches a short-lived Stripe Terminal connection token scoped to the org. Called each time the native app initializes or re-initializes the Stripe Terminal SDK. |
| POST | `/s3/generate-signed-url` | Requests a presigned S3 URL for uploading an on-device log file. Used by the file-based logger to ship diagnostic logs off the device. |
| PUT | `[presignedUrl]` | Uploads the log file directly to S3 using the presigned URL returned above. |

### Virtual Terminal — `gb2-terminal-web`

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/public/vendorOrganization/[orgId]/publicCategories` | Fetches the donation categories (funds) configured for the org — used to populate the category selector on the payment screens. |
| POST | `/organization/search` | Searches for orgs by name. Called from the org select screen when staff type to find their organization during terminal setup. |
| POST | `/api/stripe/retrieve_customer` | Looks up an existing Stripe customer record by payment intent ID. Used to pre-fill donor info after a card tap. |
| POST | `/api/stripe/retrieve_customer/payment_method` | Looks up a Stripe customer by payment method ID. Used when a saved card is reused. |
| POST | `/api/stripe/update_payment_intent` | Attaches customer name, email, and phone to an in-flight payment intent before capture. |
| POST | `/api/stripe/capture_payment_intent` | Captures the payment intent and, if recurring giving was selected, creates a Stripe subscription. This is the final step that confirms the charge. |

### Embeddable Widgets — `goodbricks-org`

All called server-side in `getServerSideProps` to hydrate the page before the widget loads.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/public/vendorOrganization/[orgId]/cause/[slug]` | Fetches a cause (fund) record — name, cover image, amount presets, fee config, zakat eligibility. Used by the cause page, donate page, and pledge page. |
| GET | `/api/public/vendorOrganization/[orgId]/campaign/[slug]` | Fetches a campaign record — goal, progress, deadline, description, cover image. Used by the campaign page, campaign donate page, and stats page. |
| GET | `/api/public/vendorOrganization/[orgId]/page/[slug]` | Fetches a custom org page — title, rich content, metadata. Used by the generic page route. |

### Embeddable Widgets — `goodbricks-donate-widget`

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/[orgId]/donate` | Submits the completed donation form. Payload includes amount, category, donor info, Stripe token, cover-fees selection, recurring options, and reCAPTCHA token. This is the main transaction endpoint. |
| GET | `/api/stripe/create_link_token` | Creates a Plaid link token to initiate the bank account connection flow for ACH donations. |
| POST | `/api/stripe/bank_token` | Exchanges the Plaid public token for a Stripe bank token after the donor completes the Plaid OAuth flow. |
