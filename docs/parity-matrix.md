# Parity matrix

Phase 0 status: source inventory completed; rebuild implementation has not started. `Not started` means the new application has not implemented or verified that requirement yet. Legacy evidence below is based on source inspection, not a live-site visual run.

## Public site

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Home, About, Services, Careers, Contact pages | Five non-empty static HTML files are present. | Not started |
| EN/FR language toggle and routing | Five pages have inline EN/FR dictionaries; EN is initially active and routes are `.html` paths. New routing decision is `/fr` and `/en`, FR default. | Not started |
| Layout, colors, fonts, animation, responsive behavior | Public tokens are `#0a1628` navy and `#c8a96e` gold; public fonts are Playfair Display and DM Sans. Home has `fadeUp`; CSS has 768px/1024px breakpoints. | Not started |
| Hero videos | `hero-1.mp4` is used on Home; `hero-2.mp4` on Services. | Not started |
| Contact form validation, anti-spam, database storage, email notification | Contact form posts to FormSubmit. A separate PHP API endpoint exists, but the page does not call it. | Not started |
| Careers job list from database | Six jobs are hard-coded in Careers JavaScript. The API and admin have a jobs table, but the page does not call the API. | Not started |
| Careers application form and private CV upload | Careers posts to Formspree and uses Uploadcare. No private CV storage or API handoff was found. | Not started |
| SEO metadata, SSR, canonical, hreflang, social metadata, JSON-LD, sitemap, robots, redirects | No metadata/SSR/routes/SEO endpoints were found in the legacy site source. | Not started |
| 404 and same-design error pages | No 404 page or error-page asset was found. | Not started |

## Admin and CMS

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Login/logout and session handling | `/admin/` uses PHP sessions, password hashes, CSRF header, and session-ID rotation. | Not started |
| Lockout and password change | API has a per-IP login rate limit and delay after bad login; no password-change screen or account lockout state was found. | Not started |
| Content, jobs, settings, messages tabs | Admin tabs and PHP CRUD endpoints exist. The public static pages do not read `items`/`settings` from the API. | Not started |
| Messages: read/unread, delete, export, CV download | View/status/delete exist (`new` / `read` / `done`); export and authenticated CV download were not found. | Not started |
| Visual Editor on all public elements | Admin/editor and `cms.js` source exist, but current pages have no CMS selectors or script reference; `seed-content.json` and block data are absent. | Not started |
| EN/FR text, links, images/video, restore original | Block API/schema supports text, URL, media, visibility, and reset by deleting an override. No actual block rows/element keys are available in the supplied structure-only dump. | Not started |
| Section visibility and ordering | `items` supports visibility/order; block schema has visibility but no order field. No section reorder UI/API was found. | Not started |
| Secure media library/uploads | PHP API has MIME allowlist, 8 MiB limit, random filenames, and a no-execution `.htaccess`; files are public and images are not re-encoded. No media-library listing endpoint was found. | Not started |

## Portfolio

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Public media portfolio with images, videos, YouTube, links/categories | PHP page supports `creative`, `web`, and `impact` item tabs, videos/images/YouTube, categories, and links. | Not started |
| Contact boxes, services/prices, social networks | Four portfolio-specific tables and public sections exist in the PHP app. Their records are absent from the SQL export. | Not started |
| Portfolio admin controls | Separate `?admin=1` login can manage media, contact boxes, services, and social links with visibility/order. It uses an `ADMIN_PASSWORD` constant. | Not started |
| Unify portfolio admin under main admin | Legacy has a separate password; target requires unified auth. | Not started |
| Portfolio bilingual management | No language toggle was found; source mixes English navigation and French section copy. | Not started |

## Quotes

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Login and `admin` / `sales` roles | JWT login and role checks exist; Argon2 password hashing is used. | Not started |
| Agent management | Admin-only create/list endpoints and an Agents frontend tab exist. | Not started |
| Service catalog CRUD and `/public/services` | API supports catalog CRUD/soft delete and a public active-catalog endpoint; catalog admin UI is API-docs-only. | Not started |
| Quote creation: catalog/custom lines, discount, currency, VAT, payment plans | Source supports catalog/custom lines, TND/EUR/USD, discount, three VAT modes, one-to-four-part plans, and custom plans. | Not started |
| Recurring lines, free packs, split billing, contract duration | Fields and UI/API validation exist; per-month split data is not consumed by the current PDF renderer. Frontend/backend free-pack totals differ when included duration is zero. | Not started |
| Quote statuses and approval | Internal APIs support `draft` / `sent` / `accepted` / `rejected` and `pending` / `confirmed` / `declined`; no public client portal endpoint was found. | Not started |
| Payment tracking | Internal API supports `unpaid` / `deposit_paid` / `fully_paid` and payment timestamps. | Not started |
| Retainer collections | Confirmation generates monthly rows; UI lists due/overdue/collected entries and allows collection state updates. | Not started |
| ZAYER-format PDF | ReportLab renderer and logo are present; exact layout details are in `docs/legacy-audit.md`. Split-bill PDF parity needs comparison with the deployed Render version. | Not started |
| Android app, HTTPS-only, CI workflow | Capacitor project and APK workflow exist; `cleartext: true` is enabled in Capacitor and Android manifest. | Not started |
| Money/VAT rounding parity | `payments.py` contains the 19% VAT and nearest-10 first-installment rule. No parity tests were found in supplied sources. | Not started |
| Existing Render quote data migration | No quote database/export is in `legacy/`; user decision is pending. | Open question |

