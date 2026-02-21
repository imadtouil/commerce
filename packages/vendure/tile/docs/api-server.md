# Server-Side API

Server-side API layer for use in Next.js `getStaticProps`, `getServerSideProps`, and API routes. Provides operations to fetch products, site information, customer wishlist, and handle authentication. Also registers Next.js API route handlers.

## Import

```typescript
// Server-side operations
import { getCommerceApi, provider, type VendureAPI, type VendureConfig, type Provider } from '@vercel/commerce-vendure/api'

// Next.js API route handler factory
import vendureAPI from '@vercel/commerce-vendure/api/endpoints'
```

## Types

```typescript { .api }
interface VendureConfig extends CommerceAPIConfig {
  commerceUrl: string         // NEXT_PUBLIC_VENDURE_SHOP_API_URL
  apiToken: string            // '' (empty, session-based auth)
  cartCookie: string          // '' (managed by Vendure session)
  customerCookie: string      // ''
  cartCookieMaxAge: number    // 30 days in seconds (2592000)
  fetch: GraphQLFetcher
}

interface Category {
  id: string
  name: string
  description: string
  slug: string
  path: string               // /${id}
  productCount: number       // productVariants.totalItems
  parent?: { id: string } | null
  children: Category[]       // nested child categories (tree structure from arrayToTree)
  expanded: boolean          // tree expansion state, defaults to false
}

interface Product {
  id: string
  name: string
  description: string
  slug: string
  path: string           // /${slug}
  images: Array<{ url: string; alt: string }>
  variants: Array<{
    id: string
    options: Array<{
      __typename: 'MultipleChoiceOption'
      id: string
      displayName: string   // option group name
      values: Array<{ label: string }>
    }>
  }>
  price: {
    value: number           // priceWithTax / 100
    currencyCode: string
  }
  options: Array<{
    id: string
    displayName: string     // option group name
    values: Array<{ label: string }>
  }>
}

type Provider = typeof provider
type VendureAPI<P extends Provider = Provider> = CommerceAPI<P>
```

## Capabilities

### getCommerceApi

Creates and returns a `CommerceAPI` instance with all Vendure operations pre-configured. Reads `NEXT_PUBLIC_VENDURE_SHOP_API_URL` from environment. Throws if the env variable is not set.

```typescript { .api }
/**
 * Creates and returns a CommerceAPI instance for Vendure.
 * @param customProvider - Optional custom provider to override defaults
 * @returns CommerceAPI instance with all operations
 * @throws Error if NEXT_PUBLIC_VENDURE_SHOP_API_URL is not set
 */
function getCommerceApi<P extends Provider>(
  customProvider?: P
): CommerceAPI<P>
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-vendure/api'

// pages/index.tsx
export async function getStaticProps() {
  const commerce = getCommerceApi()
  const { products } = await commerce.getAllProducts({ variables: { first: 12 } })

  return {
    props: { products },
    revalidate: 60,
  }
}
```

### getAllProducts

Fetches a list of products using Vendure's search API.

```typescript { .api }
/**
 * Fetches all products (via search API).
 * @param opts.variables.first - Number of products to fetch (take)
 * @param opts.config - Partial VendureConfig overrides
 * @returns { products: Product[] }
 */
async function getAllProducts(opts?: {
  variables?: { first?: number }
  config?: Partial<VendureConfig>
  preview?: boolean
}): Promise<{ products: Product[] }>
```

**Usage:**

```typescript
const commerce = getCommerceApi()
const { products } = await commerce.getAllProducts({ variables: { first: 24 } })
```

### getProduct

Fetches a single product by its slug. Returns the full product with variants and options.

```typescript { .api }
/**
 * Fetches a product by slug.
 * @param opts.variables.slug - Product slug identifier
 * @param opts.config - Partial VendureConfig overrides
 * @returns { product: Product } if found, or {} if not found
 */
async function getProduct(opts: {
  variables: { slug: string }
  config?: Partial<VendureConfig>
  preview?: boolean
}): Promise<{ product: Product } | {}>
```

**Usage:**

```typescript
// pages/products/[slug].tsx
export async function getStaticProps({ params }) {
  const commerce = getCommerceApi()
  const { product } = await commerce.getProduct({
    variables: { slug: params.slug },
  })

  if (!product) {
    return { notFound: true }
  }

  return {
    props: { product },
    revalidate: 60,
  }
}
```

### getAllProductPaths

Fetches all product slugs for static path generation.

```typescript { .api }
/**
 * Fetches all product slugs for static path generation.
 * @param opts.variables.first - Number of products to fetch (default: 100)
 * @param opts.config - VendureConfig overrides
 * @returns { products: Array<{ path: string }> } where path is /${slug}
 */
async function getAllProductPaths(opts?: {
  variables?: { first?: number }
  config?: VendureConfig
}): Promise<{ products: Array<{ path: string }> }>
```

**Usage:**

```typescript
// pages/products/[slug].tsx
export async function getStaticPaths() {
  const commerce = getCommerceApi()
  const { products } = await commerce.getAllProductPaths()

  return {
    paths: products.map(({ path }) => `/products${path}`),
    fallback: 'blocking',
  }
}
```

### getSiteInfo

