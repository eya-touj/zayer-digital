# Legacy audit

Audit date: 2026-10-20

Scope: read-only inspection of `legacy/site/`, `legacy/devis/`, and `legacy/db/`. The legacy applications were not executed and the Render service was not contacted. Files ending in `.bak` were excluded from the source audit as instructed. Secret values are intentionally omitted.

## Inventory and limits

| Location | What is present | Audit note |
|---|---|---|
| `legacy/site/zayerdigital.tn/htdocs/` | Static HTML site, PHP portfolio, PHP admin/API, compiled `/devis` bundle, video and image assets | Five HTML pages are non-empty. `formation.html` and `international.html` are zero bytes. |
| `legacy/devis/zayer-devis-platform-main/` | React/Vite frontend, Capacitor Android project, FastAPI/SQLAlchemy backend, logo, APK workflow | `backend/app/pdf_generator.py.bak` and `frontend/src/api.js.bak` were ignored. |
| `legacy/db/if0_43086171_zayer_cms.sql` | Seven MySQL `CREATE TABLE` definitions | No `INSERT` statements or row data. The file defines `admins` and `rate_hits` schemas, but contains no records. |
| `legacy/site/zayerdigital.tn/htdocs/uploads/` | `.htaccess` only | No uploaded media or CV files were supplied. |
| `legacy/site/zayerdigital.tn/htdocs/logo-email.png` | Email logo image | No Zimbra email-signature HTML file was found. |

The repository `.gitignore` ignores `legacy/`; `git check-ignore` confirmed that the site, quotes source, and SQL file are ignored. No `.env` files are present under `legacy/`.

## Public site

The five non-empty pages are standalone HTML files. Each has an inline EN/FR translation dictionary and a language toggle. The files declare `lang="en"`, show EN as active by default, and use `.html` paths rather than language-prefixed URLs. The home page also links to the empty `international.html` and `formation.html` files.

| Page / route | Sections and notable content | Language dictionary keys | Media and forms |
|---|---|---:|---|
| `/` (`index.html`) | Hero, Why Us, Process, CTA banner; shared nav/footer | 39 EN + 39 FR | Hero video `hero-1.mp4`; nav includes International and Training links. |
| `/about.html` | Page hero, mission, values, team, timeline, CTA; shared nav/footer | 45 EN + 45 FR | No page-local media found. |
| `/services.html` | Page hero, six service cards, CTA banner; shared nav/footer | 21 EN + 21 FR | Hero video `hero-2.mp4`. |
| `/careers.html` | Page hero, culture/perks, job filters/list, open-application section; shared nav/footer | 34 EN + 34 FR | Six job records are hard-coded in page JavaScript. CV upload uses Uploadcare; applications post to Formspree. |
| `/contact.html` | Page hero containing contact details and message form; shared nav/footer | 26 EN + 26 FR | Contact form posts to FormSubmit. |
| `/international.html` | Empty file | — | Zero bytes; linked from home navigation. |
| `/formation.html` | Empty file | — | Zero bytes; linked from home navigation. |

Shared navigation links include Home, About, Services, Portfolio, Careers, and Contact. The home page adds International and Training. There is no `404` page in the supplied tree. The HTML contains no canonical, `hreflang`, Open Graph/Twitter metadata, or meta description. No site-level redirects, `sitemap.xml`, or `robots.txt` were found.

### Design and assets

The five public pages and portfolio use CSS variables `--navy: #0a1628`, `--navy-mid: #122040`, `--navy-light: #1e3a6e`, and `--gold: #c8a96e` (plus white, off-white, and grey tokens). Their page styles load Playfair Display and DM Sans from Google Fonts. The admin has a separate palette including `#12295a` / `#c8a96e`, and loads Bricolage Grotesque, Inter, and Playfair Display. The quotes PDF uses a third palette, recorded below.

Files include `hero-1.mp4`, `hero-2.mp4`, the admin intro video, four contact icons, three social icons, and `logo-email.png`. No local font files or reference screenshots were found. The home page defines a `fadeUp` keyframe; several pages use `.reveal` elements with `IntersectionObserver`. The public pages have responsive breakpoints at 768px and, on some pages, 1024px.

