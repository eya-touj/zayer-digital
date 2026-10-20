# MASTER PROMPT — Rebuild ZAYER Digital as a Dockerized MERN (TypeScript) monorepo

> Paste this whole file into Codex (VS Code) as the first message, or save it as `AGENTS.md` at the repo root.
> You (Codex) work **phase by phase**. At the end of each phase you STOP, report, and wait for my approval before starting the next one.

---

## 0. Working rules (read first, apply always)

1. **Do not assume.** If information is missing or ambiguous, write the question in `docs/QUESTIONS.md`, ask me, and wait. Never invent business content, prices, legal text, URLs, or credentials.
2. **Phase gates.** Finish one phase → run its checks → write a short report (what was done, commands to run, what to verify, known gaps) → STOP. Do not start the next phase until I say "go".
3. **Copy-paste friendly docs.** For every phase, add the exact bash commands I must run to start and verify it in `docs/phases/phase-N.md`. I am a student with MERN + React Native level: explain non-obvious choices in one or two sentences.
4. **Small commits, clean history.** Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`). One logical change per commit. Branches: `main` (always working), `develop`, and one branch per phase `phase/N-short-name`, merged into `develop` after my approval, `develop` → `main` at milestones.
5. **Zero secrets in Git, ever.** Never commit real credentials, `.env`, SQL dumps, `uploads/`, CVs, or messages. Never print secret values in logs, reports, or commits. If you find credentials in the legacy files, list only the *file and key name* in `docs/legacy-audit.md`, never the value. Install a pre-commit secret scan (gitleaks) in Phase 1.
6. **Pin and verify dependencies.** Use the latest stable versions at the time you work, pinned exactly (lockfile committed), on the current Node LTS (pin it in `.nvmrc` and in Docker). Run `pnpm audit` and fix or document every high/critical finding. Do not add a dependency when the platform or an existing one does the job.
7. **Language of the site content** stays exactly as in the legacy site (EN and FR). Do not rewrite, translate, or "improve" copy.
8. **Everything is documented in English** (code comments, README, commit messages). The UI text stays EN/FR as in the legacy site.

---

## 1. Context

ZAYER Digital is a digital marketing agency in Tunisia. The current site `zayerdigital.tn` runs on free shared hosting (InfinityFree, PHP + MySQL, FTP only). I am providing the legacy code as ZIP files (sanitized: no `config.php`, no hard-coded passwords, no `.env`). Extract them into a **git-ignored** `legacy/` folder at the repo root. They are read-only reference material: never modify them and never copy secrets from them.

Legacy inventory (verify it in Phase 0; correct this list if the code says otherwise):

- **5 public pages**, static HTML, bilingual EN/FR with a language toggle: `index`, `about`, `services`, `careers`, `contact`. Hero videos (`hero-1.mp4`, `hero-2.mp4`).
  Brand: navy `#12295a`, gold `#c8a96e`; fonts Playfair Display, DM Sans, Bricolage Grotesque, Inter.
- **`/portfolio`**: single-file PHP app with its own password-protected admin: tabs for media (images, videos, YouTube), contact boxes, services with prices, social networks; uploads.
- **`/devis`**: the **sales quote platform**. Legacy source (sales-team platform) is a separate ZIP (`zayer-devis-platform-main`): React + Vite + Capacitor Android frontend, and a **Python FastAPI + SQLAlchemy** backend. Features: service catalog, custom line items, VAT 19% with payment plans (deposit/balance, split plans), quote statuses (draft/sent/accepted/rejected), client approval (pending/confirmed/declined), payment tracking (unpaid/deposit_paid/fully_paid), retainers (recurring monthly lines, per-month collections), roles `admin`/`sales`, PDF generation in ZAYER format (reportlab), `/public/services`. Ignore `*.bak` files.
- **`/admin`**: home-made CMS (vanilla HTML+JS): login; tabs Content, Job offers, Settings, Messages (contact form + job applications); and a **Visual Editor**: click any text/link/image/video on a page to edit it in EN and FR, hide it, or restore the original; secure image upload.
- **`/api`**: PHP + MySQL (tables `items`, `jobs`, `messages`, `settings`, `admins`, `blocks`, `rate_hits`), sessions + CSRF, rate limiting on login and forms.
- **Careers form** uses Formspree (to be replaced). **Email signature** HTML for Zimbra exists (keep it in `docs/email-signature/`).
- Content edited through the Visual Editor lives in the MySQL `blocks` table, not in the HTML files. I will provide a **sanitized SQL export** (no `admins`, no `rate_hits`) and the `uploads/` folder for the migration phase.

