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