## Site admin, CMS, and PHP API

`/admin/` is a single HTML/JavaScript application. Its tabs are Visual Editor, Content, Job Offers, Settings, and Messages. Content rows support page, section, order, visibility, bilingual title/subtitle/body, metadata, image URL, and link URL. The Content form's page options include `home`, `services`, `about`, `careers`, `contact`, and `portfolio`; the Visual Editor iframe list instead uses `index`, `about`, `services`, `careers`, and `contact`. Jobs support department, bilingual title/description, location, experience, order, and visibility. Messages can be viewed, marked `new` / `read` / `done`, and deleted. The source does not show message export or authenticated CV download, nor a password-change screen.

The admin uses session authentication through `/api/index.php?r=login`, sends the returned CSRF token in `X-CSRF`, rotates the PHP session ID on login, and uses `password_verify` against `admins.password_hash`. Its cookie is configured `Secure`, `HttpOnly`, and `SameSite=Lax`; the API has a 2-hour idle timeout. Login and public-message rate limits are 5 attempts per IP in 15 minutes and 5 messages per IP in 10 minutes, respectively.

The Visual Editor expects an iframe page to load `assets/cms.js`. The script and editor use these selectors/protocols:

| Legacy selector / action | Meaning |
|---|---|
| `data-cms="<bkey>"` | Localized text/rich text block. |
| `data-cms-href="<bkey>"` | Link target associated with a block. |
| `data-cms-src="<bkey>"` | Image/video source associated with a block. |
| `?cms-edit=1` | Enables pick-and-edit mode inside the admin iframe. The iframe posts selected key, element kind, HTML, href/src, page, and language to the parent. |
| `zayer-cms-refresh` | Parent-to-iframe message to reload and reapply saved block values. |

The source key convention accepts keys matching `[a-z0-9_.-]{1,120}`; the API stores the exact `bkey` and derives `page` from the prefix before the first dot when saving. The block-to-new-key mapping available from source is:

| Legacy block representation | New CMS representation |
|---|---|
| `bkey` | Preserve the suffix as the stable key. The target's canonical homepage slug is `home`, while the legacy Visual Editor uses `index`; proposed mapping is `index.<suffix>` → `home.<suffix>`. Other page prefixes can be preserved. |
| Prefix before the first `.` / `page` | `page` field; map legacy `index` to target `home`. The Content tab already uses `home`, so the legacy editor and content-row page labels are inconsistent. |
| `value_en`, `value_fr` | `value.en`, `value.fr`. |
| `href`, `src`, `alt` | Corresponding link/media/alt fields. |
| `visible` | `hidden = (visible == 0)`. |
| `updated_at` | `updatedAt`. |
| Selector-derived block type | The legacy row has no `type`; infer text from `data-cms`, media from `data-cms-src`, and link support from `data-cms-href` plus row fields. |
| Original content | Not stored in the row. The editor relies on the source HTML/schema as the original and resets by deleting the override. The schema JSON is missing in this archive. |
| Ordering / updater | No block columns exist for order or `updatedBy`; `items.sort_order` is a separate content-row field. |

This is a structural mapping only. The supplied SQL has no block rows, `seed-content.json` is absent, and none of the five public HTML pages contains a `data-cms` / `data-cms-href` / `data-cms-src` attribute or a `cms.js` reference. Therefore the supplied snapshot does not provide concrete legacy block keys or a page-element-to-key mapping; the editor cannot pick elements from these HTML files as supplied. The public pages also do not call the PHP content API. `items` and `jobs` are returned by the API, but no API integration was found in the static pages.

### PHP API routes

All routes use `/api/index.php?r=...`; state-changing admin calls use `X-CSRF`.

