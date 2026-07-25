---
name: Send a SPEI transfer idempotently
description: Move money to any Mexican bank account (CLABE) via Cuenca's SPEI rail without creating duplicates.
api: https://github.com/cuenca-mx/cuenca-python
operations: [Transfer.create, Transfer.retrieve, Transfer.all, Balance.retrieve]
---

# Send a SPEI transfer idempotently

Use Cuenca to send an interbank (SPEI) or internal transfer in Mexico. Amounts are in **centavos** (not pesos). Every transfer is protected by an **idempotency key** so retries never double-send.

## Auth
- Configure the client with an API key + secret (HTTP Basic): set `CUENCA_API_KEY` / `CUENCA_API_SECRET`, or call `cuenca.configure(api_key='PK...', api_secret='...')`.
- Use `cuenca.configure(sandbox=True)` to route to `sandbox.cuenca.com` while testing.

## Steps
1. **(Optional) Check funds** — `Balance.retrieve()` to confirm there is enough balance for `amount`.
2. **Choose an idempotency key** — use your own internal database id for the payout row. This is the key correctness rule: the same key must map to exactly one transfer.
3. **Create the transfer** — `Transfer.create(account_number=<CLABE>, amount=<centavos>, descriptor=<text shown to recipient>, recipient_name=<name>, idempotency_key=<your id>)`. Network is inferred (`spei` for external CLABEs, `internal` for Cuenca accounts).
4. **Persist the returned transfer id and status** (`created` / `submitted` / `succeeded` / `failed`).
5. **Reconcile** — poll `Transfer.retrieve(id)` or query `Transfer.all(idempotency_key=<your id>)` / `Transfer.all(status='succeeded')`. On SPEI success a `tracking_key` (clave de rastreo) is populated for the CEP receipt.

## Rules
- Amounts are integers in centavos; never send pesos.
- Reusing an idempotency key returns the existing transfer instead of creating a new one — safe to retry on timeouts/5xx.
- Errors raise `CuencaResponseException(json, status_code)` — inspect `status_code` and `json`. See errors/cuenca-problem-types.yml.
