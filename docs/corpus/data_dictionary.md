# Olist Data Dictionary

Reference for all 9 tables in the Olist marketplace dataset (orders placed
2016–2018). Maintained by the analytics team. If a column isn't listed here,
it doesn't exist — do not invent columns.

---

## olist_orders_dataset (~99,441 rows) — one row per order

- **order_id** — unique order identifier. Primary key. Joins to order_items, payments, reviews.
- **customer_id** — foreign key to the customers table. IMPORTANT: this ID is unique *per order*, not per person. To identify a real person across orders, join to customers and use `customer_unique_id`.
- **order_status** — lifecycle status. Values: `created`, `approved`, `invoiced`, `processing`, `shipped`, `delivered`, `canceled`, `unavailable`. Only `delivered` orders count toward revenue metrics (see business_context.md).
- **order_purchase_timestamp** — when the customer placed the order.
- **order_approved_at** — when payment was approved. Can be null for canceled orders.
- **order_delivered_carrier_date** — when the seller handed the package to the logistics carrier.
- **order_delivered_customer_date** — when the customer actually received it. Null unless status is `delivered`.
- **order_estimated_delivery_date** — the delivery date promised to the customer at purchase time. Used for the on-time delivery metric.

## olist_order_items_dataset (~112,650 rows) — one row per item within an order

An order with 3 units of the same product has 3 rows here (`order_item_id` = 1, 2, 3).

- **order_id** — joins to orders.
- **order_item_id** — sequence number of the item within the order (1, 2, 3, ...). NOT a product identifier.
- **product_id** — joins to products.
- **seller_id** — joins to sellers. One order can contain items from multiple sellers.
- **shipping_limit_date** — deadline for the seller to hand the item to the carrier.
- **price** — item price in Brazilian reais (BRL), excluding freight.
- **freight_value** — freight charged for this item row. Order-level freight = sum of freight_value across the order's rows.

## olist_products_dataset (~32,951 rows) — one row per product

- **product_id** — primary key.
- **product_category_name** — category in PORTUGUESE (e.g. `cama_mesa_banho`). Join to the translation table for English names.
- **product_name_lenght** — number of characters in the product title. NOTE: "lenght" is a misspelling that ships in the original data. Queries must use the misspelled column name.
- **product_description_lenght** — characters in the description (same misspelling).
- **product_photos_qty** — number of listing photos.
- **product_weight_g**, **product_length_cm**, **product_height_cm**, **product_width_cm** — physical attributes. A small number of products have nulls here.

## olist_customers_dataset (~99,441 rows) — one row per order-customer link

- **customer_id** — primary key; what orders reference. One per order.
- **customer_unique_id** — the identifier of the actual person. Count DISTINCT customer_unique_id to count real customers or find repeat buyers.
- **customer_zip_code_prefix** — first 5 digits of the customer's zip code.
- **customer_city**, **customer_state** — customer location. State is a 2-letter Brazilian state code (SP, RJ, MG, ...).

## olist_sellers_dataset (~3,095 rows) — one row per seller

- **seller_id** — primary key.
- **seller_zip_code_prefix**, **seller_city**, **seller_state** — seller location.

## olist_order_payments_dataset (~103,886 rows) — one row per payment method used on an order

An order paid partly by card and partly by voucher has multiple rows.

- **order_id** — joins to orders.
- **payment_sequential** — sequence of the payment method within the order.
- **payment_type** — `credit_card`, `boleto` (a Brazilian bank-slip payment method), `voucher`, `debit_card`, `not_defined`.
- **payment_installments** — number of installments chosen (installment purchases are very common in Brazil).
- **payment_value** — amount paid via this method. Sum per order ≈ items price + freight.

## olist_order_reviews_dataset (~99,000 rows) — one row per review

- **review_id** — review identifier.
- **order_id** — joins to orders. A small number of orders have multiple reviews.
- **review_score** — 1 (worst) to 5 (best).
- **review_comment_title**, **review_comment_message** — free text in Portuguese; null for the majority of reviews (most customers leave only a score).
- **review_creation_date** — when the review survey was sent.
- **review_answer_timestamp** — when the customer answered.

## olist_geolocation_dataset (~1,000,163 rows) — raw geocoding points

- **geolocation_zip_code_prefix** — joins (loosely) to customer/seller zip prefixes.
- **geolocation_lat**, **geolocation_lng** — coordinates.
- **geolocation_city**, **geolocation_state** — location names.
- WARNING: this table contains MANY rows per zip prefix (raw geocoded points, with inconsistent city spellings). Deduplicate or average per zip prefix before joining, or you will explode row counts.

## product_category_name_translation (71 rows)

- **product_category_name** — Portuguese category name (join key to products).
- **product_category_name_english** — English translation. A handful of categories in products have no translation row; report them under their Portuguese name.

---

## Canonical join paths

- Revenue by anything: orders → order_items (on order_id) → products / sellers as needed.
- Real customers: orders → customers (on customer_id), then group by customer_unique_id.
- English categories: products → product_category_name_translation (on product_category_name).
- Never join geolocation without deduplicating it first.