| Route | Method | Behavior / access |
|---|---|---|
| `public/content` | GET | Returns visible `items`, visible `jobs`, all `settings`, and `blocks` when the table is available. |
| `public/message` | POST | Honeypot and rate limit; validates name/email/body and writes `contact` or `application` to `messages`. The public contact/careers forms do not call this route. |
| `login`, `logout`, `me` | POST, POST, GET | Admin session login/logout/current-session check. |
| `items`, `jobs` | GET, POST | Admin list/create/update/delete; `items` can filter by page and section. |
| `messages` | GET, POST | Admin list/filter, status update, delete; no create operation. |
| `settings` | GET, POST | Admin list/upsert bilingual setting values; delete uses `a=delete`. |
| `schema` | GET | Admin-only; reads `seed-content.json`, or returns an empty array if absent. |
| `blocks` | GET, POST | Admin-only block list/upsert/delete; delete restores the HTML original by removing the override. |
| `upload` | POST multipart | Admin-only, 8 MiB limit, MIME sniffing for JPEG/PNG/WebP/GIF/MP4/PDF, random name; files go to public `htdocs/uploads/`. |

The API sanitizes block HTML/URLs, restricts upload types, and places a `.htaccess` in uploads to disable script execution and set `nosniff`. Image files are checked with `getimagesize`, but are not re-encoded. The API's upload endpoint allows PDF, while the Visual Editor picker offers image/video types. The API config key names are listed under credential locations.

## MySQL schema export

The export is schema-only: there are no row values to audit or migrate from this file.

| Table | Columns |
|---|---|
| `admins` | `id`, `name`, `email`, `password_hash`, `created_at` |
| `blocks` | `bkey`, `page`, `value_en`, `value_fr`, `href`, `src`, `alt`, `visible`, `updated_at` |
| `items` | `id`, `page`, `section`, `sort_order`, `visible`, `title_en`, `title_fr`, `sub_en`, `sub_fr`, `text_en`, `text_fr`, `meta`, `image_url`, `link_url`, `updated_at` |
| `jobs` | `id`, `sort_order`, `visible`, `dept`, `title_en`, `title_fr`, `location`, `experience`, `description_en`, `description_fr`, `updated_at` |
| `messages` | `id`, `kind`, `name`, `email`, `phone`, `subject`, `body`, `file_url`, `status`, `created_at` |
| `rate_hits` | `id`, `kind`, `ip`, `at` |
| `settings` | `skey`, `value_en`, `value_fr`, `updated_at` |

The SQL file defines `admins` and `rate_hits` tables but contains no records for them. Admin accounts remain designated for fresh creation in the rebuild. The PHP portfolio has four additional tables created by its own PHP file; those are not in this SQL export.

## Portfolio

`/portfolio/` resolves to `portfolio/index.php`. The public page contains three media tabs (`creative`, `web`, `impact`) for images, videos, and YouTube embeds; website links/categories; service descriptions; a free-quote calculator; contact boxes; and social links. It has no language toggle in the supplied source; public copy mixes English navigation with French section content. Its quote calculator requests the active service catalog from the Render API at `/public/services`.

Admin mode is enabled by `?admin=1` on the portfolio route and has its own `ADMIN_PASSWORD` constant. It supports adding/editing/deleting portfolio media and website entries, contact boxes, social links, and priced service items, plus visibility/order controls. The login compares the submitted value directly to that constant; no password hash or CSRF token was found, and the delete actions use GET links. This separate password is a parity deviation to unify under the new admin login.

The PHP file creates these tables at runtime:

| Portfolio table | Columns |
|---|---|
| `portfolio_items` | `id`, `tab_type`, `title`, `media_url`, `link_url`, `category`, `sort_order`, `created_at` |
| `contact_boxes` | `id`, `label`, `value`, `visible`, `sort_order` |
| `service_items` | `id`, `label`, `price`, `visible`, `sort_order` |
| `social_items` | `id`, `label`, `icon`, `url`, `visible`, `sort_order` |

The handler allows `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.mp4`, `.mov`, and `.webm`, up to 20 MiB, and names files with `uniqid`. It validates extensions but does not perform MIME sniffing or image re-encoding. The uploaded media lives under the public `portfolio/uploads` directory. The PHP source seeds defaults when these tables are empty, but there are no portfolio rows or uploads in the supplied export/tree.

