# Server-Side API

Server-side API for use in Next.js `getStaticProps`, `getServerSideProps`, and API routes. This module (`@vercel/commerce-kibocommerce/api`) is Node.js-only and should not be imported in client-side code.

## Import

```typescript
import { getCommerceApi, provider } from '@vercel/commerce-kibocommerce/api'
import type { KiboCommerceConfig, KiboCommerceAPI, KiboCommerceProvider } from '@vercel/commerce-kibocommerce/api'
```

## Capabilities

### provider

The default server-side provider object. Contains the default config (populated from environment variables) and all API operations. Can be spread and overridden to create a custom provider.

```typescript { .api }
import { provider } from '@vercel/commerce-kibocommerce/api'

const provider: {
  config: KiboCommerceConfig
  operations: {
    getAllPages: Function
    getPage: Function
    getSiteInfo: Function
    getCustomerWishlist: Function
    getAllProductPaths: Function
    getAllProducts: Function
    getProduct: Function
  }
}
```

**Usage** — pass a modified provider to `getCommerceApi()` to override config values:

```typescript
import { getCommerceApi, provider } from '@vercel/commerce-kibocommerce/api'

const commerce = getCommerceApi({
  ...provider,
  config: {
    ...provider.config,
    currencyCode: 'EUR',
  },
})
```

### getCommerceApi

Factory function that creates a `KiboCommerceAPI` instance with the given provider configuration.

```typescript { .api }
import { getCommerceApi, provider } from '@vercel/commerce-kibocommerce/api'

function getCommerceApi<P extends KiboCommerceProvider>(
  customProvider?: P
): KiboCommerceAPI<P>

// Default provider (uses environment variables)
const commerce = getCommerceApi()

// Custom provider with config overrides
const commerce = getCommerceApi({
  ...provider,
  config: {
    ...provider.config,
    currencyCode: 'EUR',
  }
})
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `customProvider` | `KiboCommerceProvider?` | Optional provider override; defaults to the built-in provider |

**Returns**: `KiboCommerceAPI` instance with all operation methods available.

### KiboCommerceConfig Interface

Configuration shape. The default config is populated from environment variables.

```typescript { .api }
interface KiboCommerceConfig extends CommerceAPIConfig {
  // GraphQL API
  commerceUrl: string          // KIBO_API_URL env var
  apiToken: string             // KIBO_API_TOKEN env var

  // REST API (for OAuth)
  apiHost?: string             // KIBO_API_HOST env var
  authUrl?: string             // KIBO_AUTH_URL env var
  clientId?: string            // KIBO_CLIENT_ID env var
  sharedSecret?: string        // KIBO_SHARED_SECRET env var

  // Session cookies
  cartCookie: string           // KIBO_CART_COOKIE env var
  cartCookieMaxAge: number     // Seconds; default: 2592000 (30 days)
  customerCookie: string       // KIBO_CUSTOMER_COOKIE env var
  customerCookieMaxAgeInDays: number  // Default: 30

  // Commerce settings
  currencyCode: string         // Default: 'USD'
  documentListName: string     // Default: 'siteSnippets@mozu'
  defaultWishlistName: string  // Default: 'My Wishlist'
}
```

## API Operations

Accessed via the `commerce` instance returned by `getCommerceApi()`.

### getAllProducts

Fetch all products from the Kibo catalog with optional filtering and pagination.

```typescript { .api }
commerce.getAllProducts(opts?: {
  query?: string
  variables?: {
    filter?: string
    startIndex?: number
    pageSize?: number
  }
  config?: Partial<KiboCommerceConfig>
  preview?: boolean
}): Promise<{ products: Product[] }>
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `string` | Custom GraphQL query string (overrides default) |
| `variables.filter` | `string` | Kibo filter string (e.g., `'categoryCode req shoes'`) |
| `variables.startIndex` | `number` | Pagination start index |
| `variables.pageSize` | `number` | Number of products per page |
| `config` | `Partial<KiboCommerceConfig>` | Config overrides |
| `preview` | `boolean` | Whether to fetch preview content |

