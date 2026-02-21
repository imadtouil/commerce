# Cart Data Normalization

## Overview

Implement a module that converts a Vendure Order object into the standardized `@vercel/commerce` Cart format using the package's normalization utility.

## Capabilities

### Normalize a Vendure Order to a Cart

Implement a `buildCart(order)` function that uses the Vendure commerce package's normalization utility to convert a Vendure Order object into the standard Cart format.

Given a Vendure Order with:
- `id: 42` (number)
- `createdAt: "2024-01-01T00:00:00Z"`
- `subTotalWithTax: 5000` (in cents)
- `totalWithTax: 5500` (in cents)
- `currencyCode: "USD"`
- `customer: { id: "c1" }`
- `lines: []`

The resulting Cart must have:
- `id: "42"` (converted to string)
- `subtotalPrice: 50` (divided by 100)
- `totalPrice: 55` (divided by 100)
- `currency.code: "USD"`
- `taxesIncluded: true`

[@test](./tests/normalize-cart.test.ts)

### Line item normalization

Show that each line item in the normalized Cart has:
- `id`: the line's id
- `quantity`: the line's quantity
- `variant.price`: `discountedUnitPriceWithTax / 100`
- `variant.listPrice`: `unitPriceWithTax / 100`
- `images[0].url`: `featuredAsset.preview + '?preset=thumb'`
- `discounts[0].value`: `discounts[0].amount / 100`
- `path`: `/${productVariant.product.slug}`

[@test](./tests/line-item-normalization.test.ts)

### Product variant ID and product ID

Show that each line item's `variantId` equals `productVariant.id` (string) and `productId` equals `productVariant.productId`.

[@test](./tests/line-item-ids.test.ts)

## Implementation

[@generates](./src/build-cart.ts)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Exports a `normalizeCart` utility that converts a Vendure CartFragment (Order) into the standard `@vercel/commerce` Cart format.

[@satisfied-by](@vercel/commerce-vendure)
