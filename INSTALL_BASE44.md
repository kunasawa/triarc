# INSTALLATION SOP — BASE44 DEPLOYMENT

This procedure installs the TriArc Control Room stack with Stripe Billing Authority.

## 1) Environment Variables (Required)

Set the following in your Base44 environment:

- `STRIPE_SECRET_KEY` — Stripe secret key
- `STRIPE_WEBHOOK_SECRET` — webhook signing secret (`whsec_...`)
- `APP_URL` (or `APP_BASE_URL`) — your deployed origin (e.g., `https://control.triarc.eco`)

Optional:
- `TRIARC_BILLING_GRACE_DAYS` — default `3`

## 2) Stripe Webhook Registration

In Stripe Dashboard:

**Developers → Webhooks → Add endpoint**

Endpoint URL:
- Use the deployed URL for the function `stripeWebhook` (Base44 function route)

Subscribe to events:
- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_failed`

Copy `whsec_...` into `STRIPE_WEBHOOK_SECRET`.

## 3) Gauntlet Approval Field

Before allowing Gauntlet checkout, ensure your identity record supports one of:
- `gauntlet_approved` (boolean)
- `gauntlet_status` (string) with value `APPROVED`

If your schema uses a different field name, map it inside:
- `telemetry_layer/functions/createCheckoutSession.ts`

## 4) Billing Authority Verification

Open:
- `/portal/billing`

Confirm:
- Academy/Infirmary Checkout creates Stripe subscription
- Webhook updates the identity record:
  - `tier`, `subscription_status`, `grace_until`, `entitlement_playbooks`, `entitlement_suite`

## 5) Enforcement Verification

Test states:

A) ACTIVE
- Execution: permitted (subject to governance)
- Classifier: accessible
- Playbooks: accessible per tier

B) PAST_DUE
- Execution: **locked immediately**
- Classifier/Playbooks: accessible **until** `grace_until`

C) CANCELED / UNPAID
- Execution: locked
- Classifier/Playbooks: locked
- Training: accessible