**Returns**: `{ products: Product[] }` — Array of normalized `Product` objects.

**Example**:

```typescript
// pages/products.tsx
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'

export async function getStaticProps() {
  const commerce = getCommerceApi()
  const { products } = await commerce.getAllProducts({
    variables: { pageSize: 20, startIndex: 0 },
  })
  return { props: { products } }
}
```

### getProduct

Fetch a single product by slug (maps to Kibo `productCode`).

```typescript { .api }
commerce.getProduct(opts?: {
  query?: string
  variables?: { slug: string }
  config?: Partial<KiboCommerceConfig>
  preview?: boolean
}): Promise<{ product: Product } | {}>
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `variables.slug` | `string` | Product slug/productCode |
| `query` | `string` | Custom GraphQL query string |
| `config` | `Partial<KiboCommerceConfig>` | Config overrides |
| `preview` | `boolean` | Preview mode flag |

**Returns**: `{ product: Product }` or empty object `{}` if not found.

**Example**:

```typescript
// pages/products/[slug].tsx
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'

export async function getStaticProps({ params }) {
  const commerce = getCommerceApi()
  const { product } = await commerce.getProduct({
    variables: { slug: params.slug },
  })

  if (!product) return { notFound: true }
  return { props: { product } }
}
```

### getAllProductPaths

Fetch all product paths for static site generation. Uses `startIndex: 0, pageSize: 100`.

```typescript { .api }
commerce.getAllProductPaths(opts?: {
  config?: KiboCommerceConfig
}): Promise<{ products: Array<{ path: string }> }>
```

**Returns**: `{ products: Array<{ path: string }> }` where `path` is `'/<productCode>'`.

**Example**:

```typescript
// pages/products/[slug].tsx
export async function getStaticPaths() {
  const commerce = getCommerceApi()
  const { products } = await commerce.getAllProductPaths()
  return {
    paths: products.map((p) => ({ params: { slug: p.path.replace('/', '') } })),
    fallback: 'blocking',
  }
}
```

### getSiteInfo

Fetch site category tree (taxonomy). Brands are not supported and always return `[]`.

```typescript { .api }
commerce.getSiteInfo(opts?: {
  query?: string
  variables?: any
  config?: Partial<KiboCommerceConfig>
  preview?: boolean
}): Promise<{ categories: Category[]; brands: [] }>

interface Category {
  id: string
  name: string
  slug: string
  path: string   // '/<slug>'
}
```

**Returns**: `{ categories: Category[], brands: [] }` — Normalized category tree from Kibo.

**Example**:

```typescript
export async function getStaticProps() {
  const commerce = getCommerceApi()
  const { categories } = await commerce.getSiteInfo()
  return { props: { categories } }
}
```

### getAllPages

Fetch all CMS pages from the Kibo document list (`documentListName` config).

```typescript { .api }
commerce.getAllPages(opts?: {
  url?: string
  config?: Partial<KiboCommerceConfig>
  preview?: boolean
  query?: string
}): Promise<{ pages: Page[] }>

interface Page {
  id: string
  name: string
  url: string
  body: string
  is_visible: boolean
  sort_order: number
}
```

**Returns**: `{ pages: Page[] }` — All documents from the configured document list.

**Example**:

```typescript
export async function getStaticProps() {
  const commerce = getCommerceApi()
  const { pages } = await commerce.getAllPages()
  return { props: { pages } }
}
```

### getPage

Fetch a single CMS page by ID. Respects `preview` flag: in non-preview mode, only returns visible pages (`is_visible === true`).

```typescript { .api }
commerce.getPage(opts: {
  url?: string
  variables: { id: string }
  config?: Partial<KiboCommerceConfig>
  preview?: boolean
}): Promise<{ page: Page } | {}>
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `variables.id` | `string` | Page/document ID |
| `preview` | `boolean` | If `true`, returns page regardless of `is_visible` |
| `config` | `Partial<KiboCommerceConfig>` | Config overrides |

**Returns**: `{ page: Page }` or empty `{}` if not found or not visible.

