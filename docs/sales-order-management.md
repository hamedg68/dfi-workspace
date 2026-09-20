# Sales Order Management ownership

This guide applies only to the Sales Order Management feature and its related frontend and backend
code.

## Ownership

The page combines orders owned by two different systems:

* Rahkaran-owned rows:
  * sales requests;
  * remaining sales requests (called `تتمه درخواست‌ها` for food representatives and chain customers).
* DFI-owned rows:
  * market orders;
  * Warehouse 3 orders.

DFI owns and manages the full lifecycle of DFI-originated market and Warehouse 3 orders.

Rahkaran remains the source of truth for Rahkaran sales requests and their commercial and logistics
lifecycle. DFI does not create or edit those requests, quotations, orders, issue permits, sales
delivery vouchers, or related Rahkaran documents.

For Rahkaran-owned rows, DFI only manages its own operational fulfillment metadata, including:

* supply allocation from new-price, old-price, and marked-price inventory;
* delivery date;
* delivery priority;
* automation number;
* DFI workflow and comment metadata, where applicable.

This DFI-only fulfillment data is stored in the DFI database and must not be treated as if it were
stored in Rahkaran.

## Remainders and allocations

When a Rahkaran sales request is not delivered completely, Rahkaran exposes the remaining quantity
as a remainder request. A later sales delivery voucher can reduce that remainder again. DFI supply
allocations are persistent snapshots and must be reconciled against the latest Rahkaran remainder
before they are displayed, validated, or reused. Do not assume a previously saved DFI allocation is
still valid merely because the Rahkaran sale-request ID and part code are unchanged.

Always distinguish these concepts:

```text
Rahkaran requested / delivered / remainder quantity
    !=
DFI new-price / old-price / marked-price supply allocation
```

Trace Rahkaran quantities from the Rahkaran database and DFI allocations from the DFI database.
Do not update Rahkaran lifecycle documents as part of DFI fulfillment management.
