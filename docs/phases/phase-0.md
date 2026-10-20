# Phase 0 — Audit and inventory

Date: 2026-10-20

Status: source audit completed; questions and unavailable data are recorded in `docs/QUESTIONS.md`.

## What was done

- Inventoried the supplied site, quotes platform, and schema-only SQL export without executing the legacy apps or contacting Render.
- Created `docs/legacy-audit.md` with the route/page inventory, section map, admin/API behavior, CMS key mechanism and structural mapping, SQL tables/columns, portfolio features, quotes endpoints/models/money rules/PDF layout, integrations, assets, and secret key names only.
- Created `docs/parity-matrix.md`. Rebuild statuses remain `Not started`; no feature has been marked done based only on legacy source inspection.
- Recorded open decisions and missing migration inputs in `docs/QUESTIONS.md`.
- Confirmed `legacy/` is ignored by the repository `.gitignore`.

## Checks and commands

This phase is an audit only; there is no application start command. From the repository root, these Bash commands recheck the source inventory and the main Phase 0 findings:

```bash
find legacy/site legacy/devis legacy/db -type f -print | sort
rg -n --glob '*.sql' '^CREATE TABLE|^INSERT INTO' legacy/db
rg -n 'data-cms|cms\.js' legacy/site/zayerdigital.tn/htdocs/*.html || true
rg -n 'cms-edit|zayer-cms-pick|api\(.schema|api\(.blocks' legacy/site/zayerdigital.tn/htdocs/admin/index.html
rg -n --glob '*.py' '^@app\.' legacy/devis/zayer-devis-platform-main/backend/app
git check-ignore legacy/site/zayerdigital.tn/htdocs/index.html
! rg -n '[[:blank:]]+$' docs
```

The SQL check should show table definitions and no inserts. The public-page CMS search should return no matches; the admin search should show its editor integration. The final command succeeds when there is no trailing whitespace in the Phase 0 documentation.

## Known gaps

- The SQL export contains no rows, so it cannot provide actual CMS blocks, jobs, settings, messages, or site content values. A structural block-to-new-field mapping is documented, but there are no concrete legacy keys to map.
- The current site snapshot has no CMS selectors or script references on public pages, and `seed-content.json` is missing.
- Portfolio data/media and the Zimbra signature HTML are absent from the provided tree.
- Render quote data is not in the archive; the migration decision is open.
- Two empty page files are linked from the home page, and their intended rebuilt behavior is unresolved.

## Verify

Please review `docs/legacy-audit.md`, `docs/parity-matrix.md`, and `docs/QUESTIONS.md`. In particular, answer the Render data migration question before quote migration planning, and confirm whether the supplied site snapshot is complete enough to be the CMS key-mapping source. Phase 1 has not been started.