---

## 2. Goals and non-negotiables

1. **Rebuild everything as a MERN monorepo in TypeScript**, Dockerized, in a **new clean Git repository**.
2. **Pixel-identical design** and **100% feature parity.** Nothing visible may change: layout, spacing, colors, fonts, animations, videos, copy, language toggle behavior. Nothing that works today may disappear.
3. **The admin controls everything on every page**: texts, links, images, videos, section visibility (shown/hidden), section order, job offers, portfolio, EN and FR, with the visual editor.
4. **SEO-ready via SSR** (Vite SSR + Express).
5. **Serious security** (section 8).
6. **One command** runs the whole stack: `docker compose up --build`.
7. **Portable**: it must run locally now and later on a Tunisian VPS (ATI) without code changes: only `.env` and DNS.
8. **Frontend first with mock data, backend second.**

---

## 3. Decisions already made (do not re-open)

| Topic | Decision |
|---|---|
| Language | TypeScript everywhere (strict mode), shared types and zod schemas |
| Frontend | React + Vite, **SSR via Express** (HTML rendered from DB content, then hydration) |
| Languages | EN + FR only, **FR default**, URL per language (`/fr/...`, `/en/...`), `hreflang`, sitemap |
| Backend | Node.js + Express + Mongoose + zod, **single API** |
| Database | MongoDB in Docker on the same VPS (volume + automated `mongodump` backups) |
| Auth | Server-side sessions (express-session + connect-mongo), `httpOnly` cookie, CSRF protection, argon2 |
| Email | Nodemailer + SMTP (Zimbra), credentials in `.env`; messages are always stored in MongoDB first |
| Quotes backend | **Full port of the Python backend to Node/TS** inside the single API, with parity tests (PDF and money calculations) |
| Android | Keep the Capacitor Android app for the quotes frontend |
| Migration | Migrate everything: content, `blocks`, jobs, portfolio, settings, messages, applications, uploads. **Admin accounts are NOT migrated** (created fresh) |
| Hosting | Local Docker now; ATI VPS later |

---

## 4. Tech stack

- **Monorepo**: pnpm workspaces, Node LTS, TypeScript strict, ESLint + Prettier, Vitest, Playwright, Supertest, `mongodb-memory-server` for API tests.
- **Web (public site + admin + visual editor)**: React, React Router, Vite SSR (`entry-server.tsx` / `entry-client.tsx`), CSS reproducing the legacy styles exactly (design tokens extracted from legacy CSS), no UI framework that changes the look.
- **API**: Express, Mongoose, zod, `helmet`, `express-rate-limit`, `express-session` + `connect-mongo`, CSRF (`csrf-csrf` or equivalent maintained double-submit library; do not use deprecated `csurf`), `argon2`, `multer`, `file-type` (real MIME sniffing), `sharp` (re-encode images, strip metadata), `sanitize-html` (or DOMPurify on server) for rich text, `express-mongo-sanitize`, `pino` with secret redaction, `nodemailer`, `pdfkit` or `pdfmake` for PDFs (choose after Phase 0 audit of the legacy PDF layout and justify).
- **Infra**: Docker (multi-stage, non-root), docker-compose, nginx reverse proxy, TLS-ready (certbot or Caddy documented), backup sidecar.

---

## 5. Target project structure

```
zayer-digital/
├─ apps/
│  ├─ web/                  # public site + /admin + visual editor (React, Vite SSR)
│  ├─ devis-web/            # quotes frontend (React + Vite + Capacitor Android)
│  └─ api/                  # Express API + SSR host + migrations + seeds
├─ packages/
│  └─ shared/               # zod schemas, TS types, i18n helpers, money/VAT logic
├─ docker/
│  ├─ nginx/                # nginx.conf (dev + prod), security headers, caching
│  ├─ mongo/                # init script (app user, auth enabled)
│  └─ backup/               # mongodump + retention script
├─ docs/
│  ├─ legacy-audit.md  parity-matrix.md  QUESTIONS.md  deployment.md  security.md
│  ├─ phases/phase-N.md     # copy-paste commands per phase
│  └─ email-signature/
├─ legacy/                  # git-ignored, read-only reference
├─ docker-compose.yml  docker-compose.dev.yml
├─ .env.example  .gitignore  .nvmrc  README.md  AGENTS.md
```

