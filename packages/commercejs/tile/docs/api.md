# Server-Side API

The server-side API provides data operations for Next.js SSG/SSR pages and API route handlers for login and checkout endpoints.

## Import

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'
import type { CommercejsConfig, CommercejsAPI, Provider } from '@vercel/commerce-commercejs/api'

// For API route handlers
import commercejsAPI from '@vercel/commerce-commercejs/api/endpoints'

// Individual endpoint handlers (advanced)
import checkoutApi from '@vercel/commerce-commercejs/api/endpoints/checkout'
import loginApi from '@vercel/commerce-commercejs/api/endpoints/login'
```

## Capabilities

### getCommerceApi

Creates and returns the server-side Commerce.js API instance with all operations attached.

```typescript { .api }
/**
 * @param customProvider - Optional custom provider to override defaults
 * @returns CommercejsAPI instance with all data operations
 */
function getCommerceApi<P extends Provider>(
  customProvider?: P
): CommercejsAPI<P>
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

// Use the default provider
const commerce = getCommerceApi()

// Use a custom provider
import { provider } from '@vercel/commerce-commercejs/api'
const customCommerce = getCommerceApi({ ...provider, config: { ...provider.config, apiToken: 'custom' } })
```

### CommercejsConfig

Configuration interface for the API provider.

```typescript { .api }
interface CommercejsConfig extends CommerceAPIConfig {
  commerceUrl: string        // 'https://api.chec.io/v1'
  cartCookie: string         // 'commercejs_cart_id'
  cartCookieMaxAge: number   // 2592000 (30 days in seconds)
  customerCookie: string     // 'commercejs_customer_token'
  apiToken: string           // '' (not used; authentication via public key)
  sdkFetch: typeof sdkFetch  // Low-level SDK method caller
  fetch: GraphQLFetcher      // Throws error (not implemented)
}
```

**Note:** GraphQL is not implemented; `config.fetch` always throws a `FetcherError`.

### provider

The default provider object exported from the API module.

```typescript { .api }
const provider: {
  config: CommercejsConfig
  operations: {
    getAllPages: Function
    getPage: Function
    getSiteInfo: Function
    getAllProductPaths: Function
    getAllProducts: Function
    getProduct: Function
  }
}

type Provider = typeof provider
type CommercejsAPI<P extends Provider = Provider> = CommerceAPI<P | any>
```

### commercejsAPI (API Route Handler)

Creates Next.js API route handlers for the `login` and `checkout` endpoints. Used to register all Commerce.js API routes under a catch-all route.

```typescript { .api }
/**
 * @param commerce - CommercejsAPI instance from getCommerceApi()
 * @returns Next.js API route handler (NextApiHandler)
 */
function commercejsAPI(commerce: CommercejsAPI): NextApiHandler
```

**Registered endpoints:**
- `login` → handles `/api/login?token=:token` (magic-link callback)
- `checkout` → handles `/api/commerce/checkout` (order capture)

**Usage:**

```typescript
// pages/api/commerce/[...commerce].ts
import commercejsAPI from '@vercel/commerce-commercejs/api/endpoints'
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()
export default commercejsAPI(commerce)
```

### getAllPages

Returns all pages from Commerce.js. **Stub implementation** — always returns an empty pages array. Registered as part of the standard `@vercel/commerce` operations interface.

```typescript { .api }
/**
 * @param opts - Optional configuration override
 * @returns Empty pages array (stub)
 */
async function getAllPages(opts?: {
  config?: Partial<CommercejsConfig>
  preview?: boolean
}): Promise<{ pages: [] }>
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticProps() {
  const { pages } = await commerce.getAllPages()
  // pages is always []
  return { props: { pages } }
}
```

### getPage

Returns a single page by ID. **Stub implementation** — always returns an empty object. Registered as part of the standard `@vercel/commerce` operations interface.

```typescript { .api }
/**
 * @returns Empty object (stub)
 */
async function getPage(): Promise<{}>
```

**Usage:**

```typescript
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()

export async function getStaticProps() {
  const page = await commerce.getPage()
  // page is always {}
  return { props: { page } }
}
```

### Checkout API Endpoint

The checkout endpoint handles order capture from cart. Exposed as `checkoutApi` from `@vercel/commerce-commercejs/api/endpoints/checkout`.

```typescript { .api }
import type { CheckoutAPI, CheckoutEndpoint } from '@vercel/commerce-commercejs/api/endpoints/checkout'

