# Types Reference

Complete TypeScript type definitions used throughout `@vercel/commerce-kibocommerce`. These types are normalized from Kibo-specific GraphQL types to the standard `@vercel/commerce` type interfaces.

## Cart Types {#cart}

```typescript { .api }
interface Cart {
  id: string
  customerId?: string
  email?: string
  createdAt?: string
  currency: {
    code: string    // Always 'USD' — hardcoded in normalizeCart; not derived from Kibo response
  }
  taxesIncluded: boolean
  lineItems: LineItem[]
  lineItemsSubtotalPrice?: number
  subtotalPrice?: number
  totalPrice?: number
  discounts?: Discount[]
}

interface LineItem {
  id: string
  variantId: string
  productId: string
  name: string
  quantity: number
  variant: {
    id: string
    sku?: string
    name: string
    image?: {
      url: string
    }
    requiresShipping?: boolean
    price: number
    listPrice: number
  }
  options?: Array<{
    [key: string]: any
  }>
  path: string        // productCode
  discounts?: Discount[]
}

interface Discount {
  value: number
}

// Input for adding items to cart
interface CartItemInput {
  productId: string
  variantId: string
  quantity?: number   // Must be integer >= 1
}

// Input for updating cart items
interface UpdateItemInput {
  id?: string         // Line item ID (required if no ctx.item)
  productId?: string  // Required if no ctx.item
  variantId?: string  // Required if no ctx.item
  quantity: number    // If < 1, item is removed automatically
}
```

## Customer Types {#customer}

```typescript { .api }
interface Customer {
  id: string
  firstName: string
  lastName: string
  email: string
  acceptsMarketing: boolean
}

// Input for signup
interface SignupInput {
  firstName: string
  lastName: string
  email: string
  password: string
}

// Input for login
interface LoginInput {
  email: string
  password: string
}
```

## Product Types {#product}

```typescript { .api }
interface Product {
  id: string           // productCode
  name: string
  vendor: string       // Always empty string (not supported by Kibo)
  path: string         // '/<productCode>'
  slug: string         // productCode
  price: {
    value: number
    currencyCode: string
  }
  description: string
  descriptionHtml: string
  images: ProductImage[]
  variants: ProductVariant[]
  options: ProductOption[]
}

interface ProductImage {
  url: string         // Prefixed with 'http:'
  altText?: string
}

interface ProductVariant {
  id: string          // variationProductCode
  options: ProductVariantOption[]
}

interface ProductVariantOption {
  __typename: 'MultipleChoiceOption'
  id: string          // attributeFQN
  displayName: string
  values: Array<{ label: string }>
}

interface ProductOption {
  id: string          // attributeFQN
  displayName: string
  values: Array<{ label: string }>
}

// useSearch result data
interface SearchProductsData {
  products: Product[]
  found: boolean
}

// usePrice input/output
interface UsePriceInput {
  amount: number
  baseAmount?: number
  currencyCode: string
}

interface UsePriceOutput {
  price: string           // Formatted (e.g., '$29.99')
  basePrice?: string      // Formatted base price
  discount?: string       // Discount percentage string
}
```

## Category Types {#category}

```typescript { .api }
interface Category {
  id: string           // categoryCode
  name: string
  slug: string
  path: string         // '/<slug>'
}
```

## Page Types {#page}

```typescript { .api }
interface Page {
  id: string
  name: string
  url: string
  body: string
  is_visible: boolean
  sort_order: number
}
```

## Wishlist Types {#wishlist}

