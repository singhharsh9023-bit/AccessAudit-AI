# AccessAudit AI

AI-assisted website accessibility scanner. Enter a URL, run a real browser-based
audit, and review genuine WCAG findings with rule-based fix guidance.

The scan is **real, not simulated**: a backend service drives headless Chromium
(Playwright) to load the page and runs **axe-core** inside it. Every issue shown
in the UI comes from that live audit.

## Team

| Name | Role | GitHub |
| ---- | ---- | ------ |
| **Harsh Singh** | UI/UX Designer and Frontend Developer (`frontend/`) | [@singhharsh9023-bit](https://github.com/singhharsh9023-bit) |
| **Abhishek Kumar** | Backend Developer, scanning service (`backend/`) | [@ABHISHEKABHI52](https://github.com/ABHISHEKABHI52) |

Built as a hackathon project. Licensed under the [MIT License](LICENSE).

## Project structure

```
AccessAudit AI/
├── frontend/                 React 19 + Vite + Tailwind v4 app
│   ├── public/               Static assets
│   ├── src/
│   │   ├── api/              Backend client (client.js) + tests
│   │   ├── components/
│   │   │   ├── layout/       Navbar, Footer, DashboardLayout
│   │   │   └── ui/           Reusable UI (Button, Badges, CodeBlock, Logo ...)
│   │   ├── features/         Feature-based UI blocks
│   │   │   ├── landing/      Hero, Features, HowItWorks, About, Faq, FinalCta
│   │   │   ├── dashboard/    ScanBox, StatCard, charts, ScansTable
│   │   │   ├── results/      ResultsView, IssueDetail
│   │   │   └── reports/      ReportPreview
│   │   ├── hooks/            useScanData, useMediaQuery
│   │   ├── pages/            Route-level pages (Landing, Overview, History ...)
│   │   └── utils/            Small helpers
│   ├── .env.example
│   └── package.json
├── backend/                  Node.js + Express + Playwright + axe-core service
│   ├── src/                  app, scanner, axeMapper, urlGuard, store, config
│   ├── test/                 Unit, API and real-browser E2E tests
│   ├── Dockerfile            Render / Docker deployment
│   └── README.md             API contract, config, security notes
├── package.json              Root shortcuts (run both apps from one place)
└── README.md
```

- **Frontend** (`frontend/`): React 19, React Router, Tailwind CSS v4, Lucide icons.
  Talks to the backend through `frontend/src/api/client.js`.
- **Backend** (`backend/`): Express API that validates URLs (SSRF protection),
  scans pages with Playwright + axe-core, normalizes results, and persists scan
  history to a JSON file. See [`backend/README.md`](backend/README.md) for the
  full API contract, configuration and security notes.

There is **no mock/demo data** in the application flow. If the backend is
unreachable or a scan fails, the UI shows an honest error state - it never
fabricates results.

## Prerequisites

- Node.js 18+ (developed on Node 22)
- npm
- Internet access on first backend setup (Playwright downloads Chromium once)

## Setup

```bash
npm run install:all                      # installs frontend + backend deps
npx --prefix backend playwright install chromium   # one-time Chromium download
cp frontend/.env.example frontend/.env   # optional; defaults point at localhost:8000
cp backend/.env.example backend/.env     # optional
```

Run both apps in two terminals:

```bash
npm run dev:backend      # API       -> http://localhost:8000
npm run dev              # Frontend  -> http://localhost:5173
```

`frontend/.env`:

```
VITE_API_BASE_URL=http://localhost:8000
```

Backend configuration lives in `backend/.env` (see `backend/.env.example`). By
default private/internal targets (localhost, `10.x`, `169.254.169.254`, ...) are
blocked so the scanner cannot be used for SSRF.

## Scripts

Run from the repo root:

| Command | Purpose |
| ------- | ------- |
| `npm run install:all` | Install frontend and backend dependencies |
| `npm run dev` | Start the Vite dev server |
| `npm run dev:backend` | Start the scanning API |
| `npm run build` | Production frontend build to `frontend/dist/` |
| `npm run lint` | ESLint (frontend) |
| `npm test` | Frontend + backend tests |
| `npm run test:frontend` | API-client unit tests (mocked network) |
| `npm run test:backend` | Backend unit + API + real-browser E2E tests |

## How a scan works

1. The scan form posts the URL to `POST /api/scans`.
2. The backend validates the URL (protocol, DNS, private-network checks) and
   returns `{ id, status: "scanning" }`.
3. A headless Chromium loads the page (bounded timeout), axe-core is injected,
   and the audit runs against the live DOM.
4. Results are normalized to the frontend contract and stored by scan ID.
5. The frontend polls `GET /api/scans/:id` every 1.5s until the status is
   `completed` or `failed`, then renders the genuine findings.

## Testing

```bash
npm run test:frontend    # frontend API-client tests
npm run test:backend     # backend tests, including a real Chromium scan
```

The backend E2E test scans a controlled local fixture page with a known
missing-`alt` image and asserts axe-core reports the genuine `image-alt`
violation.

## Honest limitations

- axe-core catches only a subset of accessibility issues; automated results do
  not prove WCAG compliance. Manual keyboard and screen-reader testing is still
  required (the UI says this too).
- Fix guidance is rule-based remediation text plus the axe-core docs link — it
  is **not AI-generated**, and suggestions are not auto-applied to the site.
- Scan history is stored in a local JSON file (single process, demo scale).
- The arbitrary-URL scanner has SSRF protections but no auth layer; run it on
  trusted networks only.
