# Nest Playground 3 – GraphQL

## Code-first vs Schema-first

I chose the **schema-first** approach. For me, it is important to define the schema first because the API design is cleaner and more structured this way.

However, I noticed one disadvantage — duplicated types (TypeScript types + GraphQL types).

In the long term, I believe schema-first is better for building a more understandable and well-designed API system.

---

## Orders Query

The main business logic is located in the `OrdersService`.

The `@Resolver` plays a role similar to `@Controller`, but most of the important logic is placed inside the `OrdersService`.

---

## N+1 Problem (`OrdersNaiveN1` query)

### Example (Naive Implementation)

```bash
[Nest] 56375  - 12.02.2026, 21:49:04     LOG [OrdersService] Found 3 orders (total: 3)
[Nest] 56375  - 12.02.2026, 21:49:04   DEBUG [OrderItemNaiveResolver] N+1 hit → SELECT product WHERE id = 6ea6417c-748b-41ad-8474-6b9eb2f78113
[Nest] 56375  - 12.02.2026, 21:49:04   DEBUG [OrderItemNaiveResolver] N+1 hit → SELECT product WHERE id = 6ea6417c-748b-41ad-8474-6b9eb2f78113
[Nest] 56375  - 12.02.2026, 21:49:04   DEBUG [OrderItemNaiveResolver] N+1 hit → SELECT product WHERE id = 6ea6417c-748b-41ad-8474-6b9eb2f78113

query: SELECT "Product"."id" AS "Product_id",
              "Product"."title" AS "Product_title",
              "Product"."price" AS "Product_price",
              "Product"."is_active" AS "Product_is_active",
              "Product"."stock" AS "Product_stock",
              "Product"."created_at" AS "Product_created_at",
              "Product"."updated_at" AS "Product_updated_at"
       FROM "products" "Product"
       WHERE (("Product"."id" = $1))
       LIMIT 1
-- PARAMETERS: ["6ea6417c-748b-41ad-8474-6b9eb2f78113"]
```

This query is executed **once per each order item**, causing N+1 problem.

---

### After DataLoader (`orders` query)

```bash
[Nest] 56375  - 12.02.2026, 21:50:12     LOG [OrdersService] Found 3 orders (total: 3)

query: SELECT "Product"."id" AS "Product_id",
              "Product"."title" AS "Product_title",
              "Product"."price" AS "Product_price",
              "Product"."is_active" AS "Product_is_active",
              "Product"."stock" AS "Product_stock",
              "Product"."created_at" AS "Product_created_at",
              "Product"."updated_at" AS "Product_updated_at"
       FROM "products" "Product"
       WHERE (("Product"."id" IN ($1, $2, $3, $4)))
-- PARAMETERS: ["6ea6417c-...", "a1b2c3d4-...", "f5e6d7c8-...", "12345678-..."]
```

With DataLoader, all product IDs are **batched into a single `IN(...)` query** instead of N separate queries.