```typescript { .api }
interface Wishlist {
  id?: string
  items?: WishlistItem[]
}

interface WishlistItem {
  // When includeProducts=false (default): id = productCode (string)
  // When includeProducts=true: id = Kibo wishlist item ID
  // Use id as the argument to useRemoveItem (server matches by productCode)
  id: string | number
  productId: string
  variantId: string
  product?: WishlistProduct   // Only present when includeProducts=true
}

interface WishlistProduct {
  variant_id: string
  id: string
  product_id: string
  name: string
  quantity: number
  images: Array<{ url: string; alt?: string }>
  price: {
    value: number
    retailPrice: number
    currencyCode: string
  }
  variants: Array<{
    id: string
    sku?: string
    name: string
    image?: { url: string }
  }>
  options: any[]
  path: string         // '/<productCode>'
  description?: string
}

// Input for adding items to wishlist
interface WishlistItemInput {
  productId: string
  variantId: string
}
```

## SWR Types

```typescript { .api }
// SWROptions can be passed to any SWR-based hook
interface SWROptions {
  revalidateOnFocus?: boolean
  revalidateOnMount?: boolean
  revalidateOnReconnect?: boolean
  refreshInterval?: number
  dedupingInterval?: number
  shouldRetryOnError?: boolean
  // ... any other SWR configuration options
}
```

## Endpoint Types

```typescript { .api }
// Cart API endpoint types (src/api/endpoints/cart)
type CartAPI = GetAPISchema<KiboCommerceAPI, any>
type CartEndpoint = CartAPI['endpoint']

// Login API endpoint types
type LoginAPI = GetAPISchema<KiboCommerceAPI, LoginSchema>
type LoginEndpoint = LoginAPI['endpoint']

// Customer API endpoint types
type CustomerAPI = GetAPISchema<KiboCommerceAPI, CustomerSchema>
type CustomerEndpoint = CustomerAPI['endpoint']

// Wishlist API endpoint types
type WishlistAPI = GetAPISchema<KiboCommerceAPI, any>
type WishlistEndpoint = WishlistAPI['endpoint']

// Products API endpoint types
type ProductsAPI = GetAPISchema<KiboCommerceAPI, any>
type ProductsEndpoint = ProductsAPI['endpoint']
```

## Provider Types

```typescript { .api }
import { kiboCommerceProvider } from '@vercel/commerce-kibocommerce'
import type { KibocommerceProvider } from '@vercel/commerce-kibocommerce'

// Type of the client-side provider object
type KibocommerceProvider = typeof kiboCommerceProvider

// Server-side provider type
import type { KiboCommerceProvider, KiboCommerceAPI } from '@vercel/commerce-kibocommerce/api'

type KiboCommerceProvider = {
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

type KiboCommerceAPI<P extends KiboCommerceProvider = KiboCommerceProvider> = CommerceAPI<P>
```

## Fetcher Types

```typescript { .api }
// Client-side fetcher (from @vercel/commerce/utils/types)
type Fetcher = (options: {
  url?: string
  method?: string
  variables?: Record<string, any>
  body?: Record<string, any>
}) => Promise<any>

// GraphQL fetcher (server-side)
type GraphQLFetcher = (
  query: string,
  options?: { variables?: Record<string, any>; preview?: boolean },
  headers?: HeadersInit
) => Promise<{ data: any; res: Response }>
```

## Error Types

```typescript { .api }
// From @vercel/commerce/utils/errors
class CommerceError extends Error {
  constructor(options: { message?: string; errors?: any[] })
}

class FetcherError extends Error {
  status: number
  constructor(options: { message?: string; errors?: any[]; status: number })
}

class ValidationError extends Error {
  constructor(options: { message?: string; errors?: any[] })
}
```

`CommerceError` is thrown by:
- `useAddItem` (cart) — invalid quantity
- `useLogin` — missing email or password
- `useSignup` — missing required fields
- `useAddItem` (wishlist) — customer not signed in
- `useRemoveItem` (wishlist) — customer not signed in

`ValidationError` is thrown by:
- `useUpdateItem` — non-integer quantity or missing required IDs
- `useRemoveItem` (cart) — missing item ID

`FetcherError` is thrown by:
- Client-side fetcher — non-OK HTTP responses or API errors
- Server-side GraphQL fetcher — GraphQL errors or HTTP failures
