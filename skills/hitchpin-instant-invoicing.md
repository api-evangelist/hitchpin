---
name: Create, share, and collect on a HitchPin instant invoice
description: >-
  Operating instructions for using the HitchPin Django API to create an instant
  invoice, share it with a buyer, let them pay it, and retrieve the receipt.
api: openapi/hitchpin-django-api-openapi.json
operations:
- create_instant_invoices
- share_instant_invoice
- retrieve_shared_instant_invoice
- pay_instant_invoice
- get_instant_invoice_receipt
- list_instant_invoices
- void_instant_invoice
---

# HitchPin instant invoicing

Use the HitchPin Django API (`https://apiv2.hitchpin.com`, sandbox
`https://apiv2.sandbox.hitchpin.com`) to bill a buyer on the agricultural
marketplace and collect payment.

## Authentication

Send an API token in the `Authorization` header using the `Token <token>` format
(scheme `tokenAuth`), or rely on the `sessionid` cookie for browser sessions
(scheme `cookieAuth`). See `authentication/hitchpin-authentication.yml`.

## Steps

1. **Create the invoice.** `POST /instant-invoices/` (`create_instant_invoices`)
   with the line items and buyer details. The response returns the
   `invoice_number` you will use for every later step.
2. **Share it.** `POST /instant-invoices/{invoice_number}/share/`
   (`share_instant_invoice`) to mint a public `share_hash` the buyer can open
   without an account.
3. **Buyer views it.** `GET /instant-invoice-share/{share_hash}/`
   (`retrieve_shared_instant_invoice`) returns the share-safe projection of the
   invoice.
4. **Buyer pays it.** `POST /instant-invoice-share/{share_hash}/pay/`
   (`pay_instant_invoice`). This is a money-movement operation — treat it as
   physical-consequence per `agentic-access/hitchpin-agentic-access.yml`.
5. **Retrieve the receipt.** `GET /instant-invoice-share/{share_hash}/receipt/`
   (`get_instant_invoice_receipt`) once payment succeeds.

## Housekeeping

- **List / reconcile:** `GET /instant-invoices/` (`list_instant_invoices`) is
  paginated with `limit`/`offset`; page using the `next`/`previous` URLs in the
  response envelope (`count`, `next`, `previous`, `results`).
- **Void an invoice:** `DELETE /instant-invoices/{invoice_number}/revoke/`
  (`void_instant_invoice`) revokes an invoice that should no longer be payable.

## Errors

Error responses return a `{type, message}` envelope. Expect `400` for validation
failures and `404` when an invoice number or share hash does not exist. See
`errors/hitchpin-problem-types.yml`. No idempotency-key contract is published, so
do not assume retries are automatically deduplicated.
