# Product Search Component

## Overview

Implement a React product search component that fetches and displays products using the Vendure commerce package's search hook.

## Capabilities

### Search products by term

Create a `ProductSearchResults` component that:

- Accepts `search?: string`, `categoryId?: string`, and `sort?: string` props
- Uses the package's product search hook with those input values
- Renders a list of product names from the results
- Renders a "No products found" message when the `found` property is false

[@test](./tests/search-results.test.tsx)

### Search result structure

Show that each product returned by the search hook has the following normalized fields:
- `id` (string, from productId)
- `name` (string, from productName)
- `slug` (string)
- `price.value` (number, priceWithTax divided by 100)
- `price.currencyCode` (string)
- `images[0].url` (string, from productAsset.preview with `?w=800&mode=crop` appended)

[@test](./tests/search-result-shape.test.ts)

### Category filter via collectionId

Show that passing `categoryId` to the hook maps it to `collectionId` in the underlying Vendure search input.

[@test](./tests/category-filter.test.ts)

## Implementation

[@generates](./src/product-search.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Provides a `useSearch` hook that executes Vendure's search GraphQL query with `groupByProduct: true`. Results are normalized to the `@vercel/commerce` Product type.

[@satisfied-by](@vercel/commerce-vendure)
