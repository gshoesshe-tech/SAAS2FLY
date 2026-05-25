# Backup Plan (Foundation)

This document defines the staged plan for a production-grade backup system without changing the current UI flow.

## Current state

- The app already has client-side JSON/CSV export actions in the super-admin screen.
- This is useful as a temporary owner tool but is **not** a complete backup strategy.
- Real security and access control must be enforced by Supabase RLS policies, not frontend route visibility.

## Goals

1. Daily automatic database backups.
2. Downloadable backups by date range.
3. Owner-only access.
4. Internal-only operations (customers should not see backup features).
5. Coverage for:
   - orders
   - customers (if separate table exists)
   - payments
   - invoices
   - expenses
   - products
   - services
   - team/business members
   - business settings

## Recommended phased implementation

### Phase A — Harden existing export surface (now)
- Keep current export functions.
- Restrict display to super-admin UI only (already present).
- Add explicit code comments: UI checks are convenience only; RLS is real protection.

### Phase B — Scheduled backups (future backend work)
- Add Supabase Edge Function or external job runner.
- Run daily (UTC schedule) and export snapshots to secure object storage.
- Include metadata: run time, row counts, checksum, schema version.

### Phase C — Owner download portal
- Add owner-only list of backup snapshots (date/time, size, status).
- Enable filtering by date range.
- Generate signed URL downloads with short expiry.

### Phase D — Restore & audit readiness
- Add restore runbook (manual process first).
- Log who initiated exports/downloads/restores.
- Add retention rules (e.g., daily x 30, weekly x 12, monthly x 12).

## Security requirements

- Enforce owner permissions with RLS/policies and secure functions.
- Never rely on hidden links/routes/buttons.
- Do not expose backup metadata to regular business users.
- Encrypt backup storage at rest and in transit.

## Out of scope for current repo phase

- No server-side automation added yet.
- No restore endpoint added yet.
- No schema-altering migration included in this phase.
