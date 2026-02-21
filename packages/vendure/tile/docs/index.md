# @vercel/commerce-vendure

`@vercel/commerce-vendure` is the Vendure provider adapter for the `@vercel/commerce` framework. It connects Vendure's headless GraphQL e-commerce platform to Next.js storefronts by implementing the standard commerce provider interface. The package provides React hooks for client-side cart management, authentication, customer profiles, and product search, as well as a server-side API layer for product listings, site info, and authentication operations.

## Package Information

- **Package Name**: `@vercel/commerce-vendure`
- **Package Type**: npm
- **Language**: TypeScript
- **Installation**: `npm install @vercel/commerce-vendure`
- **Peer Dependencies**: `next@^12`, `react@^18`, `react-dom@^18`
- **Module type**: ES module (`"type": "module"`)

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_VENDURE_SHOP_API_URL` | Yes | URL of the Vendure Shop API GraphQL endpoint |
| `NEXT_PUBLIC_VENDURE_LOCAL_URL` | No | Local override URL for the Shop API (takes priority over `NEXT_PUBLIC_VENDURE_SHOP_API_URL`) |

## Core Imports

```typescript
// Main provider and hook
import { CommerceProvider, useCommerce } from '@vercel/commerce-vendure'

// Cart hooks
import { useCart, useAddItem, useUpdateItem, useRemoveItem } from '@vercel/commerce-vendure/cart'

// Auth hooks
import { useLogin, useLogout, useSignup } from '@vercel/commerce-vendure/auth'

// Customer hooks
import { useCustomer } from '@vercel/commerce-vendure/customer'

// Product hooks
import { useSearch, usePrice } from '@vercel/commerce-vendure/product'

// Provider config (for custom setup)
import { vendureProvider, type VendureProvider } from '@vercel/commerce-vendure'

// Next.js config
const { commerce } = require('@vercel/commerce-vendure/next.config')

// Server-side API
import { getCommerceApi } from '@vercel/commerce-vendure/api'
```

## Basic Usage

```typescript
// _app.tsx — wrap your app with CommerceProvider
import { CommerceProvider } from '@vercel/commerce-vendure'

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

```typescript
// next.config.js
const { commerce } = require('@vercel/commerce-vendure/next.config')

module.exports = {
  ...commerce,
  // Configures image domains: localhost, demo.vendure.io, readonlydemo.vendure.io
}
```

## Capabilities

### Provider Setup & Configuration

Setup the Vendure commerce provider, configure the Next.js app, and access the commerce context.

```typescript { .api }
// Root provider component
const CommerceProvider: React.FC<{ locale?: string; children: React.ReactNode }>

// Access commerce context in any component
function useCommerce(): CommerceContextValue

// Raw provider object for custom setups
const vendureProvider: VendureProvider
type VendureProvider = typeof vendureProvider
```

[Provider Setup](./provider-setup.md)

### Cart Management

Client-side React hooks for managing the shopping cart: reading cart state, adding items, updating quantities, and removing items.

```typescript { .api }
function useCart(input?: { swrOptions?: SWROptions }): Cart & { isEmpty: boolean }

function useAddItem(): (input: AddItemInput) => Promise<Cart | null>

function useUpdateItem(ctx?: { item?: LineItem; wait?: number }): (input: UpdateItemInput) => Promise<Cart | null>

function useRemoveItem(): (input: { id: string }) => Promise<Cart | null>

interface AddItemInput {
  variantId: string
  quantity?: number  // defaults to 1; must be positive integer
}

interface UpdateItemInput {
  quantity: number
  productId?: string
  variantId?: string
}
```

[Cart Management](./cart.md)

### Authentication

Client-side React hooks for customer authentication: login, logout, and signup (customer registration).

```typescript { .api }
function useLogin(): (input: LoginInput) => Promise<null>

function useLogout(): () => Promise<null>

function useSignup(): (input: SignupInput) => Promise<null>

interface LoginInput {
  email: string
  password: string
}

interface SignupInput {
  email: string
  firstName: string
  lastName: string
  password: string
}
```

[Authentication](./auth.md)

