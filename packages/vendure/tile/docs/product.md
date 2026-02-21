# Product Search & Pricing

Client-side React hooks for searching products and formatting prices. Requires the app to be wrapped in `CommerceProvider`.

## Import

```typescript
import { useSearch, usePrice } from '@vercel/commerce-vendure/product'
// or individually:
import useSearch from '@vercel/commerce-vendure/product/use-search'
import usePrice from '@vercel/commerce-vendure/product/use-price'
```

## Types

```typescript { .api }
interface Product {
  id: string
  name: string
  description: string
  slug: string
  path: string          // /${slug}
  images: Array<{
    url: string         // productAsset.preview + '?w=800&mode=crop' or ''
  }>
  variants: any[]       // empty array for search results
  price: {
    value: number       // priceWithTax.min / 100
    currencyCode: string
  }
  options: any[]        // empty array for search results
  sku: string
}

interface SearchProductsInput {
  search?: string        // search term
  categoryId?: string    // Vendure collection ID to filter by
  brandId?: string       // not used by Vendure (included for interface compatibility)
  sort?: string          // not yet implemented in Vendure adapter
  swrOptions?: SWROptions
}

interface SearchProductsData {
  products: Product[]
  found: boolean        // true if totalItems > 0
}
```

## Capabilities

### useSearch

Searches and lists products from Vendure using the `search` GraphQL query. Accepts optional filters. Uses SWR for caching; does not revalidate on focus.

```typescript { .api }
/**
 * Searches products from Vendure's search API.
 * @param input.search - Search term/keyword
 * @param input.categoryId - Vendure collection ID to filter by
 * @param input.brandId - Brand filter (not used by Vendure, included for interface compat)
 * @param input.sort - Sort order (not yet implemented in Vendure adapter)
 * @param input.swrOptions - SWR configuration options
 * @returns SWR response with products array and found boolean
 */
function useSearch(input?: SearchProductsInput): SearchProductsData
```

**Usage:**

```typescript
import { useSearch } from '@vercel/commerce-vendure/product'

// List all products
function ProductGrid() {
  const { products, found } = useSearch()

  if (!found) return <p>No products found</p>

  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <img src={product.images[0]?.url} alt={product.name} />
          <h3>{product.name}</h3>
          <p>${product.price.value} {product.price.currencyCode}</p>
        </div>
      ))}
    </div>
  )
}

// Search with keyword
function SearchResults({ query }: { query: string }) {
  const { products, found } = useSearch({ search: query })
  // ...
}

// Filter by category
function CategoryProducts({ categoryId }: { categoryId: string }) {
  const { products } = useSearch({ categoryId })
  // ...
}
```

### usePrice

Price formatting hook (re-exported from `@vercel/commerce/product/use-price`). Formats prices according to locale.

```typescript { .api }
/**
 * Re-exported from @vercel/commerce. Formats a price value according to locale.
 */
export * from '@vercel/commerce/product/use-price'
export { default } from '@vercel/commerce/product/use-price'
```

**Usage:**

```typescript
import { usePrice } from '@vercel/commerce-vendure/product'

function ProductPrice({ value, currencyCode }: { value: number; currencyCode: string }) {
  const { price } = usePrice({
    amount: value,
    currencyCode,
  })

  return <span>{price}</span>
}
```

## Vendure GraphQL Query

The `useSearch` hook uses the `search` query:

```graphql
query search($input: SearchInput!) {
  search(input: $input) {
    items {
      productId
      productName
      description
      slug
      sku
      currencyCode
      productAsset { id preview }
      priceWithTax {
        ... on SinglePrice { value }
        ... on PriceRange { min max }
      }
    }
    totalItems
  }
}
```

The `SearchInput` sent to Vendure includes:
- `term` — from `input.search`
- `collectionId` — from `input.categoryId`
- `groupByProduct: true` — always set to group results by product

## Notes

- `useSearch` groups results by product (`groupByProduct: true`), so each result represents a unique product.
- The `brandId` input parameter is accepted but not used internally (Vendure does not have a native brand concept in the search API).
- The `sort` parameter is accepted but not yet implemented in the adapter.
- Product images in search results use the `?w=800&mode=crop` query parameter for optimization.
- Prices are stored in Vendure as integers (smallest currency unit) and divided by 100 in the normalized result.