// Handlers:
const handlers: {
  getCheckout: (...args: any[]) => Promise<{ data: null }>  // stub
  submitCheckout: CheckoutEndpoint['handlers']['submitCheckout']
}
```

**`submitCheckout` request body:**
```typescript
{
  item: {
    card: CardFields
    address: AddressFields
  }
  cartId: string
}
```

**`submitCheckout` process:**
1. `commerce.checkout.generateTokenFrom('cart', cartId)` → checkout token
2. `commerce.checkout.getShippingOptions(token, { country: 'US' })` → shipping methods
3. Selects first shipping method
4. Calls `normalizeTestCheckout()` to build checkout capture payload (test gateway)
5. `commerce.checkout.capture(token, checkoutData)` → order

**Response:** `{ data: null }`

### Login API Endpoint

Handles the magic-link login token callback. Exposed as `loginApi` from `@vercel/commerce-commercejs/api/endpoints/login`.

```typescript { .api }
import type { LoginAPI, LoginEndpoint } from '@vercel/commerce-commercejs/api/endpoints/login'
```

**Request:** `GET /api/login?token=:token`

**Handler behavior:**
1. Reads `token` query param
2. If no token: returns `{ redirectTo: deploymentUrl }`
3. Calls `commerce.customer.getToken(token, false)` → `{ jwt }`
4. Sets `Set-Cookie: commercejs_customer_token=<jwt>; Max-Age=86400; Path=/; [Secure]`

**Cookie settings:**
- `maxAge`: 86400 (24 hours)
- `path`: `/`
- `secure`: `true` in production, `false` in development

### sdkFetch Utility

Low-level utility function that calls Commerce.js SDK methods. Used internally by API operations.

```typescript { .api }
/**
 * Calls a method on the Commerce.js SDK
 * @param resource - SDK resource name (e.g., 'products', 'cart', 'checkout', 'customer', 'categories')
 * @param method - Method name on the resource (e.g., 'list', 'retrieve', 'add', 'capture')
 * @param variables - Arguments to pass to the method
 * @returns Promise resolving to the SDK method's return value
 */
async function sdkFetch<
  Resource extends keyof Commerce,
  Method extends MethodKeys<Commerce[Resource]>
>(
  resource: Resource,
  method: Method,
  ...variables: Parameters<Commerce[Resource][Method]>
): Promise<ReturnType<Commerce[Resource][Method]>>
```

**Common resource/method combinations:**
| Resource | Method | Description |
|----------|--------|-------------|
| `'products'` | `'list'` | List all products |
| `'products'` | `'retrieve'` | Get product by ID or permalink |
| `'products'` | `'getVariants'` | Get product variants |
| `'cart'` | `'retrieve'` | Get current cart |
| `'cart'` | `'add'` | Add item to cart |
| `'cart'` | `'update'` | Update cart item |
| `'cart'` | `'remove'` | Remove cart item |
| `'checkout'` | `'generateTokenFrom'` | Generate checkout token |
| `'checkout'` | `'getShippingOptions'` | Get available shipping |
| `'checkout'` | `'capture'` | Capture/place order |
| `'customer'` | `'login'` | Send magic-link email |
| `'customer'` | `'getToken'` | Exchange login token for JWT |
| `'customer'` | `'_request'` | Make authenticated API request |
| `'categories'` | `'list'` | List all categories |

## Next.js Configuration

The `next.config` export provides Next.js configuration for images and URL rewrites required by the login flow.

```typescript { .api }
// @vercel/commerce-commercejs/next.config (CommonJS)
module.exports = {
  commerce: {
    provider: 'commercejs',
    features: {
      cart: true,
      search: true,
      customCheckout: true,
      customerAuth: true,
      wishlist: false,
    }
  },
  images: {
    domains: ['cdn.chec.io'],  // Commerce.js CDN for product images
  },
  rewrites(): Array<{
    source: '/api/login/:token',
    destination: '/api/login?token=:token',
  }>
}
```

**Usage in `next.config.js`:**

```javascript
const commercejsConfig = require('@vercel/commerce-commercejs/next.config')

module.exports = {
  ...commercejsConfig,
  // Merge or extend as needed
  images: {
    domains: [
      ...commercejsConfig.images.domains,
      // additional domains
    ],
  },
}
```
