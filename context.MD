# aiappinvent Sales Campaign Portal — Session Context

**Last updated:** 2026-06-05
**Purpose:** capture state for a future session to resume mid-task without re-discovery.

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
