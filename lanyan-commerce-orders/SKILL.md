---
name: lanyan-commerce-orders
description: Use for LanYan product types, SKU, vouchers, charging coupons, product categories, points redemption, zero-price orders, payment success, inventory, order details, or points ledger changes. Model type-specific fields and complete every order path consistently across admin, backend, and mini program.
---

# LanYan Commerce And Orders

## Model By Product Type

- Require SKU only for product types that actually select a specification.
- Keep product main image, carousel images, and SKU images distinct.
- Let vouchers/charging coupons use the shortest valid exchange path; do not force confirmation data that has no business meaning.
- Define a single source of truth for type, price/points, stock, delivery/benefit, and order status.

## Category Rules

- Store hierarchy with a real parent field and deploy it before querying it.
- First-level categories own entry images; second-level categories normally own name, order, status, and parent only.
- Use deterministic parent/order/id ordering and pass the correct level's category ID to product queries.

## Complete The Transaction

1. Preview eligibility and balance.
2. Create the order safely with stock/balance validation.
3. For direct exchange or zero-price success, call the same service-level success handler as the corresponding paid flow.
4. Update status, deduct/write points ledger, grant the coupon/benefit, and record audit information atomically where possible.
5. Return an actionable message and route to the appropriate success state.

## Verify

Test normal SKU goods and voucher goods separately, including insufficient points, repeated submission, stock boundary, success page, order record, and ledger/benefit record.