Fetches site-wide information: categories (Vendure collections organized as a tree) and brands (always empty for Vendure).

```typescript { .api }
/**
 * Fetches categories and brands for the site.
 * Categories are built from Vendure collections in a tree structure.
 * Brands are always an empty array (not supported by Vendure).
 * @param opts.config - Partial VendureConfig overrides
 * @returns { categories: Category[], brands: [] }
 */
async function getSiteInfo(opts?: {
  query?: string
  variables?: any
  config?: Partial<VendureConfig>
  preview?: boolean
}): Promise<{ categories: Category[]; brands: any[] }>
```

**Usage:**

```typescript
const commerce = getCommerceApi()
const { categories } = await commerce.getSiteInfo()
// categories is a tree of Vendure collections with children
```

### login

Server-side login operation. Authenticates a customer and returns their ID.

```typescript { .api }
/**
 * Authenticates a customer server-side.
 * @param opts.variables.username - Customer email address
 * @param opts.variables.password - Customer password
 * @param opts.res - HTTP Response object (for setting cookies)
 * @param opts.config - Partial VendureConfig overrides
 * @returns { result: string } where result is the CurrentUser ID
 * @throws ValidationError for NativeAuthStrategyError, InvalidCredentialsError, NotVerifiedError
 */
async function login(opts: {
  variables: { username: string; password: string }
  res: Response
  config?: Partial<VendureConfig>
}): Promise<{ result: string }>
```

### getAllPages

Returns an empty array of pages (not implemented for Vendure).

```typescript { .api }
/**
 * Returns empty pages list. Not implemented for Vendure.
 * @returns { pages: [] }
 */
async function getAllPages(opts?: {
  config?: Partial<VendureConfig>
  preview?: boolean
}): Promise<{ pages: [] }>
```

### getPage

Returns an empty object. Not implemented for Vendure.

```typescript { .api }
/**
 * Returns empty page. Not implemented for Vendure.
 * @param opts.variables.id - Page ID (unused)
 * @returns {}
 */
async function getPage(opts: {
  variables: { id: number }
  config?: Partial<VendureConfig>
  preview?: boolean
}): Promise<{}>
```

### getCustomerWishlist

Not implemented for Vendure (wishlist is not a built-in Vendure feature).

```typescript { .api }
/**
 * Returns empty wishlist. Not implemented for Vendure.
 * @param opts.variables - Any wishlist variables (unused)
 * @param opts.includeProducts - Whether to include products (unused)
 * @returns { wishlist: {} }
 */
async function getCustomerWishlist(opts: {
  variables: any
  config?: Partial<VendureConfig>
  includeProducts?: boolean
}): Promise<{ wishlist: {} }>
```

## Next.js API Route Handler

### vendureAPI

Creates a Next.js API route handler for Vendure. Registers the checkout endpoint.

```typescript { .api }
/**
 * Creates Next.js API handlers for Vendure endpoints.
 * Currently registers: checkout endpoint (returns stub HTML).
 * @param commerce - VendureAPI instance from getCommerceApi()
 * @returns Next.js API route handler
 */
function vendureAPI(commerce: VendureAPI): NextApiHandler
```

**Usage in `pages/api/commerce/[...pages].ts`:**

```typescript
import { getCommerceApi } from '@vercel/commerce-vendure/api'
import vendureAPI from '@vercel/commerce-vendure/api/endpoints'

const commerce = getCommerceApi()
export default vendureAPI(commerce)
```

## Checkout Endpoint

The checkout endpoint is registered but returns a stub HTML page (not implemented):

```typescript { .api }
// Available at: /api/commerce/checkout
// Returns HTML indicating checkout is not yet implemented
const handlers: CheckoutEndpoint['handlers'] = {
  getCheckout: () => Promise<{ html: string; headers: { 'Content-Type': 'text/html' } }>
}
```

## Internal GraphQL Fetcher (fetch-graphql-api.ts)

Used internally by all server-side operations.

```typescript { .api }
/**
 * Internal GraphQL fetcher for server-side API operations.
 * Reads commerceUrl from VendureConfig.
 * @param query - GraphQL query string
 * @param { variables } - Query variables
 * @param headers - Additional HTTP headers
 * @returns { data: T, res: Response }
 * @throws FetcherError if GraphQL errors are present in response
 */
const fetchGraphqlApi: GraphQLFetcher
```

## Vendure GraphQL Queries Used

### getAllProducts / useSearch
```graphql
query getAllProducts($input: SearchInput!) {
  search(input: $input) {
    items { ...SearchResult }
  }
}
```

### getProduct
```graphql
query getProduct($slug: String!) {
  product(slug: $slug) {
    id name slug description
    assets { id preview name }
    variants {
      id priceWithTax currencyCode
      options { id name code groupId group { id options { name } } }
    }
    optionGroups { id code name options { id name } }
  }
}
```

### getAllProductPaths
```graphql
query getAllProductPaths($first: Int = 100) {
  products(options: { take: $first }) {
    items { slug }
  }
}
```

### getSiteInfo (getCollections)
```graphql
query getCollections {
  collections {
    items {
      id name description slug
      productVariants { totalItems }
      parent { id }
      children { id }
    }
  }
}
```
