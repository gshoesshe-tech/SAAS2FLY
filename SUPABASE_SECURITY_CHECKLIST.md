# Supabase Security Checklist (Template)

> This checklist is based on table usage found in `app.js`. It does **not** assume your exact production schema. Adjust all SQL and policy conditions to match your real column names and relationships.

## Core security principles

1. Frontend role checks are UX-only. Real enforcement must be in Supabase RLS + SQL policies.
2. Customer/business data must never be publicly readable.
3. Every table with `business_id` should be scoped to that business membership.
4. Super-admin (`platform_owner`) actions must be allowed only by policy, not by hidden UI routes.

## Tables used by the app

From current code paths, the app queries or mutates these tables:

- `business_templates`
- `businesses`
- `business_members`
- `business_settings`
- `products`
- `services`
- `orders`
- `order_items` (backup/export list)
- `inventory_movements` (backup/export list)
- `expenses`
- `payments`
- `invoices`
- `revenue_goals` (backup/export list)
- `platform_payments`

## Minimum RLS target per table

### `business_members`
- Users can read their own member rows (`user_id = auth.uid()`).
- Only business admins/owners can add team members for their own business.
- Prevent privilege escalation (e.g., setting own role to `platform_owner`) via policy and/or secure function.

### `businesses`
- User can read businesses where they are active members.
- User can update only businesses where their role allows it.
- `status` updates (`active`, `suspended`) should be limited to platform owner/admin policy.

### `business_settings`
- Read/write only for members of that `business_id` with sufficient role.
- Block cross-business updates.

### `business_templates`
- Safe as read-only public or authenticated read-only if templates are non-sensitive.
- Write operations restricted to platform owner/admin only.

### `orders`, `order_items`, `products`, `services`, `inventory_movements`, `expenses`, `payments`, `invoices`, `revenue_goals`
- Read/write restricted to users who belong to the same `business_id`.
- Optional: role-based write restrictions (viewer read-only, staff limited writes, etc.).
- No anonymous/public access.

### `platform_payments`
- Business users can create/submit their own payment proofs for their own business.
- Approval fields (`approved_by`, `approved_at`, status transitions like `approved`) only by platform owner policy.
- No public read.

## Example policy templates (adjust to your schema)

```sql
-- TEMPLATE helper: check active membership in a business
create or replace function public.is_business_member(_business_id uuid)
returns boolean
language sql
stable
as $$
  select exists (
    select 1
    from public.business_members bm
    where bm.business_id = _business_id
      and bm.user_id = auth.uid()
      and bm.is_active = true
  );
$$;
```

```sql
-- TEMPLATE helper: check platform owner role
create or replace function public.is_platform_owner()
returns boolean
language sql
stable
as $$
  select exists (
    select 1
    from public.business_members bm
    where bm.user_id = auth.uid()
      and bm.role = 'platform_owner'
      and bm.is_active = true
  );
$$;
```

```sql
-- TEMPLATE: orders table scoped by business membership
alter table public.orders enable row level security;

create policy "orders_select_own_business"
on public.orders
for select
using (public.is_business_member(business_id));

create policy "orders_insert_own_business"
on public.orders
for insert
with check (public.is_business_member(business_id));

create policy "orders_update_own_business"
on public.orders
for update
using (public.is_business_member(business_id))
with check (public.is_business_member(business_id));
```

```sql
-- TEMPLATE: platform-only approval on businesses
alter table public.businesses enable row level security;

create policy "businesses_platform_owner_can_suspend_or_activate"
on public.businesses
for update
using (public.is_platform_owner())
with check (public.is_platform_owner());
```

```sql
-- TEMPLATE: hide sensitive data from anonymous/public
revoke all on public.orders from anon;
revoke all on public.payments from anon;
revoke all on public.invoices from anon;
revoke all on public.expenses from anon;
```

## Verification checklist

- [ ] RLS enabled on every table listed above.
- [ ] `anon` cannot read customer/order/payment tables.
- [ ] Cross-business read/write tests fail.
- [ ] Viewer/staff/admin role behavior tested.
- [ ] Platform-owner-only actions enforced by policy.
- [ ] Backup queries only return authorized rows.