### Customer Profile

Client-side React hook for retrieving the authenticated customer's profile.

```typescript { .api }
function useCustomer(input?: { swrOptions?: SWROptions }): Customer | null

interface Customer {
  firstName: string
  lastName: string
  email: string
}
```

[Customer Profile](./customer.md)

### Product Search

Client-side React hook for searching and listing products from Vendure.

```typescript { .api }
function useSearch(input?: SearchInput): SearchData

interface SearchInput {
  search?: string
  categoryId?: string
  brandId?: string
  sort?: string
  swrOptions?: SWROptions
}

interface SearchData {
  products: Product[]
  found: boolean
}
```

[Product Search](./product.md)

### Server-Side API

Server-side operations for use in Next.js `getStaticProps`/`getServerSideProps`: fetch products, site info, authentication, and product paths.

```typescript { .api }
function getCommerceApi<P extends Provider>(customProvider?: P): CommerceAPI<P>

// Available operations on the returned API instance:
interface CommerceAPI {
  getAllProducts(opts?: { variables?: { first?: number }; config?: Partial<VendureConfig> }): Promise<{ products: Product[] }>
  getProduct(opts: { variables: { slug: string }; config?: Partial<VendureConfig> }): Promise<{ product: Product } | {}>
  getAllProductPaths(opts?: { variables?: { first?: number }; config?: VendureConfig }): Promise<{ products: Array<{ path: string }> }>
  getSiteInfo(opts?: { config?: Partial<VendureConfig> }): Promise<{ categories: Category[]; brands: any[] }>
  getAllPages(opts?: { config?: Partial<VendureConfig> }): Promise<{ pages: [] }>
  getPage(opts: { variables: { id: number }; config?: Partial<VendureConfig> }): Promise<{}>
  login(opts: { variables: { username: string; password: string }; res: Response; config?: Partial<VendureConfig> }): Promise<{ result: string }>
  getCustomerWishlist(opts: { variables: any; config?: Partial<VendureConfig> }): Promise<{ wishlist: {} }>
}
```

[Server-Side API](./api-server.md)

### Next.js API Route Handler

Register Vendure API routes in Next.js API endpoints.

```typescript { .api }
function vendureAPI(commerce: VendureAPI): NextApiHandler
```

[Server-Side API](./api-server.md)

### Utilities

Internal data normalization and tree utilities.

```typescript { .api }
function normalizeCart(order: CartFragment): Cart
function normalizeSearchResult(item: SearchResultFragment): Product
function arrayToTree<T extends HasParent>(nodes: T[], currentState?: RootNode<T>): RootNode<T>
```

[Utilities](./utilities.md)

## Unimplemented Features (Stubs)

The following are present in the package but **not implemented** for Vendure:

| Feature | Module | Notes |
|---------|--------|-------|
| Wishlist | `@vercel/commerce-vendure/wishlist/use-wishlist` | Always returns `{ data: null }`. Vendure does not have built-in wishlist. `wishlist: false` in commerce.config.json. |
| Wishlist add item | `@vercel/commerce-vendure/wishlist/use-add-item` | Empty stub (`emptyHook`). Not implemented. |
| Wishlist remove item | `@vercel/commerce-vendure/wishlist/use-remove-item` | Empty stub (`emptyHook`). Not implemented. |
| Checkout | `@vercel/commerce-vendure/checkout/use-checkout` | Empty stub hook. |
| Checkout API endpoint | `@vercel/commerce-vendure/api/endpoints` | Registered but returns stub HTML page. |
| Pages | `getCommerceApi().getAllPages()` | Always returns empty array. |
| Customer address management | `@vercel/commerce-vendure/customer/address/use-add-item` | Empty stub. |
| Customer card management | `@vercel/commerce-vendure/customer/card/use-add-item` | Empty stub. |

## Common Types

```typescript { .api }
// SWR options shared across hooks
interface SWROptions {
  revalidateOnFocus?: boolean
  revalidateIfStale?: boolean
  revalidateOnReconnect?: boolean
  dedupingInterval?: number
  [key: string]: any
}
```