## Architecture, migration, and security acceptance

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Strict TypeScript MERN monorepo and shared Zod schemas | Legacy website is HTML/PHP; quotes are React JavaScript + Python. | Not started |
| Vite SSR through Express and typed repositories | Legacy pages are static HTML with client-side language switching; quotes frontend is a Vite SPA. | Not started |
| Frontend-first mock repositories | No mock repository layer found. | Not started |
| Single API and MongoDB with authentication | Legacy site uses MySQL; quotes API uses SQLite/PostgreSQL. | Not started |
| Migrate CMS content, jobs, portfolio, settings, messages, applications, media | SQL dump has no rows; portfolio tables and media are absent. Application delivery is external. | Not started |
| Migrate quotes | Source code only is local; live Render data is not in the archive and migration is undecided. | Open question |
| One-command Docker stack, nginx, backups, restore | No Docker or Mongo backup stack found. | Not started |
| Security checklist, audit logging, private CVs, hardened uploads | Legacy controls are partial; see source observations in `docs/legacy-audit.md`. | Not started |
| Dependency audit and gitleaks history scan | No new monorepo dependencies or scan configuration created in Phase 0. | Not started |

## Security acceptance checklist

| Requirement | Legacy evidence / finding | Rebuild status |
|---|---|---|
| Argon2id passwords, password policy, constant-time compare, no default admin | Site admin verifies password hashes; portfolio compares a password constant directly; quotes use Argon2. | Not started |
| Hardened sessions, rotation, idle/absolute timeout, revocation | PHP site rotates session ID and has a 2-hour idle timeout; quotes use 60-minute JWTs stored in browser localStorage. | Not started |
| CSRF on every state-changing request | Site admin API uses `X-CSRF`; portfolio actions have no CSRF control found; quotes use bearer-token API calls. | Not started |
| Helmet/CSP/HSTS/security headers | PHP JSON responses and uploads set `nosniff`; no strict CSP/HSTS/nginx policy exists in the supplied tree. | Not started |
| Rate limits across login, forms, uploads, API, and nginx | Site PHP limits login and messages only; portfolio uploads and quotes API have no rate limiter found. | Not started |
| Zod validation and unknown-field rejection on every endpoint | PHP validates selected fields manually; quotes use Pydantic; no shared Zod schemas. | Not started |
| NoSQL operator injection protection | Legacy persistence is MySQL/SQLite/PostgreSQL; no Mongo layer exists. | Not started |
| Sanitized rich text and safe output | PHP block writes use an allowlist sanitizer; React renders escaped text. | Not started |
| Secure uploads and private CV storage | Site API has MIME checks but stores public media; portfolio checks extensions only; Careers CVs go to Uploadcare. | Not started |
| Strict CORS allowlist | Quotes API defaults `ALLOWED_ORIGINS` to `*`; no site-wide CORS configuration was found. | Not started |
| Role and object-level authorization | Quotes API checks roles/quote ownership; site admin uses one admin session; portfolio has separate password gate. | Not started |
| Environment-only secrets and startup validation | Site config/PHP constants and quotes env variables are present; quotes API has a setup-key fallback in code. | Not started |
| Mongo auth, least-privilege user, unpublished port | No MongoDB deployment exists in the legacy source. | Not started |
| Redacted structured logs and admin audit log | No redacted structured logger or admin audit log was found. | Not started |
| Non-root/minimal/read-only Docker, health checks, limits | No Docker images or compose configuration were found. | Not started |
| Clean dependency audit, gitleaks hook/CI | No rebuild dependency audit or gitleaks setup has run. | Not started |
| Security documentation and deploy checklist | No security document was found in the supplied legacy tree; rebuild docs have not started. | Not started |

