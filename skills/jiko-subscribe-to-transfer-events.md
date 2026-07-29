---
name: Subscribe to Jiko transfer events and reconcile
description: Use the Jiko Customer API to subscribe to real-time transfer/webhook events and fetch the corresponding transfer details for reconciliation.
api: jiko:jiko-customer-api
operations:
  - create_subscription_api_v1_subscriptions__post
  - get_transfer_request_api_v1_transfer_requests__transfer_id___get
  - list-transactions-v2
---

# Subscribe to Jiko transfer events and reconcile

Grounded in the Jiko Customer API. Base path `/api/v1`. See
`authentication/jiko-authentication.yml`, `conventions/jiko-conventions.yml`,
`asyncapi/jiko-webhooks.yml`, and `rate-limits/jiko-rate-limits.yml`.

## Auth
1. Authenticate with OAuth 2.0. Use the authorization code flow (user-facing) or
   client credentials (backend). All clients use **Private Key JWT** — sign a JWT with
   your private key and send it as `client_assertion`; PKCE is required, DPoP optional.
2. Request the scopes you need: `transfers.read`, `subscriptions.write`,
   `subscriptions.read` (see `scopes/jiko-scopes.yml`). Access tokens live 15 minutes;
   refresh with the refresh token (auth code flow only).

## Steps
1. **Create the subscription** — call `create_subscription_api_v1_subscriptions__post`
   (POST `/api/v1/subscriptions/`) with the event types to receive (e.g.
   `transfers.on-us.received`, `transfers.wire.in.success`), your callback URL, and a
   shared secret (≥16 chars) used to verify delivery signatures.
2. **Receive events** — each webhook body carries `event_id`, `event_type`,
   `timestamp`, a `payload` (with a `wire_id` or `on_us_id`), and `customer_id`.
3. **Fetch transfer detail** — pass the `wire_id`/`on_us_id` as `{transfer_id}` to
   `get_transfer_request_api_v1_transfer_requests__transfer_id___get`
   (GET `/api/v1/transfer-requests/{transfer_id}/`).
4. **Reconcile** — use `list-transactions-v2` (filterable by card and portal ids) to
   match the transfer against ledger transactions.

## Rules
- Respect rate limits: Transactions and Transfer Requests (read) are 120/min; watch
  `X-RateLimit-*` headers and back off on `429` using `Retry-After`.
- Never poll when a webhook covers the event — subscribe instead.