**Example**:

```typescript
export async function getStaticProps({ params }) {
  const commerce = getCommerceApi()
  const { page } = await commerce.getPage({ variables: { id: params.id } })
  if (!page) return { notFound: true }
  return { props: { page } }
}
```

### getCustomerWishlist

Fetch a customer's wishlist by customer ID and wishlist name. Used server-side; client-side wishlist uses `useWishlist` hook.

```typescript { .api }
commerce.getCustomerWishlist(opts: {
  variables: { customerId: string; wishlistName: string }
  config?: KiboCommerceConfig
  includeProducts?: boolean
}): Promise<{ wishlist: Wishlist | {} }>
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `variables.customerId` | `string` | Customer account ID (integer as string) |
| `variables.wishlistName` | `string` | Wishlist name to query (use `config.defaultWishlistName` = `'My Wishlist'`) |
| `includeProducts` | `boolean` | Whether to include full product details |
| `config` | `KiboCommerceConfig` | Config overrides |

**Returns**: `{ wishlist: Wishlist }` or `{ wishlist: {} }` on error/not found.

**Note**: Errors are caught internally and return empty wishlist rather than throwing. The endpoint handlers auto-create the wishlist if it doesn't exist when adding items.

## Internal API Utilities

### APIAuthenticationHelper

Handles Kibo OAuth application-level authentication. Used internally by the server-side fetcher.

```typescript { .api }
import { APIAuthenticationHelper } from '@vercel/commerce-kibocommerce/api/utils/api-auth-helper'

class APIAuthenticationHelper {
  constructor(
    config: KiboCommerceConfig,
    authTicketCache?: AuthTicketCache
  )

  authenticate(): Promise<AppAuthTicket>
  refreshTicket(kiboAuthTicket: AppAuthTicket): Promise<AppAuthTicket>
  getAccessToken(): Promise<string>
}

interface AppAuthTicket {
  access_token: string
  token_type: string
  expires_in: number
  expires_at: number
  refresh_token: string | null
}

interface AuthTicketCache {
  getAuthTicket(): Promise<AppAuthTicket | undefined>
  setAuthTicket(ticket: AppAuthTicket): void
}
```

**Methods**:

| Method | Description |
|--------|-------------|
| `authenticate()` | Performs OAuth client credentials grant; stores ticket in cache |
| `refreshTicket(ticket)` | Refreshes an expired auth ticket |
| `getAccessToken()` | Returns valid access token; auto-authenticates or refreshes as needed |

The default cache implementation uses a module-level in-memory object that persists across requests during development. The cache checks `expires_at` (milliseconds) to detect expiry.

Auth endpoint: `POST {authUrl}/api/platform/applications/authtickets/oauth`

Refresh endpoint: `POST {authUrl}/api/platform/applications/authtickets/refresh-ticket`

### CookieHandler

Manages shopper session cookies in API route handlers. Used internally by cart, customer, and wishlist endpoint handlers.

```typescript { .api }
import CookieHandler from '@vercel/commerce-kibocommerce/api/utils/cookie-handler'

class CookieHandler {
  headers: HeadersInit | undefined   // Set-Cookie header after setAnonymousShopperCookie()

  constructor(config: KiboCommerceConfig, req: NextRequest)

  getAnonymousToken(): Promise<{ response: AnonymousShopperToken; accessToken: string }>
  isShopperCookieAnonymous(): boolean
  setAnonymousShopperCookie(anonymousShopperTokenResponse: AnonymousShopperToken): void
  getAccessToken(): string | null
}
```

**Constructor**: Reads and decodes the `customerCookie` from `req.cookies`; extracts `accessToken`.

**Methods**:

| Method | Returns | Description |
|--------|---------|-------------|
| `getAnonymousToken()` | `Promise<{response, accessToken}>` | Fetches anonymous shopper token from Kibo GraphQL |
| `isShopperCookieAnonymous()` | `boolean` | `true` if no `customerAccount` in cookie session |
| `setAnonymousShopperCookie(response)` | `void` | Populates `this.headers` with base64-encoded Set-Cookie |
| `getAccessToken()` | `string \| null` | Returns decoded access token from cookie, or `null` |

Cookie values are base64-encoded JSON strings. The `prepareSetCookie` utility handles encoding.

## Normalize Functions

These functions convert Kibo-specific GraphQL types to standard `@vercel/commerce` types. They are used internally by operations and endpoint handlers.

```typescript { .api }
import {
  normalizeProduct,
  normalizePage,
  normalizeCart,
  normalizeCustomer,
  normalizeCategory,
  normalizeWishlistItem,
} from '@vercel/commerce-kibocommerce/lib/normalize'

