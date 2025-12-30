# Ecommerce DB schema (PostgreSQL)

This project uses PostgreSQL running on port `5000` with DB `myapp`.

Connection command (from `db_connection.txt`):
- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

## Tables created

### products
- `id` (BIGSERIAL, PK)
- `sku` (TEXT, UNIQUE, NOT NULL)
- `name` (TEXT, NOT NULL)
- `description` (TEXT, NOT NULL, default '')
- `image_url` (TEXT, nullable)
- `price_cents` (INTEGER, NOT NULL, >= 0)
- `currency_code` (CHAR(3), NOT NULL, default 'USD')
- `active` (BOOLEAN, default TRUE)
- `created_at`, `updated_at` (TIMESTAMPTZ)

### inventory
- `product_id` (BIGINT, PK, FK -> products.id, ON DELETE CASCADE)
- `quantity` (INTEGER, NOT NULL, >= 0)
- `reserved` (INTEGER, NOT NULL, >= 0)
- `updated_at` (TIMESTAMPTZ)

### carts
- `id` (BIGSERIAL, PK)
- `status` (TEXT, default 'open', one of: open/checked_out/abandoned)
- `created_at`, `updated_at` (TIMESTAMPTZ)

### cart_items
- `id` (BIGSERIAL, PK)
- `cart_id` (BIGINT, FK -> carts.id, ON DELETE CASCADE)
- `product_id` (BIGINT, FK -> products.id, ON DELETE RESTRICT)
- `quantity` (INTEGER, NOT NULL, > 0)
- `unit_price_cents` (INTEGER, NOT NULL, >= 0)
- `currency_code` (CHAR(3), default 'USD')
- `created_at`, `updated_at` (TIMESTAMPTZ)
- Unique constraint: `(cart_id, product_id)`

### orders
- `id` (BIGSERIAL, PK)
- `cart_id` (BIGINT, UNIQUE, FK -> carts.id, ON DELETE SET NULL)
- `status` (TEXT, default 'placed', one of: placed/paid/shipped/cancelled/refunded)
- `customer_email` (TEXT, nullable)
- `subtotal_cents`, `tax_cents`, `shipping_cents`, `total_cents` (INTEGER, >= 0)
- `currency_code` (CHAR(3), default 'USD')
- `created_at`, `updated_at` (TIMESTAMPTZ)

### order_items
- `id` (BIGSERIAL, PK)
- `order_id` (BIGINT, FK -> orders.id, ON DELETE CASCADE)
- `product_id` (BIGINT, FK -> products.id, ON DELETE SET NULL)
- `product_name` (TEXT, NOT NULL)  # snapshot for historical accuracy
- `quantity` (INTEGER, NOT NULL, > 0)
- `unit_price_cents` (INTEGER, NOT NULL, >= 0)
- `line_total_cents` (INTEGER, NOT NULL, >= 0)
- `currency_code` (CHAR(3), default 'USD')
- `created_at` (TIMESTAMPTZ)

## Seed data
Seeded 5 products (SKU-TSHIRT-001, SKU-MUG-001, SKU-HOODIE-001, SKU-CAP-001, SKU-BAG-001) and corresponding inventory quantities.

Note: Seed inserts were done with `ON CONFLICT (sku) DO UPDATE` for products and `ON CONFLICT (product_id) DO UPDATE` for inventory, so re-running seeding is safe (idempotent).
