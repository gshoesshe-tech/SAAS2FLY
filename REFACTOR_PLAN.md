# Safe Refactor Plan for `app.js` (No visual redesign)

This plan breaks refactoring into small, reversible steps while preserving behavior.

## Rules
- No UI redesign.
- No feature deletion.
- Keep hash routing behavior intact.
- Validate app after each step.

## Target structure
- `js/config.js`
- `js/supabaseClient.js`
- `js/state.js`
- `js/router.js`
- `js/utils.js`
- `js/api.js`
- `js/render/`
- `js/modules/`

## Incremental steps

1. **Config extraction (completed in this phase)**
   - Move runtime Supabase values to `config.js` + `config.example.js`.
   - Keep `app.js` behavior unchanged except config source.

2. **Utilities extraction**
   - Move pure helpers (`peso`, `today`, `esc`, `toast`, csv helpers) to `js/utils.js`.
   - Keep global function names unchanged to avoid breaking inline handlers.

3. **State extraction**
   - Move `state` object and related getters into `js/state.js`.

4. **Supabase client module**
   - Move client creation and session bootstrap into `js/supabaseClient.js`.

5. **Router module**
   - Move `route()` and hash utilities to `js/router.js`.

6. **API layer module**
   - Extract table operations into `js/api.js` with consistent error handling.

7. **Render separation**
   - Move dashboard/landing/auth render functions into `js/render/*` files.

8. **Feature modules**
   - Move domain modules (`orders`, `invoices`, `team`, `reports`, backups, admin) into `js/modules/*`.

## Validation checklist per step
- App loads without console runtime errors.
- Login/signup routing still works.
- Dashboard pages still render.
- CRUD save flows still work for unchanged modules.
- Super-admin gating still functional.
