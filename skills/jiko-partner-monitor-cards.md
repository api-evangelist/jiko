---
name: Authenticate to the Jiko Partner API and monitor customer cards
description: Log in to the Jiko Partner API, discover webhook event types, and list a customer's cards — signing every request per Jiko's HMAC + idempotency requirements.
api: jiko:jiko-partner-api
operations:
  - login_api_v1_login__post
  - list_event_types_api_v1_events_types__get
  - list_customer_cards_api_v1_customers__customer_id__cards__get
---

# Authenticate to the Jiko Partner API and monitor customer cards

Grounded in the Jiko Partner API. Base path `/api/v1`. See
`authentication/jiko-authentication.yml`, `conventions/jiko-conventions.yml`, and
`data-model/jiko-data-model.yml`.

## Auth (every request)
Jiko provisions a `username`, `password`, and `shared_secret`, and allowlists your IP.
1. **Login** — call `login_api_v1_login__post` (POST `/api/v1/login/`) to get a bearer
   token (60-minute lifetime; re-login when it expires).
2. On **every** request, send three headers:
   - `Authorization: Bearer <access_token>`
   - `x-jiko-idempotency: <random UUID>` (unique per action; replay window is 1 hour)
   - `x-jiko-signature:` base64 `HMAC-SHA256(x-jiko-idempotency + pathname + body, shared_secret)`

## Steps
1. **Discover events** — call `list_event_types_api_v1_events_types__get`
   (GET `/api/v1/events/types/`) to enumerate subscribable event types, including
   `card.status.*` and `card.transaction.approved` / `card.transaction.rejected`.
2. **List cards** — call `list_customer_cards_api_v1_customers__customer_id__cards__get`
   (GET `/api/v1/customers/{customer_id}/cards/`); each card includes its `pocket_id`.
3. Optionally subscribe to the card events from step 1 so status/transaction changes
   are pushed rather than polled.

## Rules
- The idempotency key is part of the signature input — generate a fresh UUID per
  request and sign with it.
- Requests must come from an allowlisted IP or they are rejected.
- Pagination uses `offset`/`count`; responses wrap results in a `List` envelope with
  `offset`, `count`, and `items`.
