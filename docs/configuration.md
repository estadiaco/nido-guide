# Configuration reference

Every variable and knob you set to run this project, in one place, so nothing is buried in
code. Grouped by where it's set.

> **Two repos.** The system is split across two repositories:
> - **`nido-guide`** (this repo, public) — the tablet guide. Hosted on GitHub Pages at
>   **https://nido.estadiaco.com**.
> - **`nido-dashboard`** (private) — the owner dashboard + backend. Hosted on Vercel at
>   **https://app.estadiaco.com**.
>
> Sections 1 and 3 below are configured in the **`nido-dashboard`** repo; section 2 is this
> (tablet) repo.

## 1. Dashboard environment variables

Set in the **`nido-dashboard`** repo — locally in its `.env.local` (gitignored) and, for
production, in the Vercel project's environment settings. A fill-in template with the same
names lives in that repo's `.env.example`.

| Variable | Kind | Used by | What it does | Where to get it |
|---|---|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | public | dashboard (browser + server), provision script | Your Supabase project's API URL | Supabase → Project Settings → API → Project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | public | dashboard (browser) | Browser access under row-level security. Public by design | Supabase → Project Settings → API → anon public |
| `SUPABASE_SERVICE_ROLE_KEY` | **secret** | publish route (server), provision script | Full-access key that **bypasses** security rules. Writes published files, provisions properties. Never commit or expose | Supabase → Project Settings → API → service_role |
| `OPERATOR_USER_IDS` | **server-only** | provision route | Comma-separated Supabase user ids allowed to provision properties. No `NEXT_PUBLIC_` prefix | The user ids from Supabase → Authentication → Users |
| `NEXT_PUBLIC_TABLET_BASE_URL` | public | dashboard (browser) | Base URL of the deployed guide; builds the full tablet URL shown after publishing. Defaults to the production tablet URL (`https://nido.estadiaco.com/`) if unset | Your deployed guide's base URL |

**Prefix rule:** `NEXT_PUBLIC_` means the value is sent to the browser (fine for public
values). No prefix means server-only — that's why the service-role key and the operator
allow-list have none. On Vercel, set these in the project's env settings, keeping the
secrets server-only.

## 2. Tablet app knobs (this repo)

| Where | Name | What it does |
|---|---|---|
| `index.html` URL | `?p=<key>` | **Preferred** — the short tablet URL, where `<key>` is the property's serving key: its unguessable `public_id` by default, or its readable `slug` if `url_key_mode='slug'`. The guide expands it to `<publishBase>/<key>/config.json` using `settings.json`. Set as the tablet's start URL (Fully Kiosk). |
| `index.html` URL | `?config=<url>` | Escape hatch — load a config from an explicit URL (wins over `?p=`). **Restricted to same-origin** (a cross-origin URL is ignored). Defaults to the in-repo `config.json` when neither is given. |
| `settings.json` | `publishBase` | The Supabase Storage folder that holds published files — the one place the Supabase URL lives (config, not code). Change it only if the Supabase project changes. Same-origin with the guide and precached for offline. |
| `service-worker.js` | `CACHE_VERSION` | Cache-buster (currently `v13`). Bump it (`v13` → `v14` …) after editing `index.html`, `settings.json`, or a theme, so tablets discard the old cached shell. See `service-worker.js:2`. |
| `index.html` | `POLL_INTERVAL_MS` | How often the tablet re-checks its published config while online and reloads if it changed — currently **60 s** (a named constant near the poller in `index.html`; lower it to ~30 s for snappier updates, at a bit more egress). |
| `config.json` | `theme` | Which theme (visual look) the guide uses. Picked in the dashboard; a `themes/*.css` path (validated both server-side on publish and on the tablet). |
| `config.json` | `layout` | Which layout (tab structure) the guide uses (`standard`, `rules-as-tab`, …). Picked in the dashboard; absent → `standard`. Defined in `index.html`'s `LAYOUTS` registry. |
| `config.json` | `property.*`, `en`/`es` | The guide's text content (guest name, wifi, rules…). See the main [README](../README.md). |

## 3. Supabase project settings (one-time, not in code)

- Run the schema from the **`nido-dashboard`** repo (`supabase/schema.sql`) — creates the
  `properties` table, its row-level-security rules, and the public-read `published` bucket.
- **Auth → Providers → Email → Confirm email:** your preference (off is convenient for
  testing; on for real use).

## 4. Standing up a fresh environment — checklist

1. **Supabase:** create a project; run `nido-dashboard/supabase/schema.sql`; set email
   confirmation as desired.
2. **Dashboard (`nido-dashboard`):** fill its `.env.local` from `.env.example`; deploy to
   Vercel and set the same env vars there (secrets server-only).
3. **Tablet (`nido-guide`):** deploy on GitHub Pages; confirm `settings.json` `publishBase`
   points at the Supabase project; set `NEXT_PUBLIC_TABLET_BASE_URL` in the dashboard to the
   guide's base URL.
4. **Provision** a property (operator panel), have the owner **publish**, and point the
   tablet's start URL at `https://nido.estadiaco.com/?p=<slug>`.

Related (in the `nido-dashboard` repo): `docs/storage.md` (how published files are stored
and served), `docs/security.md` (isolation, secrets), `docs/operator-provisioning.md`,
`docs/installer-runbook.md`.
