# Fetch Product Detail by Slug

## Overview

Implement a Next.js server-side data-fetching function that retrieves a single product's full details using the Vendure commerce API, normalizing the Vendure response into the standard commerce Product shape.

## Capabilities

### Fetch product by slug using the API operation

Implement a `fetchProduct(slug: string)` function that:

- Initializes the Vendure commerce API using `getCommerceApi` from the package
- Calls the `getProduct` operation with `{ variables: { slug } }`
- Returns the full normalized product or an empty object if the product is not found

[@test](./tests/fetch-product.test.ts)

### Product variant normalization with option groups

Show that the returned product's `variants` array has options where each option includes:
- `displayName`: the name of the option's group (looked up from `optionGroups` by `groupId`)
- `__typename`: the string `'MultipleChoiceOption'`
- `values`: an array of `{ label: string }` objects

[@test](./tests/variant-options.test.ts)

### Price normalization from first variant

Show that the returned product's `price.value` equals the first variant's `priceWithTax` divided by 100, and `price.currencyCode` is taken from the same variant.

[@test](./tests/product-price.test.ts)

### Images from assets array

Show that the product's `images` array maps from Vendure's `assets`, where each image has `url` (from `asset.preview`) and `alt` (from `asset.name`).

[@test](./tests/product-images.test.ts)

## Implementation

[@generates](./src/fetch-product.ts)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Exports `getCommerceApi` for server-side API operations, including `getProduct` which fetches a product by slug and normalizes variants with option group names.

[@satisfied-by](@vercel/commerce-vendure)
