# Provider Setup & Configuration

Setup the Vendure commerce provider in a Next.js application, configure image domains, and access the commerce context from any component.

## Capabilities

### CommerceProvider

The root React context provider for Vendure commerce. Wrap your application with this component to enable all commerce hooks.

```typescript { .api }
/**
 * Root provider that sets up the Vendure commerce context.
 * Must wrap the application (typically in _app.tsx).
 * @param locale - BCP 47 locale string (e.g., 'en-us'). Defaults to 'en-us'.
 * @param children - Child React components
 */
const CommerceProvider: React.FC<{
  locale?: string
  children: React.ReactNode
}>
```

**Usage:**

```typescript
import { CommerceProvider } from '@vercel/commerce-vendure'

// In _app.tsx
export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

### useCommerce

React hook to access the commerce context. Must be used within a `CommerceProvider`.

```typescript { .api }
/**
 * Returns the commerce context value.
 * Must be used inside a CommerceProvider.
 */
function useCommerce(): CommerceContextValue
```

**Usage:**

```typescript
import { useCommerce } from '@vercel/commerce-vendure'

function MyComponent() {
  const commerce = useCommerce()
  // Access commerce context properties
}
```

### vendureProvider

The raw Vendure provider configuration object. Used internally by `CommerceProvider`. Can be passed to `getCommerceProvider` for custom setups.

```typescript { .api }
const vendureProvider: {
  locale: 'en-us'
  cartCookie: 'session'
  fetcher: Fetcher
  cart: {
    useCart: SWRHook<GetCartHook>
    useAddItem: MutationHook<AddItemHook>
    useUpdateItem: MutationHook<UpdateItemHook>
    useRemoveItem: MutationHook<RemoveItemHook>
  }
  customer: {
    useCustomer: SWRHook<CustomerHook>
  }
  products: {
    useSearch: SWRHook<SearchProductsHook>
  }
  auth: {
    useLogin: MutationHook<LoginHook>
    useLogout: MutationHook<LogoutHook>
    useSignup: MutationHook<SignupHook>
  }
}

type VendureProvider = typeof vendureProvider
```

**Import:**

```typescript
import { vendureProvider, type VendureProvider } from '@vercel/commerce-vendure'
```

### Fetcher

The client-side GraphQL fetcher used by all hooks to communicate with the Vendure Shop API.

```typescript { .api }
/**
 * GraphQL fetcher for the Vendure Shop API.
 * Reads endpoint from environment variables (NEXT_PUBLIC_VENDURE_LOCAL_URL takes priority,
 * falls back to NEXT_PUBLIC_VENDURE_SHOP_API_URL).
 * Sends POST requests with GraphQL query and variables.
 * Uses credentials: 'include' for session cookie support.
 *
 * @throws {Error} If no Shop API URL is configured
 * @throws {FetcherError} On HTTP errors or GraphQL errors in the response
 */
const fetcher: Fetcher
```

**Import:**

```typescript
import { fetcher } from '@vercel/commerce-vendure/fetcher'
```

### Next.js Configuration

The Next.js configuration module, exposed as `@vercel/commerce-vendure/next.config`.

```typescript { .api }
/**
 * Returns Next.js configuration for Vendure commerce.
 * Configures:
 *   - commerce.provider: 'vendure'
 *   - commerce.features.wishlist: false
 *   - images.domains: ['localhost', 'demo.vendure.io', 'readonlydemo.vendure.io']
 */
module.exports: {
  commerce: {
    provider: string   // 'vendure'
    features: {
      wishlist: boolean  // false
    }
  }
  images: {
    domains: string[]
  }
}
```

**Usage in next.config.js:**

```javascript
const { commerce } = require('@vercel/commerce-vendure/next.config')

module.exports = {
  ...commerce,
  // Your additional Next.js config
}

// Or spread just the commerce config into withCommerce:
const withCommerce = require('@vercel/commerce/config')
module.exports = withCommerce({
  ...commerce,
})
```

### Commerce Config (commerce.config.json)

The internal configuration file defining features:

```json
{
  "provider": "vendure",
  "features": {
    "wishlist": false
  }
}
```

- `wishlist: false` — Wishlist feature is disabled for Vendure (not implemented).

### Environment Variables

```typescript { .api }
// Required: Vendure Shop API GraphQL endpoint
NEXT_PUBLIC_VENDURE_SHOP_API_URL = "https://your-vendure-instance.com/shop-api"

// Optional: Local URL override (takes priority over NEXT_PUBLIC_VENDURE_SHOP_API_URL)
NEXT_PUBLIC_VENDURE_LOCAL_URL = "http://localhost:3000/shop-api"
```

Both variables are public (prefixed `NEXT_PUBLIC_`) and are embedded in the browser bundle.
