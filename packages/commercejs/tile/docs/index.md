# @vercel/commerce-commercejs

Commerce.js provider for [Next.js Commerce](https://github.com/vercel/commerce), implementing cart management, product search, customer authentication, and checkout via the [Commerce.js](https://commercejs.com/) headless e-commerce platform.

This package implements the `@vercel/commerce` provider interface using the `@chec/commerce.js` SDK, providing React hooks for client-side operations and server-side API operations for Next.js data fetching.

## Package Information

- **Package Name**: @vercel/commerce-commercejs
- **Package Type**: npm (scoped)
- **Language**: TypeScript
- **Installation**: `npm install @vercel/commerce-commercejs`
- **Peer Dependencies**: `next ^12`, `react ^18`, `react-dom ^18`

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_COMMERCEJS_PUBLIC_KEY` | **Yes** | Commerce.js public API key (found in Commerce.js dashboard → Developer → API keys) |
| `NEXT_PUBLIC_COMMERCEJS_DEPLOYMENT_URL` | No | Custom base URL for login callbacks (overrides auto-detection) |
| `NEXT_PUBLIC_VERCEL_URL` | No | Vercel deployment URL (auto-set on Vercel platform) |

## Core Imports

```typescript
// Main provider and hooks
import { CommerceProvider, useCommerce, commercejsProvider } from '@vercel/commerce-commercejs'
import type { CommercejsProvider } from '@vercel/commerce-commercejs'

// Cart hooks
import { useCart, useAddItem, useUpdateItem, useRemoveItem } from '@vercel/commerce-commercejs/cart'

// Auth hooks
import { useLogin, useLogout, useSignup } from '@vercel/commerce-commercejs/auth'

// Checkout hooks
import { useCheckout, useSubmitCheckout } from '@vercel/commerce-commercejs/checkout'

// Customer hook
import { useCustomer } from '@vercel/commerce-commercejs/customer'

// Customer card hooks (stubs)
import { useCards, useAddItem as useAddCard } from '@vercel/commerce-commercejs/customer/card'

// Customer address hooks (stubs)
import { useAddresses, useAddItem as useAddAddress } from '@vercel/commerce-commercejs/customer/address'

// Product hooks
import { useSearch, usePrice } from '@vercel/commerce-commercejs/product'

// Server-side API
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

// Next.js config
import commercejsConfig from '@vercel/commerce-commercejs/next.config'

// Types
import type {
  CommercejsCart,
  CommercejsLineItem,
  CommercejsProduct,
  CommercejsVariant,
  CommercejsCategory,
  CommercejsCheckoutCapture,
} from '@vercel/commerce-commercejs/types'

// Constants
import { CART_COOKIE, CUSTOMER_COOKIE, API_URL, LOCALE } from '@vercel/commerce-commercejs/constants'
```

## Basic Usage

```typescript
// _app.tsx - Wrap your app with CommerceProvider
import { CommerceProvider } from '@vercel/commerce-commercejs'

export default function MyApp({ Component, pageProps }) {
  return (
    <CommerceProvider locale="en-us">
      <Component {...pageProps} />
    </CommerceProvider>
  )
}
```

```typescript
// Component using cart
import { useCart, useAddItem } from '@vercel/commerce-commercejs/cart'

function CartButton({ productId, variantId }) {
  const { data: cart, isEmpty } = useCart()
  const addItem = useAddItem()

  const handleAdd = async () => {
    await addItem({ productId, variantId, quantity: 1 })
  }

  return (
    <button onClick={handleAdd}>
      Add to Cart ({cart?.lineItems?.length ?? 0} items)
    </button>
  )
}
```

```typescript
// pages/api/commerce/[...commerce].ts - Server-side API routes
import commercejsAPI from '@vercel/commerce-commercejs/api/endpoints'
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()
export default commercejsAPI(commerce)
```

## Architecture

- **CommerceProvider**: React context provider configured with the Commerce.js provider object
- **React Hooks**: Client-side SWR hooks for data fetching and mutation hooks for cart/auth operations
- **Server Operations**: Server-side functions for SSG/SSR product and category data
- **API Endpoints**: Next.js API route handlers for login callback and checkout submission
- **SDK Integration**: Uses `@chec/commerce.js` SDK initialized with `NEXT_PUBLIC_COMMERCEJS_PUBLIC_KEY`

## Features

| Feature | Status | Notes |
|---------|--------|-------|
| Cart | ✅ Enabled | Full CRUD via Commerce.js SDK |
| Product Search | ✅ Enabled | Filter by search, category, sort |
| Customer Auth | ✅ Enabled | Magic-link login via email |
| Custom Checkout | ✅ Enabled | Uses test gateway |
| Wishlist | ❌ Disabled | Intentionally stubbed |

## Capabilities

### Provider Setup

Configure the Commerce.js provider at the application root using `CommerceProvider` and access the commerce context with `useCommerce`.

```typescript { .api }
const CommerceProvider: React.FC<{ locale?: string; children: React.ReactNode }>

function useCommerce(): CommerceContextValue

const commercejsProvider: CommercejsProvider
type CommercejsProvider = {
  locale: string
  cartCookie: string
  customerCookie: string
  fetcher: Fetcher
  cart: { useCart: Handler; useAddItem: Handler; useUpdateItem: Handler; useRemoveItem: Handler }
  checkout: { useCheckout: Handler; useSubmitCheckout: Handler }
  customer: {
    useCustomer: Handler
    card: { useCards: Handler; useAddItem: Handler }
    address: { useAddresses: Handler; useAddItem: Handler }
  }
  products: { useSearch: Handler }
  auth: { useLogin: Handler; useLogout: Handler; useSignup: Handler }
}
```

[Provider Setup](./provider.md)

### Cart Management

Full cart operations including fetching cart state, adding products, updating quantities (debounced), and removing items.

```typescript { .api }
function useCart(input?: { swrOptions?: SWROptions }): CartResponse & { isEmpty: boolean }

function useAddItem(): (input: AddItemInput) => Promise<Cart>

function useUpdateItem(ctx?: { item?: LineItem; wait?: number }): (input: UpdateItemInput) => Promise<Cart>

function useRemoveItem(): (input: { id: string }) => Promise<Cart>

interface AddItemInput {
  productId: string
  variantId?: string
  quantity?: number  // default: 1
}

interface UpdateItemInput {
  id?: string
  quantity?: number
  productId?: string
  variantId?: string
}

interface Cart {
  id: string
  createdAt: string
  currency: { code: string }
  taxesIncluded: boolean
  lineItems: LineItem[]
  lineItemsSubtotalPrice: number
  subtotalPrice: number
  totalPrice: number
}

interface LineItem {
  id: string
  variantId: string
  productId: string
  name: string
  quantity: number
  discounts: any[]
  path: string
  options: Array<{ name: string; value: string }>
  variant: {
    id: string
    sku: string
    name: string
    requiresShipping: boolean
    price: number
    listPrice: number
    image: { url: string }
  }
}
```

[Cart Management](./cart.md)

### Authentication

Magic-link email-based customer authentication. Login sends an email with a link; logout clears the local session cookie.

```typescript { .api }
function useLogin(): (input: { email: string }) => Promise<null>

function useLogout(): () => Promise<null>

function useSignup(): () => void  // stub, not implemented
```

[Authentication](./auth.md)

### Checkout

Custom checkout flow using Commerce.js's checkout token system. Reads card and address fields from the checkout context.

```typescript { .api }
function useCheckout(): {
  data: { hasPayment: boolean; hasShipping: boolean }
  submit: () => Promise<any>
}

function useSubmitCheckout(): (input?: any) => Promise<any>
```

[Checkout](./checkout.md)

### Customer Profile

Retrieve authenticated customer data. Card and address management hooks are present but stubbed (return empty state).

```typescript { .api }
function useCustomer(input?: { swrOptions?: SWROptions }): SWRResponse<Customer | null>

interface Customer {
  id: string
  firstName: string
  lastName: string
  email: string
  phone: string
}
```

[Customer Profile](./customer.md)

### Product Catalog

Client-side product search hook and server-side operations for listing all products, fetching by slug, and getting site categories.

```typescript { .api }
function useSearch(input?: SearchInput): SWRResponse<{ products: Product[]; found: boolean }>

interface SearchInput {
  search?: string
  categoryId?: string | number
  brandId?: string | number
  sort?: 'trending-desc' | 'latest-desc' | 'price-asc' | 'price-desc'
  swrOptions?: SWROptions
}

interface Product {
  id: string
  name: string
  description: string
  descriptionHtml: string
  slug: string
  path: string
  images: Array<{ url: string; alt: string }>
  price: { value: number; currencyCode: string }
  variants: ProductVariant[]
  options: ProductOption[]
}
```

[Product Catalog](./product.md)

### Server-Side API

Server-side data operations for Next.js SSG/SSR pages and API route handlers for login and checkout endpoints.

```typescript { .api }
function getCommerceApi(customProvider?: Provider): CommercejsAPI

interface CommercejsAPI {
  getAllProducts(opts?: { config?: Partial<CommercejsConfig> }): Promise<{ products: Product[] }>
  getAllProductPaths(opts?: { config?: Partial<CommercejsConfig> }): Promise<{ products: Array<{ path: string }> }>
  getProduct(opts?: { variables?: { slug: string }; config?: Partial<CommercejsConfig> }): Promise<{ product: Product }>
  getSiteInfo(opts?: { config?: Partial<CommercejsConfig> }): Promise<{ categories: Category[]; brands: [] }>
  getAllPages(opts?: { config?: Partial<CommercejsConfig>; preview?: boolean }): Promise<{ pages: [] }>
  getPage(): Promise<{}>
}

function commercejsAPI(commerce: CommercejsAPI): NextApiHandler
```

[Server-Side API](./api.md)

## Types

```typescript { .api }
// Re-exported from @chec/commerce.js
export type { CommercejsCart } from '@chec/commerce.js/types/cart'
export type { CommercejsLineItem } from '@chec/commerce.js/types/line-item'
export type { CommercejsCheckoutCapture } from '@chec/commerce.js/types/checkout-capture'
export type { CommercejsProduct } from '@chec/commerce.js/types/product'
export type { CommercejsVariant } from '@chec/commerce.js/types/variant'
export type { CommercejsCategory } from '@chec/commerce.js/types/category'
```

## Constants

```typescript { .api }
const CART_COOKIE = 'commercejs_cart_id'        // Cookie name for cart ID
const CUSTOMER_COOKIE = 'commercejs_customer_token' // Cookie name for customer JWT
const API_URL = 'https://api.chec.io/v1'        // Commerce.js API base URL
const LOCALE = 'en-us'                           // Default locale
```

## Next.js Configuration

```typescript { .api }
// Import via @vercel/commerce-commercejs/next.config
const commercejsConfig: {
  commerce: { provider: string; features: Record<string, boolean> }
  images: { domains: string[] }  // includes 'cdn.chec.io'
  rewrites(): Array<{ source: string; destination: string }>
}
```

Usage in `next.config.js`:
```javascript
const commercejsConfig = require('@vercel/commerce-commercejs/next.config')

module.exports = {
  ...commercejsConfig,
  // Add or override config as needed
}
```
