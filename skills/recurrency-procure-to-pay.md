---
name: Recurrency procure-to-pay
description: Discover a supplier's catalog, check price and availability, then create a sales order via the Recurrency e-procurement API.
api: openapi/recurrency-openapi.yml
operations: [listSuppliers, listItems, getItemPrice, createOrder, getOrderDetails]
---

# Recurrency procure-to-pay

Use the Recurrency e-procurement API to go from supplier discovery to a placed order.

## Authenticate
1. POST `https://api.recurrency.com/oauth/token` (use `api-staging.recurrency.com` for testing) with `Content-Type: application/json` and body `{"grant_type":"client_credentials","client_id":"...","client_secret":"..."}`.
2. Read `access_token` from the response (scope `recurrency:api:tenant`, valid 7 days) and send it on every call as `authorization: Bearer {access_token}`.

## Steps
1. **listSuppliers** — `GET /marketplace/suppliers`. Pick the target supplier and keep its `slug`.
2. **listItems** — `GET /marketplace/suppliers/{slug}/items` with `searchQuery`/filters and `limit`/`offset` to find the item; keep its `itemId`. Paginate with `offset` until `totalCount` is covered.
3. **getItemPrice** — `GET /marketplace/suppliers/{slug}/item-price?itemId=&salesLocId=&sourceLocId=` (all three required) to confirm `unitPrice` and `quantityAvailable` before ordering.
4. **createOrder** — `POST /orders` with at least `customerId` and a `lines[]` array of `{itemId, qtyOrdered, unitPrice, unitOfMeasure}`. Read `orderId` from the response.
5. **getOrderDetails** — `GET /order-details/{salesOrderId}` to confirm status, totals and line items.

## Rules
- **No idempotency key** is supported; do not blindly retry `createOrder` — on a network failure, reconcile with `listOrders`/`getOrderDetails` before re-posting.
- Errors come back as `{ "message": ..., "details": {...} }` (custom envelope, not RFC 9457); read `details` for field-level causes. See `errors/recurrency-problem-types.yml`.
- `uploadItems` and any item batch is capped at **100 items per request**.
- List endpoints page with `limit` (default 20) / `offset` (default 0) and sort with `sortBy`/`sortDir`.
