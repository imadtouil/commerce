# Provider Setup

## Overview

`@vercel/commerce-kibocommerce` requires wrapping the application with `CommerceProvider` and mounting API routes. The provider handles session state, cart cookies, and authentication context for all child components.

## Package Information

- **Entry point**: `@vercel/commerce-kibocommerce`
- **API entry point**: `@vercel/commerce-kibocommerce/api`
- **Next.js config**: `@vercel/commerce-kibocommerce/next.config`

## CommerceProvider

The root React context provider. Must wrap the entire application (typically in `_app.tsx`) for all commerce hooks to work.

```typescript { .api }
import { CommerceProvider } from '@vercel/commerce-kibocommerce'

function CommerceProvider(props: {
  children: React.ReactNode
  locale?: string
}): JSX.Element
```

**Props**:

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `React.ReactNode` | - | Application tree |
| `locale` | `string` | `'en-us'` | Locale string for commerce context |

**Example**:

```typescript
// pages/_app.tsx
import type { AppProps } from 'next/app'
import { CommerceProvider } from '@vercel/commerce-kibocommerce'

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

## useCommerce

Access the commerce context object within any component inside `CommerceProvider`.

```typescript { .api }
import { useCommerce } from '@vercel/commerce-kibocommerce'

function useCommerce(): CommerceContextValue
```

**Returns**: `CommerceContextValue` from `@vercel/commerce` — the provider configuration and context.

## kiboCommerceProvider

The raw provider configuration object. Used internally by `CommerceProvider`, but can be referenced directly when needed.

```typescript { .api }
import { kiboCommerceProvider } from '@vercel/commerce-kibocommerce'

const kiboCommerceProvider: {
  locale: 'en-us'
  cartCookie: 'kibo_cart'
  fetcher: Fetcher
  cart: {
    useCart: SWRHook
    useAddItem: MutationHook
    useUpdateItem: MutationHook
    useRemoveItem: MutationHook
  }
  wishlist: {
    useWishlist: SWRHook
    useAddItem: MutationHook
    useRemoveItem: MutationHook
  }
  customer: { useCustomer: SWRHook }
  products: { useSearch: SWRHook }
  auth: {
    useLogin: MutationHook
    useLogout: MutationHook
    useSignup: MutationHook
  }
}

type KibocommerceProvider = typeof kiboCommerceProvider
```

## API Routes Setup

Mount all commerce API endpoints in a single catch-all Next.js API route.

```typescript { .api }
import kiboCommerceAPI from '@vercel/commerce-kibocommerce/api/endpoints'
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'
import type { KiboCommerceAPI } from '@vercel/commerce-kibocommerce/api'

function kiboCommerceAPI(commerce: KiboCommerceAPI): NextApiHandler
```

**Example** (`pages/api/commerce/[...path].ts`):

```typescript
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'
import kiboCommerceAPI from '@vercel/commerce-kibocommerce/api/endpoints'

const commerce = getCommerceApi()
export default kiboCommerceAPI(commerce)
```

This mounts the following API endpoint paths:
- `GET/POST/PUT/DELETE /api/commerce/cart` — Cart operations
- `POST /api/commerce/login` — Customer login
- `GET /api/commerce/logout` — Customer logout
- `POST /api/commerce/signup` — Customer registration
- `GET /api/commerce/customer` — Current customer data
- `GET/POST/DELETE /api/commerce/wishlist` — Wishlist operations
- `GET /api/commerce/catalog/products` — Product search

## Next.js Config Integration

```typescript { .api }
// next.config module (CommonJS)
import commerce from '@vercel/commerce-kibocommerce/next.config'

const nextConfig = {
  commerce: {
    provider: 'kibocommerce',
    features: {
      wishlist: boolean
      cart: boolean
      search: boolean
      customerAuth: boolean
    }
  },
  serverRuntimeConfig: {
    kiboAuthTicket: null
  },
  images: {
    domains: string[]
  }
}
```

**Usage** (`next.config.js`):

```javascript
const { withCommerceConfig } = require('@vercel/commerce/config')
const commerce = require('@vercel/commerce-kibocommerce/next.config')

module.exports = withCommerceConfig({
  commerce,
  // ... other next config
})
```

## Environment Variables

All variables must be set before starting the server:

| Variable | Description | Example |
|----------|-------------|---------|
| `KIBO_API_URL` | Kibo GraphQL endpoint | `https://t1234-s1234.sandbox.mozu.com/graphql` |
| `KIBO_CART_COOKIE` | Cookie name for cart session | `kibo_cart` |
| `KIBO_CUSTOMER_COOKIE` | Cookie name for customer session | `kibo_customer` |
| `KIBO_CLIENT_ID` | OAuth application client ID | `KIBO.APP.1.0.0.Release` |
| `KIBO_SHARED_SECRET` | OAuth application shared secret | `your_shared_secret` |
| `KIBO_AUTH_URL` | Kibo auth service base URL | `https://home.mozu.com` |
| `KIBO_API_TOKEN` | Optional static API token | (optional) |
| `KIBO_API_HOST` | Kibo REST API host | (optional) |

The `KIBO_CLIENT_ID` and `KIBO_SHARED_SECRET` are found in the [Kibo eCommerce Dev Center](https://mozu.com/login). The package uses OAuth client credentials grant (`grant_type: client_credentials`) to obtain and cache access tokens server-side.
