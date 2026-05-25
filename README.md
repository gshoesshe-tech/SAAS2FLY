# SAAS2FLY

Business OS style SaaS frontend for small business owners (static HTML/CSS/JS + Supabase).

## Project Overview

This project provides:
- Landing page and pricing flow
- Auth + onboarding flow
- Business dashboard modules (orders, products, services, expenses, payments, invoices, reports, team)
- Template-driven business setup
- Super-admin area for approvals and backup exports

Current architecture is intentionally simple (no build pipeline) to keep deployment easy on static hosting.

## File Structure

- `index.html` — entry page, loads Supabase SDK + runtime config + app script.
- `styles.css` — complete UI styling for landing/auth/dashboard.
- `app.js` — main app logic (routing, rendering, Supabase interactions, modules, admin, exports).
- `config.example.js` — template config file for developers/deploy.
- `config.js` — runtime config file loaded by browser (keep real values out of git when possible).
- `SUPABASE_SECURITY_CHECKLIST.md` — RLS and policy checklist derived from table usage.
- `BACKUP_PLAN.md` — future backup architecture plan.
- `REFACTOR_PLAN.md` — safe incremental refactor roadmap.
- `assets/*.jpg` — payment QR assets used in checkout/verification UI.

## Local Setup

1. Serve files with any static server (examples):
   - `python -m http.server 8080`
   - or VS Code Live Server
2. Open `http://localhost:8080`.
3. Configure Supabase runtime config:
   - Copy `config.example.js` to `config.js`
   - Set:
     - `window.__APP_CONFIG__.SUPABASE_URL`
     - `window.__APP_CONFIG__.SUPABASE_ANON_KEY`

## Supabase Config Pattern (Static Hosting Safe Pattern)

This app uses runtime config from `config.js`:

```js
window.__APP_CONFIG__ = {
  SUPABASE_URL: "https://YOUR-PROJECT-REF.supabase.co",
  SUPABASE_ANON_KEY: "YOUR_SUPABASE_ANON_KEY"
};
```

Notes:
- Supabase anon keys are expected to be used in frontend clients.
- **Do not treat anon keys as secrets.**
- **Real security must be enforced by strict RLS policies and safe SQL permissions.**

## Netlify Deployment Steps

1. Connect repository to Netlify.
2. Build command: none (static app).
3. Publish directory: project root.
4. Ensure deployed site has a valid `config.js` with correct project values.
   - Option A: keep non-production values in repo `config.js` and replace during deployment pipeline.
   - Option B: generate `config.js` from Netlify env vars during deploy.
5. Verify routes that rely on hash fragments (`#landing`, `#app/home`) work without rewrites.

## Security Warning (Important)

Frontend checks (like hidden buttons or role-based page routing) are only UX controls.
They are **not** real authorization.

Use `SUPABASE_SECURITY_CHECKLIST.md` to enforce:
- RLS on every data table
- business scoping by membership
- owner/platform-only approvals
- no public reads of customer/order/payment data

## Backup / Export

Current app includes client-side export actions in super-admin UI:
- Full JSON backup
- Per-table CSV exports

These are temporary owner tools and depend on backend policy correctness.
See `BACKUP_PLAN.md` for production-grade direction:
- daily automated backups
- date-range backup downloads
- owner-only access controls
- auditable backup operations

## Future Roadmap

1. Incrementally split `app.js` following `REFACTOR_PLAN.md`.
2. Centralize error handling for Supabase operations.
3. Add backend automation for scheduled backups.
4. Add stronger role-based policy model and tests.
5. Add integration tests for critical business flows.
