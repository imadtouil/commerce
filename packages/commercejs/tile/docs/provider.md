# Provider Setup

The `@vercel/commerce-commercejs` package is configured via the `CommerceProvider` React component and the `commercejsProvider` object.

## Capabilities

### CommerceProvider

Wraps the application with the Commerce.js provider context. Must be placed at the app root (e.g., `_app.tsx`).

```typescript { .api }
const CommerceProvider: React.FC<{
  locale?: string
  children: React.ReactNode
}>
```

**Usage:**

```typescript
import { CommerceProvider } from '@vercel/commerce-commercejs'

// pages/_app.tsx
export default function MyApp({ Component, pageProps }) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

### useCommerce

Returns the current commerce context. Can be used to access the locale and other context values.

```typescript { .api }
function useCommerce(): CommerceContextValue
```

**Usage:**

```typescript
import { useCommerce } from '@vercel/commerce-commercejs'

function MyComponent() {
  const commerce = useCommerce()
  // commerce.locale, etc.
}
```

### commercejsProvider

The raw provider configuration object, used internally by `CommerceProvider`. Can be passed to `getCommerceProvider()` for advanced customization.

```typescript { .api }
const commercejsProvider: {
  locale: string           // 'en-us'
  cartCookie: string       // 'commercejs_cart_id'
  customerCookie: string   // 'commercejs_customer_token'
  fetcher: Fetcher
  cart: {
    useCart: SWRHookHandler
    useAddItem: MutationHookHandler
    useUpdateItem: MutationHookHandler
    useRemoveItem: MutationHookHandler
  }
  checkout: {
    useCheckout: SWRHookHandler
    useSubmitCheckout: MutationHookHandler
  }
  customer: {
    useCustomer: SWRHookHandler
    card: {
      useCards: SWRHookHandler
      useAddItem: MutationHookHandler
    }
    address: {
      useAddresses: SWRHookHandler
      useAddItem: MutationHookHandler
    }
  }
  products: {
    useSearch: SWRHookHandler
  }
  auth: {
    useLogin: MutationHookHandler
    useLogout: MutationHookHandler
    useSignup: MutationHookHandler
  }
}

type CommercejsProvider = typeof commercejsProvider
```

**Usage:**

```typescript
import { commercejsProvider } from '@vercel/commerce-commercejs'
import { getCommerceProvider } from '@vercel/commerce'

// Create a custom provider (advanced)
const CustomCommerceProvider = getCommerceProvider(commercejsProvider)
```

### SDK Initialization

The Commerce.js SDK is initialized automatically via `NEXT_PUBLIC_COMMERCEJS_PUBLIC_KEY`. In development, the SDK is initialized in debug mode. If the key is missing in development, an error is thrown.

```typescript
// Internal initialization (src/lib/commercejs.ts)
// The SDK is a singleton accessible as `commerce` throughout the package.
// Requires: NEXT_PUBLIC_COMMERCEJS_PUBLIC_KEY env variable
import Commerce from '@chec/commerce.js'
const commerce = new Commerce(
  process.env.NEXT_PUBLIC_COMMERCEJS_PUBLIC_KEY,
  process.env.NODE_ENV === 'development'  // enables debug logging
)
```

### Fetcher

The default fetcher handles two modes:
- **SDK mode** (no `url`): Calls Commerce.js SDK methods using `query` (resource name) and `method`
- **Custom API route mode** (with `url`): Makes HTTP fetch to the given URL with JSON body

```typescript { .api }
type Fetcher = (options: {
  url?: string
  query?: string   // Commerce.js SDK resource name (e.g., 'cart', 'products', 'customer')
  method?: string  // SDK method name (e.g., 'retrieve', 'add', 'list')
  variables?: any
  body?: Record<string, any>
}) => Promise<any>
```

**Errors:**
- Throws `FetcherError` (status 400) if `query` is not a valid SDK resource
- Throws `FetcherError` (status 400) if `method` is not a valid method on the resource
