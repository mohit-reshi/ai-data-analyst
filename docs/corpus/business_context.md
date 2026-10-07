# Olist — Business Context & Official Metric Definitions

Maintained by the analytics team. These are the AGREED definitions; any number
reported to leadership must follow them.

## Who we are

Olist is a Brazilian e-commerce enablement platform: small and medium sellers
list their products through Olist, and Olist places them on major online
marketplaces and handles the storefront experience. When a shopper buys, the
seller ships the item; logistics partners deliver it; the customer is invited
to leave a review after the estimated delivery date. This dataset covers real,
anonymized orders from late 2016 through 2018. All monetary values are in
Brazilian reais (BRL).

## Who uses this assistant

- **Operations leads** — delivery performance, carrier hand-off delays, regional bottlenecks.
- **Category managers** — sales, pricing, and review performance by product category.
- **Seller success team** — seller-level GMV, shipping-deadline compliance, review scores.
- **Finance** — GMV, payment-mix, and installment trends.

## Official metric definitions

- **GMV (Gross Merchandise Value)** — sum of `price` across order items for orders with status `delivered`. Freight is EXCLUDED from GMV. Canceled and undelivered orders are excluded.
- **AOV (Average Order Value)** — GMV divided by the number of distinct delivered orders.
- **On-time delivery rate** — among delivered orders, the share where `order_delivered_customer_date` ≤ `order_estimated_delivery_date` (comparing dates). This is our headline logistics KPI; target is 90%+.
- **Average delivery time** — days between `order_purchase_timestamp` and `order_delivered_customer_date`, for delivered orders.
- **Cancellation rate** — orders with status `canceled` divided by all orders in the period.
- **Repeat customer rate** — share of `customer_unique_id`s with 2+ delivered orders. Reminder: NEVER compute this with `customer_id`, which changes every order and will always show ~0% repeat customers.
- **Average review score** — mean of `review_score` over reviews linked to delivered orders in the period.
- **Seller shipping compliance** — share of order items handed to the carrier (`order_delivered_carrier_date`) on or before `shipping_limit_date`.

## Reporting conventions

- Periods are based on `order_purchase_timestamp` unless a question is explicitly about delivery dates.
- 2016 data is a small pilot (a few hundred orders); trend analyses normally start from January 2017. Data ends in 2018, so "last quarter"-style questions must be anchored to the dataset's timeline, not today's date.
- The `boleto` payment type is a Brazilian cash-based bank slip: the customer receives a voucher and pays at a bank/ATM, so payment approval (`order_approved_at`) can lag purchase by 1–2 business days. This is expected, not a data error.
- Installment purchases (`payment_installments` > 1) are standard consumer behavior in Brazil, especially for higher-ticket categories; high installment counts are not fraud signals.

## Out of scope for this assistant

Anything not derivable from these tables: profit margins and costs (we have no
cost data), inventory levels, web traffic/conversion, marketing spend, and
individual customer PII. The assistant should say it cannot answer these
rather than guessing.
