# Product Catalog

Product search hook for client-side filtering and sorting, plus server-side operations for SSG/SSR product fetching.

## Import

```typescript
// Client-side hooks
import { useSearch, usePrice } from '@vercel/commerce-commercejs/product'

// Server-side operations (used via getCommerceApi())
import { getCommerceApi } from '@vercel/commerce-commercejs/api'
const commerce = getCommerceApi()
// commerce.getAllProducts(), commerce.getProduct(), etc.
```

## Types

```typescript { .api }
interface Product {
  id: string
  name: string
  description: string       // Plain text description
  descriptionHtml: string   // Same as description (HTML not distinguished)
  slug: string              // Product permalink
  path: string              // URL path (e.g., '/my-product')
  images: Array<{
    url: string
    alt: string             // image description or filename
  }>
  price: {
    value: number           // Raw price number
    currencyCode: string    // Always 'USD' in current implementation
  }
  variants: ProductVariant[]
  options: ProductOption[]
}

interface ProductVariant {
  id: string
  sku: string
  options: Array<{
    id: string               // Option value ID
    displayName: string      // Option group name (e.g., 'Color', 'Size')
    __typename: 'MultipleChoiceOption'
    values: Array<{ label: string }>  // Option value label (e.g., 'Red', 'Large')
  }>
}

interface ProductOption {
  id: string           // Variant group ID
  displayName: string  // Variant group name (e.g., 'Color', 'Size')
  values: Array<{ label: string }>  // All available option values
}
```

## Capabilities

### useSearch

Searches and lists products from Commerce.js with optional filtering by search query, category, or brand, and optional sorting. Uses SWR for caching.

```typescript { .api }
/**
 * @param input - Search parameters (all optional)
 * @returns SWR response with matching products and a found flag
 */
function useSearch(input?: SearchInput): {
  data: {
    products: Product[]
    found: boolean     // true if pagination.total > 0
  } | null | undefined
  error: any
  isLoading: boolean
}

interface SearchInput {
  search?: string                 // Full-text search query
  categoryId?: string | number    // Filter by Commerce.js category ID
  brandId?: string | number       // Filter by brand ID (passed but not used in Commerce.js)
  sort?: SortOption
  swrOptions?: SWROptions
}

type SortOption =
  | 'trending-desc'   // Sort by updated date, descending
  | 'latest-desc'     // Sort by updated date, descending
  | 'price-asc'       // Sort by price, ascending
  | 'price-desc'      // Sort by price, descending
```

**Sort mapping:**
| `sort` value | Commerce.js params |
|---|---|
| `'trending-desc'` | `sortBy: 'updated', sortDirection: 'desc'` |
| `'latest-desc'` | `sortBy: 'updated', sortDirection: 'desc'` |
| `'price-asc'` | `sortBy: 'price', sortDirection: 'asc'` |
| `'price-desc'` | `sortBy: 'price', sortDirection: 'desc'` |

**Default SWR options:** `revalidateOnFocus: false`

**Usage:**

```typescript
import { useSearch } from '@vercel/commerce-commercejs/product'

function ProductList() {
  const { data, isLoading } = useSearch({
    search: 'shirt',
    categoryId: 'cat_abc123',
    sort: 'price-asc',
  })

  if (isLoading) return <div>Loading...</div>
  if (!data?.found) return <div>No products found</div>

  return (
    <ul>
      {data.products.map(product => (
        <li key={product.id}>
          <a href={product.path}>{product.name}</a>
          <span>${product.price.value}</span>
          {product.images[0] && (
            <img src={product.images[0].url} alt={product.images[0].alt} />
          )}
        </li>
      ))}
    </ul>
  )
}

// List all products (no filters)
function AllProducts() {
  const { data } = useSearch()
  return <div>{data?.products.length} products</div>
}
```

### usePrice

Re-exported from `@vercel/commerce/product/use-price`. Formats a price value according to locale and currency.

```typescript { .api }
// Re-export from @vercel/commerce
export { usePrice, default } from '@vercel/commerce/product/use-price'
```

**Usage:** Refer to `@vercel/commerce` documentation for `usePrice` details.

## Server-Side Operations

All server-side operations are accessed via the `CommercejsAPI` instance from `getCommerceApi()`. These are intended for use in Next.js `getStaticProps`, `getServerSideProps`, and API routes.

### getAllProducts

Fetches all products from Commerce.js, sorted by sort_order.

```typescript { .api }
/**
 * @param opts - Optional configuration override
 * @returns All products normalized to @vercel/commerce Product type
 */
async function getAllProducts(opts?: {
  config?: Partial<CommercejsConfig>
}): Promise<{ products: Product[] }>
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticProps() {
  const { products } = await commerce.getAllProducts()
  return { props: { products } }
}
```

### getAllProductPaths

Fetches all product permalinks for use in static path generation.

```typescript { .api }
/**
 * @param opts - Optional configuration override
 * @returns Array of product URL paths
 */
async function getAllProductPaths(opts?: {
  config?: Partial<CommercejsConfig>
}): Promise<{ products: Array<{ path: string }> }>
```

**Returns:** Each product as `{ path: '/permalink' }` (permalink prefixed with `/`)

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticPaths() {
  const { products } = await commerce.getAllProductPaths()
  return {
    paths: products.map(({ path }) => ({ params: { slug: path.slice(1) } })),
    fallback: false,
  }
}
```

### getProduct

Fetches a single product by its permalink/slug, including all variants.

```typescript { .api }
/**
 * @param opts - Query options including the product slug
 * @returns Single product with variants, or undefined if not found
 */
async function getProduct(opts?: {
  variables?: { slug: string }   // Product permalink/slug to look up
  config?: Partial<CommercejsConfig>
  preview?: boolean
}): Promise<{ product: Product | undefined }>
```

**Implementation detail:** Fetches product by permalink type, then separately fetches variants via `products.getVariants(productId)`.

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticProps({ params }) {
  const { product } = await commerce.getProduct({
    variables: { slug: params.slug },
  })

  if (!product) return { notFound: true }
  return { props: { product } }
}
```

### getSiteInfo

Fetches product categories. Brands are not supported (returns empty array).

```typescript { .api }
/**
 * @param opts - Optional configuration override
 * @returns Categories list and empty brands array
 */
async function getSiteInfo(opts?: {
  config?: Partial<CommercejsConfig>
  preview?: boolean
}): Promise<{
  categories: Category[]
  brands: []
}>

interface Category {
  id: string
  name: string
  slug: string
  path: string   // '/' + slug
}
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticProps() {
  const { categories } = await commerce.getSiteInfo()
  return { props: { categories } }
}
```
