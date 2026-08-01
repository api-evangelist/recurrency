---
name: Recurrency upload supplier catalog
description: Upload and verify a supplier's product catalog into Recurrency in batches of up to 100 items.
api: openapi/recurrency-openapi.yml
operations: [listSuppliers, uploadItems, listItems]
---

# Recurrency upload supplier catalog

Publish supplier product data into Recurrency, then verify it landed.

## Authenticate
POST `https://api.recurrency.com/oauth/token` with `grant_type=client_credentials` + `client_id`/`client_secret`; send the returned token as `authorization: Bearer {access_token}`.

## Steps
1. **listSuppliers** — `GET /marketplace/suppliers` to confirm the target supplier `slug`.
2. **uploadItems** — `POST /marketplace/suppliers/{slug}/items` with `{ "items": [ ... ] }`. Each item requires `product_name`; optional fields include `part_number`, `list_price`, `on_hand`, `images[]`, `attributes[]`, `documents[]`, `categories[]`. **Send at most 100 items per request** — chunk larger catalogs and repeat.
3. Read `totalCreated` and `createdItems[]` from the `201` response.
4. **listItems** — `GET /marketplace/suppliers/{slug}/items?searchQuery=...` to verify uploaded items are queryable.

## Rules
- Batch cap is **100 items/request**; a `400` with `details` (e.g. `product_name` missing) means fix and resend that batch only.
- No idempotency key: on an ambiguous failure, verify with `listItems` before re-uploading to avoid duplicates.
- Errors use the `{ message, details }` envelope — see `conventions/recurrency-conventions.yml` and `errors/recurrency-problem-types.yml`.