**Frontend-first rule:** the web app talks to data only through a typed `ContentRepository` (and similar repositories for jobs, portfolio, messages, quotes). Phase 2 to 5 use a `MockRepository` with fixtures extracted from the legacy HTML. Phase 6+ adds `ApiRepository` / Mongo-backed implementations selected by env var, with no UI change.

---

## 6. Content and CMS model (critical)

- Every editable element on every page has a **stable key** (e.g. `home.hero.title`). Build small components (`<CmsText>`, `<CmsLink>`, `<CmsImage>`, `<CmsVideo>`, `<CmsSection>`) that render content by key.
- A content block stores: `key`, `page`, `type` (text | richtext | link | image | video | section), `value: { en, fr }`, `hidden`, `order`, `originalValue` (for "restore original"), `updatedAt`, `updatedBy`.
- Sections are first-class: visibility toggle and **ordering** per page.
- **Visual Editor** (admin only): same rendered page + overlay. Hover/click any keyed element → edit EN and FR, hide/show, restore original, replace image/video (secure upload), edit link target. Save is validated server-side (zod) and sanitized. Public pages never ship editor code.
- Phase 0 must reverse-engineer how legacy `blocks` identify elements (selectors? indexes? keys?) and produce a **mapping table legacy-block → new key** so the migration preserves every edit.
- SSR output must include the correct language, `<title>`, meta description, canonical, `hreflang`, Open Graph/Twitter tags, JSON-LD (Organization), driven by editable CMS fields. Redirect legacy URLs (`/index.html`, `/about.html`, etc.) to the new language URLs with 301. Generate `sitemap.xml` and `robots.txt`. Cache rendered HTML and **invalidate on admin save**.

---

## 7. Features checklist (every item must reach "done" in `docs/parity-matrix.md`)

**Public site**
- [ ] 5 pages (home, about, services, careers, contact), EN/FR with language toggle, pixel-identical
- [ ] Hero videos, animations, responsive behavior, fonts, colors
- [ ] Contact form → validation, anti-spam (honeypot + rate limit), stored in DB, email notification
- [ ] Careers: job offers list (from DB), application form with **file upload (CV)**, stored privately, email notification
- [ ] SEO (section 6), 404 and error pages in the same design

**Admin (`/admin`)**
- [ ] Login/logout, session handling, lockout after failed attempts, change password
- [ ] Tabs: Content, Job offers (CRUD, publish/unpublish), Settings, Messages (contact + applications: read/unread, delete, export, authenticated CV download)
- [ ] Visual Editor with all abilities from section 6
- [ ] Control of texts, links, images, videos, section visibility and order, on all pages, EN and FR
- [ ] Media library with secure uploads

**Portfolio (`/portfolio`)**
- [ ] Public portfolio: images, videos, YouTube embeds, contact boxes, services with prices, social networks, same look
- [ ] Admin tabs for all of the above. If the legacy has a separate admin password, unify under the main admin auth (document the change in the parity matrix and tell me)

**Quotes (`/devis`)**
- [ ] Login, roles `admin` / `sales`, agent management
- [ ] Service catalog CRUD (soft delete), `/public/services`
- [ ] Quote creation with catalog and custom lines, recurring lines, free packs, split billing, contract duration, discount, currency, VAT modes (`split`/`advance`/`none`), payment plans
- [ ] Statuses, client approval, payment tracking, retainer collections
- [ ] **PDF generation identical to the Python output** (layout, logo, wording, number formats)
- [ ] Android app (Capacitor) builds, HTTPS only (remove `cleartext: true`), CI workflow kept and cleaned
- [ ] Money logic port with exact rounding rules of the legacy `payments.py` (e.g., first installment rounded to the nearest 10), covered by unit tests

---

## 8. Security checklist (acceptance-blocking)

