# Our Way of Life — Timeline & Story Site

A bilingual+ZH (English / Español / 中文) community-research site for the
**Our Way of Life Archive**. Hosts an interactive historical timeline and a
multilingual story-submission survey, with a password-gated admin editor that
lets non-developers update content directly through the browser.

**Live site:** https://wjxqmv.top
**Embedded on:** https://www.ourwayoflifearchive.com (Square Online)

---

## Two ways to make changes

### 1. Content edits (no code, no install) — the admin editor

Most updates (text, year labels, video URLs, adding new timeline events,
hiding events, editing the survey questions) happen here. **No GitHub, no
deploy, no Netlify knowledge required.**

- **URL:** https://wjxqmv.top/admin/
- Sign in with the admin password (ask the project owner)
- Choose **Edit timeline** or **Edit story form** from the chooser
- Edit text inline in any of EN / Español / 中文
- Click **Save changes** — live within seconds
- Click **Revert to baseline** to restore the snapshot if anything breaks

### 2. Code edits (for structural changes, new features, bug fixes)

Requires Node.js, the Netlify CLI, and write access to this repo + the
Netlify project (`cami-project`).

```bash
git clone https://github.com/camacosta100/our-way-of-life-timeline.git
cd our-way-of-life-timeline
npm install
netlify link            # link to the cami-project Netlify site (need access)
netlify dev             # local dev server with functions
```

To deploy:

```bash
netlify deploy --prod
```

---

## Repo layout

```
.
├── index.html                       # Timeline page (lives at /)
├── story/index.html                 # Survey / story-submission form (/story/)
├── admin/
│   ├── index.html                   # Admin landing (chooser)
│   ├── story/index.html             # Edit story form
│   └── timeline/index.html          # Edit timeline
├── content.js                       # Default i18n strings + question list,
│                                    # loaded by both the story page and admin
├── netlify/functions/
│   ├── submit-story.mjs (.js)       # Receives form submissions → emails via Resend
│   ├── get-content.mjs              # Reads admin overrides from Netlify Blobs
│   └── save-content.mjs             # Writes admin overrides + login notifications
├── snapshots/
│   ├── story-baseline.json          # Frozen "known good" state for Revert button
│   └── timeline-baseline.json
├── netlify.toml                     # Netlify build config
└── package.json                     # Single dep: @netlify/blobs
```

---

## How the admin editor works

The site has two storage surfaces, both backed by Netlify Blobs:

- `?surface=story` — overrides for the survey form (i18n keys + the dynamic
  question list).
- `?surface=timeline` — overrides for the timeline (per-element text in
  en/es/zh, year labels, YouTube embed URLs, hidden events, reordering,
  custom new events).

On page load, the public pages fetch their respective overrides and merge
them on top of the defaults from `content.js` / the static HTML. Admin saves
overwrite the Blob and take effect on the next visitor's page load.

**Auth:** the editor uses a single shared password stored as `ADMIN_PASSWORD`
in Netlify env vars. Save attempts hit a rate limit (5 failed/min/IP), and
every successful login sends a notification email to
`ourwayoflifearchive@gmail.com` via Resend so unexpected sign-ins are visible.

---

## Environment variables (set in Netlify)

| Name              | Used by               | Purpose                                          |
|-------------------|-----------------------|--------------------------------------------------|
| `ADMIN_PASSWORD`  | `save-content.mjs`    | Gates admin saves                                |
| `RESEND_API_KEY`  | `submit-story.js`, `save-content.mjs` | Sends story-submission and login-notification emails |

---

## Domain

`wjxqmv.top` is registered at Porkbun and pointed at Netlify via:

- `A` record on apex → `75.2.60.5`
- `CNAME` on `www` → `cami-project.netlify.app`

The old domain `lalacami.art` has been retired.

---

## Translations

The 216 timeline segments were translated to Simplified Chinese in bulk
using parallel sub-agents (one-time job). New events added through the
admin editor need their Chinese translations entered manually — there's no
auto-translate wired in.

---

## Conventions

- Plain HTML + vanilla JS + a few Netlify Functions. No frameworks, no
  build step.
- Default branch is `main`. Push to deploy via Netlify CLI (no
  GitHub-triggered auto-deploy is wired up).
- Functions use Netlify v2 ESM format (`.mjs`, `export default`). The
  one v1 file (`submit-story.js`) predates that convention.
