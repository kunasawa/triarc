# TRIARC SOVEREIGN CONTROL ROOM — CANON MASTER BUILD

**Package:** TriArc Control Room — Canon Master

**Authority:** This ZIP is the single source of truth for the deployed Control Room stack.

**Included Spine:**
- Data Hardening (FIXED baseline)
- Identity & Authority
- Governance surfaces
- Publishing / Rollout policy
- Stripe Billing Authority (3-layer completion)
- Entitlement Enforcement Surface (Library / Weekly / Nexus)

**Commercial Doctrine (Locked):**
- **Academy** — $97 / month — Price ID: `price_1StpvJCnl3FbLIYRZynjlZYM`
- **Infirmary** — $147 / month — Price ID: `price_1Stpe3Cnl3FbLIYRS2OZmqk8`
- **Gauntlet** — $197 / month — Price ID: `price_1T0fafCnl3FbLIYRgQSeOd1t`

**Suite Binding (Locked):**
- Weekly Regime Classifier is part of the suite (not optional add-on).

**Playbook Access (Locked):**
- Academy: **ES/MES playbook** (Aircraft Carrier)
- Infirmary: **ES/MES + NQ/MNQ**
- Gauntlet: **All playbooks** (ES/NQ/CL/GC)

**Billing Enforcement (Locked):**
- **GRACE ≠ Execution**
  - Execution locks immediately when billing is not ACTIVE.
  - Content (Playbooks + Classifier) may remain accessible until `grace_until`.
  - Training remains accessible always.

**Gauntlet Purchase Control (Locked):**
- Gauntlet checkout requires governance approval (`gauntlet_approved=true` or `gauntlet_status=APPROVED`),
  unless staff-bypass role is present.

---

## Folder Map

- `/telemetry_layer/` — Control Room web app (Vite)
- `/telemetry_layer/functions/` — server functions (Stripe webhook, sessions, sync)
- `/telemetry_layer/supabase/` — Supabase schema + Edge Functions (telemetry ingest + uplink binding)

---

## Canon Entry Points

- Portal Billing Authority: `/portal/billing`
- Weekly surface: `/weekly`
- Nexus surface: `/nexus`
- Library / Playbooks: `/library`

---

## Required Configuration

See:
- `INSTALL_BASE44.md`
- `INSTALL_WEBFLOW.md`

---

## Telemetry Binding (Supabase)

TriArc Nexus emits TradingView alert webhooks containing an `uplink_key`.

**Rule:** TradingView cannot authenticate to Supabase, so `user_id` must be resolved by **pre-binding** the `uplink_key` inside the Control Room.

**Edge Functions (Supabase):**
- `bind_uplink_key` (AUTH REQUIRED): binds an `uplink_key` to the logged-in operator.
- `nexus_ingest` (NO AUTH): receives TradingView envelope, stores raw events, and writes flattened `STATE_SNAPSHOT` rows.

If an `uplink_key` is not yet bound, `nexus_ingest` will still store telemetry (raw + snapshot) but `user_id` will be `NULL` until the key is bound.

