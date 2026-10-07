# Analyst Notes — Known Quirks & Gotchas

Working notes from the analytics team. Read before writing any query.

## The customer_id trap (most common mistake)

`orders.customer_id` is a fresh ID for every order. Counting distinct
`customer_id` counts ORDERS, not people, and makes every customer look like a
one-time buyer. For anything about people (repeat rate, customer lifetime
orders), join orders → customers and use `customer_unique_id`. About 3% of
unique customers have more than one order.

## Order status pitfalls

- `order_delivered_customer_date` is null for anything not delivered — never compute delivery time without filtering `order_status = 'delivered'`.
- `unavailable` means the seller could not fulfill after payment (usually out of stock). It is operationally distinct from `canceled`; report them separately unless asked for a combined "failed orders" number.
- A tiny number of `delivered` orders have missing timestamps; exclude nulls rather than imputing.

## Money math

- Order revenue = SUM of `order_items.price` per order (multiple rows per order!). Joining orders to order_items and summing without GROUP BY order_id will double-count nothing, but averaging "price" at the order level without aggregating first understates order values.
- `payments.payment_value` per order ≈ items + freight, but small rounding gaps and voucher edge cases exist. For revenue use order_items.price (per the GMV definition), not payments.
- Vouchers can produce many small payment rows on one order.

## Text and naming quirks

- `product_name_lenght` and `product_description_lenght` are misspelled in the source data ("lenght"). Queries referencing "length" (spelled correctly) will fail with "no such column".
- Product categories are Portuguese snake_case; always join the translation table when presenting to stakeholders. A few categories (e.g. some niche ones) have no English translation row — a LEFT JOIN keeps them.
- Review comments are Portuguese free text and mostly null; only ~40% of reviews include a written message.

## Geolocation warnings

The geolocation table is raw geocoding output: ~1M rows, many per zip prefix,
with inconsistent accents/spellings in city names. Standard practice here:
build a deduplicated lookup (one averaged lat/lng per zip prefix) before any
join. Joining it raw multiplies your row counts by hundreds.

## Timeline notes

- Coverage: Sep 2016 – Oct 2018 (2016 is a tiny pilot; start trends at 2017-01).
- There is a known one-off spike around Black Friday (late Nov 2017) — a real event, not a data error.
- The final weeks of the dataset under-represent `delivered` orders (they hadn't arrived yet when the snapshot was taken); be careful with delivery metrics in the last month.
