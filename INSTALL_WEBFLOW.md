# INSTALLATION SOP — WEBFLOW + MEMBERSTACK FRONT DOOR

Webflow is the distribution surface. Control Room is the governed portal.

## 1) Webflow Role Routing (Recommended)

Create minimal navigation:
- Portal
- Weekly
- Nexus
- Library
- Governance

All governed pages should route into the Control Room domain and require auth.

## 2) Memberstack (Auth + RBAC)

Memberstack should:
- Authenticate identity
- Provide role/tier claims (if used)

Stripe authority remains canonical in the Control Room:
- Tier is enforced from Stripe state (webhook + sync), not from client claims.

## 3) Checkout Links

- Academy/Infirmary can link to `/portal/billing` (preferred)
- Gauntlet must route to approval workflow first

## 4) Canon Rule

Webflow may display doctrine and summaries.
All execution-adjacent surfaces remain inside the Control Room portal.

## 5) Uplink Key Binding (Required for Telemetry Ownership)

When an operator installs TriArc Nexus in TradingView, the script emits webhooks containing an `uplink_key`.

**Rule:** TradingView cannot authenticate into Supabase. The Control Room must bind the `uplink_key` to the operator identity first.

Implementation:
- Provide a portal page (inside Control Room) where the operator pastes their `uplink_key`.
- Call Supabase Edge Function: `bind_uplink_key`.
- After binding, incoming `STATE_SNAPSHOT` events will resolve to the correct `user_id`.

