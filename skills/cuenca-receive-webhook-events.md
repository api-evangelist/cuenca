---
name: Register and consume webhook events
description: Subscribe an HTTPS endpoint to Cuenca events and process transaction, card, deposit and user notifications.
api: https://github.com/cuenca-mx/cuenca-python
operations: [Endpoint.create, Endpoint.all, Endpoint.deactivate, Webhook.retrieve]
---

# Register and consume webhook events

Cuenca pushes events (deposits received, transfers/card transactions updated, users created, etc.) to HTTPS endpoints you register.

## Auth
- API key + secret (HTTP Basic), same as the transfer skill. Use `sandbox=True` while testing.

## Steps
1. **Expose an HTTPS receiver** that accepts POST requests and responds 2xx quickly.
2. **Register the endpoint** — `Endpoint.create(...)` with your receiver URL and the events to subscribe to.
3. **List / verify** registered endpoints with `Endpoint.all()`; `Endpoint.deactivate(id)` to remove one.
4. **Handle deliveries** — each delivered `Webhook` has an `event` (e.g. `deposit.create`, `transaction.update`, `card_transaction.update`, `user.create`) and a `payload` object. Route on `event`.
5. **Fetch/replay** a specific event with `Webhook.retrieve(id)` or query the webhook log.

## Event catalog (from asyncapi/cuenca-webhooks.yml)
`card_transaction.create/update`, `user.create/update/delete`, `transaction.create/update`, `deposit.create/update`, `withdrawal.create/update`, `cash_deposit.create/update`, `bank_account.create/update`.

## Rules
- Treat deliveries as at-least-once: dedupe on the event/resource id.
- Respond 2xx fast; do slow work asynchronously.
