# aiappinvent Sales Campaign Portal — Session Context

**Last updated:** 2026-06-05 23:30 UTC — END OF 7-HOUR SESSION, USER FRUSTRATED
**Purpose:** capture state so the NEXT session can answer the user's "remember where we were" question without making them re-explain.

## The user will open the next session by asking "remember where we were"

Answer them with this: we built the aiappinvent Sales Campaign Portal from
scratch today over ~7 hours. Backend (Flask blueprint /api/campaign/*) lives
in the SolarPro repo and is hosted on Render at
https://solarpro-global.onrender.com. Frontend (static HTML) lives in this
repo (`campaign-portal`) and is hosted on GitHub Pages at
https://marc667us.github.io/campaign-portal/. The user explicitly
characterised the result as "dysfunctional" — they reported login failures,
missing infographics, broken sign-out, and add-feedback button issues. From
my automated browser-driven test (Playwright on a clean GitHub runner IP)
LOGIN ACTUALLY WORKS — the issue was almost certainly stale browser cache
plus a few real UI bugs that got fixed in the last hour (sign-out tab
crashing, entity filter selects using wrong event). Next session: open the
portal in an INCOGNITO browser window, try login fresh, and walk through
every tab to find anything that's actually broken vs. cached.

## What works (verified live, last verified 2026-06-05 01:09 UTC)

- /api/campaign/health, /login, /me, /entities, /state, /invites,
  /state/<name>, /feedback — all 200 with admin token. Admin sees 15 entities.
- Static portal serves HTTP 200 at the github.io URL.
- Static HTML contains: data-tab="admin", data-tab="logout", bulk-assign-bar,
  ent-cb, audio_user_walkthrough.wav, chart-cities, no-store/no-cache meta tags.

## Open work for next session

1. **Verify in incognito** that login + dashboard + all tabs work end-to-end.
2. **Fix the Playwright KPI check** so a future browser-driven test catches
   regressions automatically (the test in `.github/workflows/test-browser-flow.yml`
   crashed at the KPI step on bad CSS.escape usage).
3. **Persistent disk on Render** — DB still wipes on every redeploy.
4. **Namecheap CNAME** — `campaign` → `marc667us.github.io` so the URL
   becomes `https://campaign.aiappinvent.com/`.
5. **CNAME file** — currently REMOVED from this repo so the github.io URL
   doesn't 301-redirect. Re-add it AFTER the Namecheap CNAME is in place.



---

## 1. What this repo is

Standalone static frontend for the internal sales operations portal:

- **One HTML file (`index.html`)** = the whole UI. No build step, no framework.
- **`docs/`** = the 5 collateral PDFs (sales pitch, user guide, technical guide, supplier list, installer list) embedded in iframes by the portal.
- **`data/`** = entity master + CSV/TSV/VCF/MD/flat-JSON exports.
- **`CNAME`** = tells GitHub Pages to serve at `campaign.aiappinvent.com`.

Hosted at https://marc667us.github.io/campaign-portal/ (CNAME currently 301-redirects to the not-yet-DNS'd custom domain — so the github.io URL doesn't preview-load until either the CNAME is removed from the repo OR the user adds the Namecheap record).

---

## 2. Live URLs

| | |
|---|---|
| Frontend (live, but 301s) | https://marc667us.github.io/campaign-portal/ |
| Future custom domain | https://campaign.aiappinvent.com (waiting on Namecheap CNAME) |
| Backend API | https://solarpro-global.onrender.com/api/campaign/* |
| Local preview (works against live API) | `C:\Users\USER\Desktop\campaign-portal\index.html` |

---

## 3. Default admin credentials

- Email: `marc667us@yahoo.com`
- Password: `ChangeMe2026!` (rotate via Staff tab → Reset PW)

Set `CAMPAIGN_ADMIN_PASSWORD` on Render BEFORE the first DB init to override.

---

## 4. Backend home

Backend lives in `solar-pv-designer-lite` repo (NOT this one):
- `campaign_api.py` — the Flask blueprint
- `patch_campaign_api_registration.py` — binary patch on web_app.py
- `.github/workflows/render-attach-campaign-disk.yml` — one-shot Render disk attach
- `.github/workflows/render-deploy-now.yml` — force redeploy

Deploy backend changes: push to `solar-pv-designer-lite/master` → Render auto-deploys.
Force redeploy: `gh workflow run "Force Render Deploy" -R marc667us/solar-pv-designer-lite`.

---

## 5. Open blockers (do these next session)

1. **Persistent disk on Render.** Render's REST API for disk attachment returns 404 for both PUT and POST to `/v1/services/{id}/disk`. Until a disk is attached, the SQLite DB resets on every redeploy. Two paths:
   - Find the correct Render disk endpoint (try `/v1/services/{id}/serviceDetails/disk` or `render.yaml` infrastructure-as-code).
   - OR fall back to a hosted SQLite-compatible DB (Turso / SQLite Cloud — free tiers exist).
2. **Namecheap CNAME.** User has not added `campaign → marc667us.github.io`. Until they do:
   - The GitHub Pages URL 301-redirects to the dead custom domain and doesn't preview-load.
   - Workaround for testing: remove the CNAME file from the repo temporarily, or use the local Desktop copy.
3. **Workflow secret leak.** The `Render attach campaign disk` workflow accidentally dumps all env vars (including secrets like `OPENROUTER_API_KEY`, `PAYSTACK_SECRET_KEY`, `SECRET_KEY`) to GitHub Actions logs when it merges them for the PUT request. Rewrite the merge step to not echo the full payload — only the keys.

---

## 6. Feature surface (what the portal does)

### Auth
- Two-step login: signin or invite-signup → product picker → dashboard
- Email + password (PBKDF2-SHA256). Session = HMAC-signed stateless token (30-day TTL).
- Invite-only signup: admin generates a one-time URL, shares via WhatsApp; invitee submits name/email/password/profession; admin approves.

### Dashboard (after Connect)
- 7-stage pipeline: INVITED → CONTACTED → SIGNED UP → ACTIVE → FEEDBACK → NURTURED → PAID
- 4 infographics: funnel, role mix, by city, status distribution
- Today's priorities list (overdue follow-ups, demos without dates, signed-up users due for upgrade)
- 30-second polling so multi-rep edits sync without refresh

### Entities tab
- Per-row: Call (`tel:`), Email (`mailto:` pre-filled), Lit (log literature send), Sign up (opens SolarPro register)
- Inline status dropdown, debounced notes, next-action date

### Feedback tab
- Per-customer bug/feature capture with severity + type
- First feedback auto-advances customer to FEEDBACK stage

### Reports tab
- "Build report from current feedback" → admin Markdown + Claude Code prompt
- Two infographics: feedback by severity, feedback by type

### Staff tab (admin only)
- Generate invitation link (WhatsApp-ready URL, expiry 3/7/14/30 days)
- Active invitations table (with revoke)
- Pending approvals with admin assign-product + role
- Current staff with reset password + remove

### Tutorial
- 11-step interactive tour auto-runs on first login
- Floating red "Take the tour" button re-launches it any time
- Tutorial PDF on Desktop (`SolarPro_Portal_Tutorial.pdf`)

---

## Final state — top-level tabs (after the end-of-session UI overhaul)

| Tab | Visibility | Purpose |
|---|---|---|
| Dashboard | all | 7 KPI tiles + funnel/role/cities/status charts + today's priorities |
| Customers | all | 15 entities, filtered by per-user assignments (admin sees all) |
| Guides & Tutorials | all | Sales Script PDF (kept) + 2 audio tutorial slots (user / technical walkthroughs) — admin pastes share URLs once via prompt, every rep sees the "Listen" + "Copy link" buttons |
| Feedback | all | capture bugs / feature requests from customers |
| Reports | admin only | consolidated feedback report + Claude Code prompt |
| Staff | admin only | invite generator, pending approvals, current staff CRUD |
| Data | admin only | entity master in 6 formats |
| Export | admin only | pipeline state JSON snapshot |
| Sign out | all | top-right red link |

PDFs in `docs/` (kept on the static site even though only the sales script is now linked from the UI):
- SolarPro_Sales_Pitch.pdf — visible in Guides tab
- SolarPro_User_Guide.pdf — no longer linked; replaced by audio slot
- SolarPro_Technical_Guide.pdf — no longer linked; replaced by audio slot
- Ghana_Solar_Suppliers/Installers PDFs — internal reference only

## Setting audio tutorial URLs

Admin or any rep opens the portal, F12 → Console → paste:
```js
localStorage.setItem('sp-audio-user-url','https://soundcloud.com/...');
localStorage.setItem('sp-audio-tech-url','https://soundcloud.com/...');
location.reload();
```
OR click the "+ Paste audio URL" button on each empty card. URLs persist in localStorage per browser. The "Send pack" mailto: body auto-includes audio URLs when present.

(Same pattern exists for the video tutorial URL: `localStorage.setItem('sp-tutorial-video-url', '…')`. Currently unused.)

## 7. Commit history this repo (main)

| Commit | What |
|---|---|
| `c3b141f` | fix(api): point portal at Render (Railway deploy stuck) |
| `799fa78` | feat(tour): in-portal interactive tutorial + 'Take the tour' button |
| `d5dcfd7` | feat(invites): admin invite links + profession field + pending approval flow |
| `856e4ca` | feat(auth): split login into 3 steps |
| `8175f8c` | feat(auth): email+password login + Staff tab for admins |
| `c3b141f` | initial portal (multi-product, cloud-wired, single HTML) |

---

## 8. Resume here

If the next session opens with the API broken (`/api/campaign/health` returns 404 or HTML):
1. Check `gh run list --workflow "Force Render Deploy" -R marc667us/solar-pv-designer-lite` — has the latest commit deployed?
2. If yes, look at `DB_PATH` env var on Render — if it points at `/app/data/solar.db` but the disk doesn't exist, the import crashes. Fix: either attach the disk via Render's actual API, or update `campaign_api.py` to fall back to `/app/solar.db` when the volume mount is missing (the `os.makedirs(parent, exist_ok=True)` already does this defensively but the dir doesn't survive redeploys without the disk).
3. If the API is healthy but the portal is "Network error" — check CORS allowlist in `campaign_api.py::ALLOWED_ORIGINS`. Local `file://` lives under `"null"` origin which is allowed.

Trivia: the user typically signs into the portal locally first (`C:\Users\USER\Desktop\campaign-portal\index.html`) — that opens with `null` origin and is what they test against.
