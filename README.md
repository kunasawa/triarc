[README_TELEMETRY_EDGE_FUNCTIONS.md](https://github.com/user-attachments/files/25508696/README_TELEMETRY_EDGE_FUNCTIONS.md)
# Supabase Telemetry Edge Functions — TriArc Control Room

This folder contains the **schema** (migrations) and **Edge Functions** required to bind TriArc Nexus telemetry to operator identities.

## Why binding is required

TradingView alerts cannot send a Supabase auth token.

Therefore, the webhook ingest function can only identify an operator by **uplink_key**.
To attach telemetry to a user, the Control Room must **bind** that uplink_key to the operator account ahead of time.

## Functions

### 1) `bind_uplink_key` (AUTH REQUIRED)

Purpose: binds an `uplink_key` to the currently authenticated operator.

Input:
```json
{ "uplink_key": "TA-PRM-XXXX-YYYY" }
```

Behavior:
- If the key is unused → creates `public.uplink_keys` row.
- If key already belongs to the same user → safe upsert.
- If key belongs to a different user → **409** (refuse takeover).

### 2) `nexus_ingest` (NO AUTH)

Purpose: receives TradingView webhook envelopes from TriArc Nexus.

Expected envelope:
```json
{ "event_type": "STATE_SNAPSHOT", "uplink_key": "TA-PRM-...", "payload": { ... } }
```

Writes:
- `public.nexus_events` (raw payload)
- `public.state_snapshots` (flattened) for `STATE_SNAPSHOT`

User resolution:
- Looks up `public.uplink_keys` by `uplink_key` (active only).
- If found → `state_snapshots.user_id` is populated.
- If missing → snapshot is stored, but `user_id` remains `NULL` until the key is bound.

## Deployment notes

- Set Edge Function secrets:
  - `SUPABASE_URL`
  - `SUPABASE_ANON_KEY`
  - `SUPABASE_SERVICE_ROLE_KEY`

- Configure your TradingView webhook URL to point at `nexus_ingest`.

TriArc doctrine: **Permission > Opportunity**.