function normalizeProduct(productNode: KiboProduct, config: { currencyCode: string }): Product
function normalizePage(page: KiboDocument): Page
function normalizeCart(data: KiboCart): Cart
function normalizeCustomer(customer: KiboCustomerAccountInput): Customer
function normalizeCategory(category: KiboPrCategory): Category
function normalizeWishlistItem(item: KiboWishlistItem, config: { currencyCode: string }, includeProducts?: boolean): WishlistItem
```

## Utility Functions

```typescript { .api }
import { getCookieExpirationDate } from '@vercel/commerce-kibocommerce/lib/get-cookie-expiration-date'
import getSlug from '@vercel/commerce-kibocommerce/lib/get-slug'
import { prepareSetCookie } from '@vercel/commerce-kibocommerce/lib/prepare-set-cookie'
import { buildProductSearchVars } from '@vercel/commerce-kibocommerce/lib/product-search-vars'
import { setCookies } from '@vercel/commerce-kibocommerce/lib/set-cookie'

// Calculate cookie expiration date
function getCookieExpirationDate(maxAgeInDays: number): Date

// Strip leading/trailing slashes from path
function getSlug(path: string): string

// Build Set-Cookie header value with base64-encoded value
function prepareSetCookie(name: string, value: string, options?: {
  maxAge?: number    // Max-Age in seconds
  expires?: Date     // Expires date (used only if maxAge not set)
}): string

// Build Kibo GraphQL product search variables from UI input
function buildProductSearchVars(opts: {
  categoryCode?: string
  pageSize?: number
  filters?: Record<string, string[]>  // facet filters { [facetKey]: [values] }
  startIndex?: number
  sort?: 'latest-desc' | 'price-asc' | 'price-desc' | 'trending-desc' | ''
  search?: string
}): {
  query: string
  startIndex: number
  pageSize: number
  sortBy: string
  filter: string
  facetTemplate: string
  facetValueFilter: string
}

// Set multiple Set-Cookie response headers
function setCookies(res: any, cookies: string[]): void
```

## Client-Side Fetcher Module

The client-side HTTP fetcher function. This is used internally by the provider but can be imported directly for custom integrations.

```typescript { .api }
import fetcher from '@vercel/commerce-kibocommerce/fetcher'

// Client-side HTTP fetch function conforming to the @vercel/commerce Fetcher interface.
// Automatically sets Content-Type: application/json when body or variables are provided.
// Parses the JSON response extracting the `data` field from { data: ... } wrapper.
// Throws FetcherError on non-OK HTTP responses.
const fetcher: Fetcher
```

## Internal API Utility Functions

Low-level utilities used by endpoint handlers internally.

```typescript { .api }
import getAnonymousShopperToken from '@vercel/commerce-kibocommerce/api/utils/get-anonymous-shopper-token'
import getCustomerId from '@vercel/commerce-kibocommerce/api/utils/get-customer-id'

// Fetches an anonymous shopper token from Kibo GraphQL using the anonymous shopper token query
async function getAnonymousShopperToken(opts: {
  config: KiboCommerceConfig
}): Promise<string | undefined>

// Decodes the customer cookie and fetches the customer account ID from Kibo GraphQL.
// Uses `x-vol-user-claims` header with the decoded access token.
async function getCustomerId(opts: {
  customerToken: string   // base64-encoded JSON string with { accessToken }
  config: KiboCommerceConfig
}): Promise<string | undefined>
```
