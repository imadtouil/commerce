# @vercel/commerce-kibocommerce

Kibo Commerce provider for the Next.js Commerce framework. This package implements the `@vercel/commerce` provider interface using Kibo's GraphQL API, enabling full e-commerce functionality: shopping cart, wishlist, customer authentication, and product catalog browsing.

## Package Information

- **Package Name**: `@vercel/commerce-kibocommerce`
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install @vercel/commerce-kibocommerce`
- **Peer Dependencies**: `next ^12`, `react ^18`, `react-dom ^18`

## Core Imports

```typescript
// Client-side: React provider and hook
import { CommerceProvider, useCommerce } from '@vercel/commerce-kibocommerce'

// Server-side: API factory
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'

// Next.js config integration
import commerce from '@vercel/commerce-kibocommerce/next.config'
```

## Basic Usage

```typescript
// _app.tsx - wrap your app with CommerceProvider
import { CommerceProvider } from '@vercel/commerce-kibocommerce'

export default function MyApp({ Component, pageProps }) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

```typescript
// pages/api/commerce/[...path].ts - mount API routes
import { getCommerceApi } from '@vercel/commerce-kibocommerce/api'
import kiboCommerceAPI from '@vercel/commerce-kibocommerce/api/endpoints'

const commerce = getCommerceApi()
export default kiboCommerceAPI(commerce)
```

## Architecture

This package is structured as two layers:

- **Client-side hooks**: React hooks using SWR for data fetching and mutations, imported from the package root or specific subpaths. Used in Next.js pages and components.
- **Server-side API**: Node.js-only API handler factory and operation functions for Next.js API routes. Handles authentication, GraphQL fetching, and data normalization.

The provider requires Kibo credentials configured via environment variables. Authentication uses Kibo's OAuth client credentials flow, caching auth tickets in memory for reuse.

## Environment Variables

```bash
KIBO_API_URL=https://t1234-s1234.sandbox.mozu.com/graphql
KIBO_CART_COOKIE=kibo_cart
KIBO_CUSTOMER_COOKIE=kibo_customer
KIBO_CLIENT_ID=KIBO.APP.1.0.0.Release
KIBO_SHARED_SECRET=your_shared_secret
KIBO_AUTH_URL=https://home.mozu.com
```

## Capabilities

### Provider Setup

Root React provider wrapping the application. Required for all client-side hooks to function.

```typescript { .api }
function CommerceProvider(props: {
  children: React.ReactNode
  locale?: string
}): JSX.Element

function useCommerce(): CommerceContextValue
```

[Provider Setup](./setup.md)

### Cart Management

Client-side hooks for reading and mutating the shopping cart. Cart is identified via the `kibo_cart` cookie.

```typescript { .api }
function useCart(input?: { swrOptions?: SWROptions }): {
  data: Cart | null
  isEmpty: boolean
  isLoading: boolean
  error?: Error
}

function useAddItem(): (item: CartItemInput) => Promise<Cart>

function useUpdateItem(ctx?: {
  item?: LineItem
  wait?: number
}): (input: UpdateItemInput) => Promise<Cart>

function useRemoveItem(ctx?: {
  item?: LineItem
}): (input?: { id?: string }) => Promise<Cart | null>
```

[Cart Management](./hooks-cart.md)

### Authentication

Client-side hooks for customer login, logout, and account creation.

```typescript { .api }
function useLogin(): (input: { email: string; password: string }) => Promise<void>

function useLogout(): () => Promise<void>

function useSignup(): (input: {
  firstName: string
  lastName: string
  email: string
  password: string
}) => Promise<void>
```

[Authentication](./hooks-auth.md)

### Product Search

Client-side hook for querying the product catalog with search, category, brand, and sort filters.

```typescript { .api }
function useSearch(input?: {
  search?: string
  categoryId?: string | number
  brandId?: number
  sort?: string
  swrOptions?: SWROptions
}): {
  data: SearchResult | null
  isLoading: boolean
  error?: Error
}

function usePrice(input: {
  amount: number
  baseAmount?: number
  currencyCode: string
}): { price: string; basePrice?: string; discount?: string }
```

[Product Search](./hooks-product.md)

### Wishlist Management

Client-side hooks for customer wishlist operations. Requires an authenticated customer.

```typescript { .api }
function useWishlist(input?: {
  includeProducts?: boolean
  swrOptions?: SWROptions
}): {
  data: Wishlist | null
  isEmpty: boolean
  isLoading: boolean
  error?: Error
}

function useAddItem(): (item: WishlistItemInput) => Promise<Wishlist>

function useRemoveItem(ctx?: { wishlist?: Wishlist }): (input: { id: string | number }) => Promise<Wishlist>
```

[Wishlist Management](./hooks-wishlist.md)

### Customer

Client-side hook for reading the current authenticated customer. Also includes stub hooks for customer address and card management (not implemented).

```typescript { .api }
function useCustomer(input?: { swrOptions?: SWROptions }): {
  data: Customer | null
  isLoading: boolean
  error?: Error
}

// Stubs — not implemented, return empty objects
import useAddItem from '@vercel/commerce-kibocommerce/customer/address/use-add-item'
import useAddItem from '@vercel/commerce-kibocommerce/customer/card/use-add-item'
```

[Customer](./hooks-customer.md)

### Server-Side API

Server-side API factory and operations for use in Next.js API routes. Provides product, page, category, and wishlist data fetching with Kibo GraphQL integration.

```typescript { .api }
function getCommerceApi<P extends KiboCommerceProvider>(
  customProvider?: P
): KiboCommerceAPI<P>

interface KiboCommerceConfig extends CommerceAPIConfig {
  apiHost?: string
  clientId?: string
  sharedSecret?: string
  authUrl?: string
  customerCookieMaxAgeInDays: number
  currencyCode: string
  documentListName: string
  defaultWishlistName: string
}
```

[Server-Side API](./server-api.md)

### Checkout (Stub)

The checkout module exports `useCheckout` but it is a stub/placeholder with no implementation. The fetcher and hook return empty objects. Do not rely on this for production checkout flows.

```typescript { .api }
import useCheckout from '@vercel/commerce-kibocommerce/checkout/use-checkout'

// Returns empty object; not implemented
function useCheckout(input?: any): {}
```

### Types

All TypeScript types, interfaces, and data shapes used across the package.

[Types Reference](./types.md)
