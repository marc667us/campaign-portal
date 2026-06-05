# aiappinvent — Sales Campaign Portal

Standalone intranet portal for the aiappinvent sales team. Static site, no backend.

- **Live:** https://campaign.aiappinvent.com
- **Hosted:** GitHub Pages (free, HTTPS auto-provisioned via Let's Encrypt)
- **Independent of:** the SolarPro production app

## What's inside

- `index.html` — the portal (login → product → dashboard → entities → feedback → reports)
- `docs/` — sales pitch + user guide + technical guide PDFs (5 files)
- `data/` — entity master data in 6 formats (JSON, CSV, TSV, VCF, MD)
- `CNAME` — tells GitHub Pages to serve at campaign.aiappinvent.com

## Pipeline

INVITED → CONTACTED *(call + literature sent)* → SIGNED UP → ACTIVE → FEEDBACK → NURTURED → PAID

## Updating

This is a static deployment. Push any commit to `main` → GitHub Pages redeploys in ~30 s.

To regenerate the PDFs or data exports, the build scripts live in the SolarPro repo
(`solar-pv-designer-lite/`):

```
python build_collateral_pdfs.py      # rebuilds the 3 collateral PDFs
python build_beta_invitee_pdfs.py    # rebuilds the 2 invitee PDFs
python export_entity_data.py         # rebuilds all 6 data exports
```

Copy outputs from `solar-pv-designer-lite/docs/` and `solar-pv-designer-lite/data/exports/`
into this repo's `docs/` and `data/`, then `git push`.

## Architecture note

Originally bundled inside the SolarPro Flask app; moved out 2026-06-05 so the
sales portal and the production app deploy and scale independently.