- [ ] Passwords hashed with **argon2id**; password policy; constant-time comparison; no default admin password: first admin created via a CLI seed script reading `.env`
- [ ] Sessions: `httpOnly`, `Secure`, `SameSite`, `__Host-` prefix where possible, rotation on login, idle + absolute timeout, server-side revocation
- [ ] CSRF protection on every state-changing request
- [ ] `helmet` with a strict CSP (nonces for SSR inline scripts), HSTS (at nginx), `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`, frame protection
- [ ] Rate limiting: login (strict + lockout), contact, applications, uploads, API globally; also at nginx level
- [ ] Input validation with **zod on every endpoint** (body, query, params); reject unknown fields
- [ ] NoSQL injection protection (sanitize operators, never pass raw request objects to Mongoose queries)
- [ ] HTML sanitization for any rich text, output encoded by React; no `dangerouslySetInnerHTML` without sanitization
- [ ] **Uploads**: real MIME detection (magic bytes), extension allowlist, size limits per type, random file names (UUID), images re-encoded with `sharp`, never executable, stored outside the web root, served with `nosniff` and safe `Content-Disposition`; **CVs and applications are private** (authenticated admin download only); path traversal impossible
- [ ] CORS strict allowlist from env (no `*`); same-origin by default
- [ ] Authorization checks on every admin/sales route (role-based), including object-level checks
- [ ] Secrets only via env; `.env.example` with placeholders only; startup fails fast if required secrets are missing or weak
- [ ] Mongo: auth enabled, app user with least privilege, port **not** published to the host
- [ ] Logging with redaction (no passwords, tokens, CV content); audit log for admin actions
- [ ] Docker: non-root user, minimal images, read-only filesystem where possible, healthchecks, resource limits
- [ ] Dependency audit clean or documented; gitleaks pre-commit + CI check
- [ ] `docs/security.md` documenting threats, controls, and a pre-deployment checklist

---

## 9. Docker and Git requirements

**Docker**
- `docker-compose.yml` (prod-like): services `mongo`, `api` (API + SSR), `nginx`, `backup`; named volumes for DB, uploads (public media), private uploads (CVs), backups, and certs.
- `docker-compose.dev.yml`: hot reload for web and api.
- Run with **`docker compose up --build`**; document `down`, logs, reset, restore-from-backup commands.
- Healthchecks, restart policies, `.env.example` for root and each app, no secret baked into images.
- nginx: reverse proxy, gzip/brotli, long cache for hashed assets, correct video range requests, upload size limit, TLS-ready config (documented with certbot or Caddy), HTTP→HTTPS redirect in prod.
- Backup: scheduled `mongodump` with retention + uploads volume backup + documented restore, tested.

**Git**
- `git init` fresh repo, first commit = scaffold only. `.gitignore` covers `.env*` (except `.env.example`), `node_modules`, `dist`, `legacy/`, uploads, dumps, backups, Android build artifacts, IDE files.
- `README.md`: what it is, prerequisites, one-command start, env vars table, project structure, scripts, troubleshooting.
- Branching and commit conventions from section 0. Tag `v0.1.0` after Phase 12.

---

## 10. Step-by-step plan (frontend first)

**Phase 0 — Audit and inventory (no code yet)**
Read `legacy/` fully. Produce `docs/legacy-audit.md`: every page, section, editable element, route, API endpoint, DB table/columns, visual-editor mechanism, `blocks` keying scheme, portfolio and devis features, PDF layout, third-party services, and a list of secret locations (names only). Produce `docs/parity-matrix.md` (every feature, status). List open questions in `docs/QUESTIONS.md`. **STOP.**

**Phase 1 — Repo, monorepo, tooling, Docker skeleton**
`git init`, pnpm workspace, TS configs, lint/format/test, gitleaks hook, `.gitignore`, `.env.example`, README skeleton, minimal compose (mongo + api hello + nginx) that starts with one command. **STOP.**

**Phase 2 — Public website frontend (mock data)**
Extract design tokens and assets. Rebuild the 5 pages with `Cms*` components, EN/FR toggle and `/fr` `/en` routing, SSR working with the `MockRepository`. Add Playwright **visual regression** comparing each page (desktop, tablet, mobile; EN and FR) against screenshots of the legacy pages; report the diff and fix until visually identical. **STOP.**

**Phase 3 — Admin and Visual Editor frontend (mock)**
Login screen (mock), dashboard tabs (Content, Jobs, Settings, Messages), Visual Editor overlay with all abilities, section visibility/order UI, media picker. Works against mock repositories; state persisted in memory/localStorage for demo only. **STOP.**