## Quotes platform (`/devis`)

The local source is a separate project at `legacy/devis/zayer-devis-platform-main/`; the site tree also contains a compiled Vite bundle under `htdocs/devis/`. The source frontend is React 18 + Vite, with Capacitor Android. The backend is Python 3.11.9, FastAPI, SQLAlchemy, and SQLite locally / PostgreSQL by `DATABASE_URL`; it is deployed separately on Render per the supplied note. The frontend calls a configurable `VITE_API_URL` and stores the JWT in `localStorage`.

The frontend has login, New Quote, My Quotes, and admin-only Agents tabs. Quote creation supports catalog/custom lines, a discount, TND/EUR/USD, optional PDF narrative fields, recurring retainers, free packs, startup-pack duration, per-month split billing, contract duration, and payment plans. Lists support client search/grouping and admin agent filtering. Users can update quote status, approval state, and payment state; they can download PDFs and mark recurring collections collected. Admins can create sales agents. The README says catalog CRUD is currently through FastAPI `/docs`, not a frontend tab.

| Method and path | Access / behavior |
|---|---|
| `GET /` | Health/status. |
| `POST /auth/register`, `POST /auth/login`, `GET /auth/me` | Setup-key-gated registration, JWT login, current user. |
| `GET /services`, `GET /public/services` | Authenticated catalog; public active catalog for the portfolio calculator. |
| `POST /services`, `PATCH /services/{service_id}`, `DELETE /services/{service_id}` | Admin catalog create/update/soft-delete. |
| `POST /quotes`, `GET /quotes`, `GET /quotes/{quote_id}` | Create/list/read quotes; sales users see their own quotes, admins can filter by agent. |
| `PATCH /quotes/{quote_id}/status`, `/approval`, `/payment` | Update quote lifecycle, client-approval value, and payment value. These are authenticated internal actions; no public client portal endpoint is present. |
| `GET /quotes/{quote_id}/pdf` | Authenticated PDF download, with object-level owner/admin check. |
| `GET /retainer-collections`, `PATCH /retainer-collections/{collection_id}` | List and mark recurring collections; owner/admin checks. |
| `POST /agents`, `GET /agents` | Admin-only agent creation and summary list. |

The SQLAlchemy tables and columns are:

| Entity | Columns |
|---|---|
| `users` (`User`) | `id`, `full_name`, `email`, `hashed_password`, `role`, `is_active`, `created_at` |
| `services` (`Service`) | `id`, `name`, `description`, `category`, `default_unit`, `default_price`, `is_active`, `created_at` |
| `quotes` (`Quote`) | `id`, `devis_number`, `client_name`, `client_responsable`, `client_extra`, `statut`, `proposal_title`, `proposal_subtitle`, `objectifs`, `pourquoi_zayer`, `fonctionnalites`, `phases`, `devise`, `tva_mention`, `remise_pct`, `paiement`, `notes_paiement`, `status`, `created_at`, `acompte_pct`, `acompte_montant`, `solde_montant`, `payment_plan`, `tva_mode`, `approval_status`, `approved_at`, `declined_at`, `payment_status`, `first_payment_at`, `final_payment_at`, `created_by_id` |
| `quote_line_items` (`QuoteLineItem`) | `id`, `quote_id`, `service_id`, `description`, `qte_label`, `qte_multiplier`, `unit_price`, `total`, `is_custom`, `is_recurring`, `monthly_price`, `start_delay_months`, `duree_incluse`, `split_bill`, `split_breakdown`, `contract_duration`, `is_free_pack` |
| `retainer_collections` (`RetainerCollection`) | `id`, `line_item_id`, `quote_id`, `month_number`, `due_date`, `amount`, `is_collected`, `collected_at` |

`objectifs`, `fonctionnalites`, `phases`, `paiement`, and `split_breakdown` hold serialized list data. The model also defines ORM relationships such as quote creator, quote line items, and collection-to-quote/line relations. No local quote database or data export is present.

