# Product Search

Client-side React hooks for product catalog browsing, search, and price formatting. All hooks must be used within a `CommerceProvider`.

## Imports

```typescript
import useSearch from '@vercel/commerce-kibocommerce/product/use-search'
import usePrice from '@vercel/commerce-kibocommerce/product/use-price'
```

## Capabilities

### useSearch

SWR-based hook for querying the product catalog. Supports full-text search, category filtering, brand filtering, and sorting.

```typescript { .api }
import useSearch from '@vercel/commerce-kibocommerce/product/use-search'

function useSearch(input?: {
  search?: string
  categoryId?: string | number
  brandId?: number
  sort?: 'latest-desc' | 'price-asc' | 'price-desc' | 'trending-desc' | ''
  swrOptions?: SWROptions
}): {
  data: SearchProductsData | null
  isLoading: boolean
  error?: Error
}

interface SearchProductsData {
  products: Product[]
  found: boolean
}
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | `string` | Full-text search query |
| `categoryId` | `string \| number` | Filter by category ID (must be integer-parseable) |
| `brandId` | `number` | Filter by brand ID (must be integer) |
| `sort` | `string` | Sort order (see sort values below) |
| `swrOptions` | `SWROptions` | SWR configuration overrides (default: `revalidateOnFocus: false`) |

**Sort values**:

| Value | Description |
|-------|-------------|
| `'latest-desc'` | Newest first (`createDate desc`) |
| `'price-asc'` | Lowest price first |
| `'price-desc'` | Highest price first |
| `'trending-desc'` | Trending (default, empty sort) |
| `''` | Default sort |

**Returns**:

| Field | Type | Description |
|-------|------|-------------|
| `data.products` | `Product[]` | Array of normalized products |
| `data.found` | `boolean` | Whether products were found |
| `isLoading` | `boolean` | Whether the request is in-flight |
| `error` | `Error \| undefined` | Error from the fetch, if any |

**API endpoint**: `GET /api/commerce/catalog/products` with query params:
- `search` — search text
- `categoryId` — category filter
- `brandId` — brand filter
- `sort` — sort order

**Example**:

```typescript
import useSearch from '@vercel/commerce-kibocommerce/product/use-search'

function ProductListing({ categoryId }: { categoryId: string }) {
  const { data, isLoading } = useSearch({
    categoryId,
    sort: 'price-asc',
  })

  if (isLoading) return <div>Loading products...</div>

  return (
    <ul>
      {data?.products.map((product) => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  )
}
```

```typescript
// Full-text search example
function SearchResults({ query }: { query: string }) {
  const { data, isLoading } = useSearch({ search: query })

  return (
    <div>
      {isLoading ? (
        <p>Searching...</p>
      ) : (
        <p>Found {data?.products.length ?? 0} results for "{query}"</p>
      )}
    </div>
  )
}
```

### usePrice

Re-exported from `@vercel/commerce/product/use-price`. Formats a numeric price into a localized currency string.

```typescript { .api }
import usePrice from '@vercel/commerce-kibocommerce/product/use-price'

function usePrice(input: {
  amount: number
  baseAmount?: number
  currencyCode: string
}): {
  price: string
  basePrice?: string
  discount?: string
}
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `amount` | `number` | The sale/current price amount |
| `baseAmount` | `number?` | The original/base price (for showing discounts) |
| `currencyCode` | `string` | ISO 4217 currency code (e.g., `'USD'`) |

**Returns**:

| Field | Type | Description |
|-------|------|-------------|
| `price` | `string` | Formatted price string (e.g., `'$29.99'`); empty string if `currencyCode` is falsy |
| `basePrice` | `string \| undefined` | Formatted base price (if `baseAmount` provided and `baseAmount > amount`) |
| `discount` | `string \| undefined` | Discount percentage string (if `baseAmount` > `amount`) |

**Note on guard**: The implementation checks `typeof amount !== 'number'` to return early. Since `typeof NaN === 'number'` in JavaScript, a `NaN` amount passes this guard and formats as `'$NaN'`. Always ensure `amount` is a valid finite number before calling `usePrice`.

**Example**:

```typescript
import usePrice from '@vercel/commerce-kibocommerce/product/use-price'

function ProductPrice({ amount, currencyCode }: { amount: number; currencyCode: string }) {
  const { price } = usePrice({ amount, currencyCode })
  return <span>{price}</span>
}

// With discount display
function ProductPriceWithDiscount({
  salePrice,
  retailPrice,
  currencyCode,
}: {
  salePrice: number
  retailPrice: number
  currencyCode: string
}) {
  const { price, basePrice, discount } = usePrice({
    amount: salePrice,
    baseAmount: retailPrice,
    currencyCode,
  })
  return (
    <div>
      <span>{price}</span>
      {basePrice && <s>{basePrice}</s>}
      {discount && <span>Save {discount}</span>}
    </div>
  )
}
```

## Server-Side Product Operations

For use in `getStaticProps` / `getServerSideProps`, see [Server-Side API](./server-api.md) for:
- `getAllProducts` — Fetch all products with pagination
- `getProduct` — Fetch a single product by slug/productCode
- `getAllProductPaths` — Fetch all product paths for SSG