**Phase 4 — Portfolio frontend + its admin tabs (mock)**
Pixel-identical public portfolio and admin tabs. **STOP.**

**Phase 5 — Quotes frontend (mock)**
Rebuild `devis-web` from `legacy` sources (keep behavior and design), Capacitor config cleaned, mock API client. **STOP.**

**Phase 6 — Backend core**
Express app, config validation, Mongo connection, Mongoose models, session auth, argon2, CSRF, helmet, rate limits, error handling, logging, zod middleware, health endpoints, seed CLI for first admin. Tests. **STOP.**

**Phase 7 — CMS API + SSR with real data**
Content, sections, ordering, settings, jobs, portfolio APIs; swap `MockRepository` → Mongo-backed repositories; SSR from DB; cache + invalidation; SEO endpoints (sitemap, robots, redirects). Visual Editor connected end to end. **STOP.**

**Phase 8 — Messages, careers, uploads, email**
Contact and application endpoints, private CV storage, secure upload pipeline, Nodemailer notifications, admin Messages tab (read/unread/delete/export/download). Security tests for uploads. **STOP.**

**Phase 9 — Quotes backend port + parity**
Port models, money/VAT/payment-plan logic (into `packages/shared` with unit tests mirroring `payments.py`), quote lifecycle, approvals, payments, retainers, roles, catalog, PDF generation. **Parity tests**: same sample quotes produce the same amounts and a visually matching PDF versus the Python output (render both to images and compare; document tolerances). **STOP.**

**Phase 10 — Data migration**
Idempotent scripts importing from the sanitized SQL export and `uploads/`: content/`blocks` (using the mapping table), jobs, portfolio, settings, messages, applications, files. Dry-run mode, counts report, rollback instructions. Admin accounts created fresh. **STOP.**

**Phase 11 — Production Docker, nginx, backups**
Prod compose, multi-stage images, nginx hardening, TLS-ready, backup sidecar + restore test, `docs/deployment.md` for a generic Ubuntu VPS with Docker, including the **ATI VPS checklist** (root/SSH access, ≥2 GB RAM (4 GB preferred), fixed public IP, ports 80/443 open, DNS A records for `zayerdigital.tn`, firewall, unattended upgrades, fail2ban). **STOP.**

**Phase 12 — Hardening, QA, release**
Full security checklist pass, `pnpm audit`, E2E tests (Playwright: public flows, admin login, edit-and-publish, uploads, quote → PDF), Lighthouse/SEO check, accessibility pass without changing the look, final parity matrix review, README polish, tag `v0.1.0`. **STOP and report.**

---

## 11. Acceptance criteria (the project is done only when all are true)

1. `cp .env.example .env` (fill placeholders) then `docker compose up --build` starts the full stack; no manual steps besides creating the first admin via the documented CLI.
2. All five public pages are **visually identical** to legacy in EN and FR on desktop/tablet/mobile (Playwright visual diffs within a documented small tolerance, zero unexplained differences).
3. `docs/parity-matrix.md` shows **every** legacy feature as done and verified, with any deliberate deviation listed and approved by me.
4. From the admin, I can change any text, link, image, video, visibility and order on any page in EN and FR, and the public site (SSR HTML) reflects it immediately.
5. All legacy `blocks`, jobs, portfolio items, settings, messages, applications, and uploads are migrated, with a counts report that matches the source.
6. Quotes: money calculations match the legacy logic to the cent, PDFs match the legacy layout, Android app builds over HTTPS only.
7. Every item in the security checklist is implemented and verified (tests or documented manual proof); `pnpm audit` has no unresolved high/critical issues.
8. Git history is clean (Conventional Commits, phase branches), `gitleaks` finds nothing in the full history, `.env`, dumps, `legacy/` and uploads are absent from the repo.
9. Backup and restore have been demonstrated end to end.
10. README and `docs/` let me redeploy on a fresh VPS by copy-pasting commands.

---

## 12. Start now

Do **Phase 0 only**. Before reading anything, confirm that `legacy/` exists and contains the site ZIP content, the quotes platform sources, and (if provided) the SQL export and `uploads/`. If something is missing, ask me. Then produce the Phase 0 deliverables and stop.