Money rules in `backend/app/payments.py` use 19% VAT. A valid payment plan has one to four positive percentages summing to 100; otherwise it falls back to `acompte_pct`, then 100%. HT, VAT, and TTC are rounded to two decimals. For a multi-part plan, the first payment is rounded to the nearest 10 (half-up via `floor(x / 10 + 0.5)`) and capped at TTC; remaining parts are distributed by the remaining percentages, with the final part adjusted to make the sum exact. `advance` charges all VAT with the first payment; `none` omits VAT. Package line totals use `unit_price × duree_incluse` (or zero for a free pack); recurring packages require contract duration greater than included duration. Confirming approval creates one collection per recurring line/month. One inconsistency to verify during the port: the frontend shows any `is_free_pack` line as zero, while the backend's legacy/simple-line path only zeros a free pack when `duree_incluse` is positive.

PDF generation uses ReportLab, A4, margins 18 mm left/right, 14 mm top, 20 mm bottom, navy `#0F1B33`, gold `#B8935A`, the ZAYER logo, Times/Helvetica styles, and a footer with quote number/date/client/page. Sections include proposal title/subtitle, client/date/status, agency introduction, objectives and rationale, features and phases, service recap, price table with discount and VAT, payment schedule, recurring fee table, payment terms, and agency/client signatures. Amounts use spaces as thousands separators, trim `.00`, and append the currency. The route builds split-bill breakdown data, but the current PDF renderer does not consume that field; this needs confirmation against the deployed Render version before PDF parity work.

The `seed.py` script fills an empty service catalog with starter entries and creates an admin only when the seed email/password environment variables are supplied. The Android project has a GitHub Actions APK workflow. `capacitor.config.json` sets `cleartext: true`, and the Android manifest sets `usesCleartextTraffic="true"`; the rebuild requirement is HTTPS-only. Frontend `package-lock.json` and backend pinned requirements are present. Android template example tests are present, but no quote money/PDF parity tests or frontend/backend feature tests were found.

The source uses JWT HS256 with a 60-minute default expiration and Argon2 password hashing. `JWT_SECRET_KEY` is checked for a minimum length. `ALLOWED_ORIGINS` defaults to `*`, registration falls back to a placeholder setup key if the environment variable is absent, and no rate-limit middleware is present in the source. The Render database contents are not in `legacy/`; whether to migrate existing quotes is explicitly open in `docs/QUESTIONS.md`.

## Third-party integrations

| Integration | Use found in source |
|---|---|
| Google Fonts | Public site, portfolio, admin fonts. |
| FormSubmit | Contact page form delivery. |
| Formspree | Career application delivery. |
| Uploadcare | Hosted CV upload widget on Careers. |
| Render | Quotes API and the portfolio's public service-catalog request. |
| YouTube | Portfolio embeds and thumbnails. |
| Facebook, Instagram, LinkedIn, WhatsApp | Social/contact links. |
| Vercel | README deployment instructions for the quotes frontend. |

## Secret and credential locations (names only)

No secret values are included here.

| File | Key/variable names found |
|---|---|
| `legacy/site/zayerdigital.tn/htdocs/api/config.php` | `db_host`, `db_name`, `db_user`, `db_pass`, `setup_key` |
| `legacy/site/zayerdigital.tn/htdocs/portfolio/index.php` | `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`, `ADMIN_PASSWORD` |
| `legacy/devis/zayer-devis-platform-main/backend/.env.example` | `DATABASE_URL`, `JWT_SECRET_KEY`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `ADMIN_SETUP_KEY`, `ALLOWED_ORIGINS`, `SEED_ADMIN_NAME`, `SEED_ADMIN_EMAIL`, `SEED_ADMIN_PASSWORD` |
| `legacy/devis/zayer-devis-platform-main/frontend/.env.example` | `VITE_API_URL` |
| `legacy/devis/zayer-devis-platform-main/backend/app/seed.py` | Reads `SEED_ADMIN_NAME`, `SEED_ADMIN_EMAIL`, `SEED_ADMIN_PASSWORD` from the environment. |

