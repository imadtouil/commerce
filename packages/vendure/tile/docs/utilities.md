# Utilities

Internal utility functions for normalizing Vendure API responses and building tree structures from flat arrays. These are used internally by the provider but can also be imported directly.

## Import

```typescript
// Normalization utilities
import { normalizeCart, normalizeSearchResult } from '@vercel/commerce-vendure/utils/normalize'

// Tree utility
import { arrayToTree, type HasParent, type TreeNode, type RootNode } from '@vercel/commerce-vendure/utils/array-to-tree'
```

## Types

```typescript { .api }
// HasParent: base constraint for nodes used in arrayToTree
interface HasParent {
  id: string
  parent?: { id: string } | null
}

// TreeNode: a node with children and expansion state
type TreeNode<T extends HasParent> = T & {
  children: Array<TreeNode<T>>
  expanded: boolean
}

// RootNode: the root of a tree
interface RootNode<T extends HasParent> {
  id?: string
  children: Array<TreeNode<T>>
}

// Cart: normalized @vercel/commerce cart type (from normalize.ts)
interface Cart {
  id: string
  createdAt: string
  taxesIncluded: boolean
  lineItemsSubtotalPrice: number
  currency: { code: string }
  subtotalPrice: number
  totalPrice: number
  customerId?: string
  lineItems: Array<{
    id: string
    name: string
    quantity: number
    url: string
    variantId: string
    productId: string
    images: Array<{ url: string }>
    discounts: Array<{ value: number }>
    path: string
    variant: {
      id: string
      name: string
      sku: string
      price: number
      listPrice: number
      image: { url: string }
      requiresShipping: boolean
    }
  }>
}

// Product: normalized @vercel/commerce product type (from normalize.ts)
interface Product {
  id: string
  name: string
  description: string
  slug: string
  path: string
  images: Array<{ url: string }>
  variants: any[]
  price: { value: number; currencyCode: string }
  options: any[]
  sku: string
}
```

## Capabilities

### normalizeCart

Converts a Vendure `CartFragment` (Order) to the `@vercel/commerce` `Cart` type. Handles price conversion (divides integer amounts by 100), maps line items, and includes discount information.

```typescript { .api }
/**
 * Normalizes a Vendure Order (CartFragment) to @vercel/commerce Cart type.
 * - Prices are converted from smallest currency unit to decimal (divided by 100)
 * - taxesIncluded is always true (Vendure uses WithTax prices)
 * - requiresShipping is always true for variants
 * - Image URLs use '?preset=thumb' query parameter
 *
 * @param order - Vendure CartFragment (active order)
 * @returns Normalized Cart object
 */
function normalizeCart(order: CartFragment): Cart
```

**CartFragment fields used:**

```typescript
interface CartFragment {
  id: string | number
  code: string
  createdAt: string
  totalQuantity: number
  subTotal: number
  subTotalWithTax: number
  total: number
  totalWithTax: number
  currencyCode: string
  customer?: { id: string }
  lines: Array<{
    id: string
    quantity: number
    linePriceWithTax: number
    discountedLinePriceWithTax: number
    unitPriceWithTax: number
    discountedUnitPriceWithTax: number
    featuredAsset?: { id: string; preview: string }
    discounts: Array<{ description: string; amount: number }>
    productVariant: {
      id: string
      name: string
      sku: string
      price: number
      priceWithTax: number
      stockLevel: string
      product: { slug: string }
      productId: string
    }
  }>
}
```

### normalizeSearchResult

Converts a Vendure `SearchResultFragment` to the `@vercel/commerce` `Product` type. Used by `useSearch` (client-side) and `getAllProducts` (server-side).

```typescript { .api }
/**
 * Normalizes a Vendure SearchResult to @vercel/commerce Product type.
 * - Uses priceWithTax.min / 100 for price value (supports PriceRange)
 * - Image URL appends '?w=800&mode=crop' for optimization
 * - variants and options are empty arrays for search results
 *
 * @param item - Vendure SearchResultFragment
 * @returns Normalized Product object
 */
function normalizeSearchResult(item: SearchResultFragment): Product
```

**SearchResultFragment fields used:**

```typescript
interface SearchResultFragment {
  productId: string
  productName: string
  description: string
  slug: string
  sku: string
  currencyCode: string
  productAsset?: { id: string; preview: string }
  priceWithTax:
    | { __typename: 'SinglePrice'; value: number }
    | { __typename: 'PriceRange'; min: number; max: number }
}
```

### arrayToTree

Converts a flat array of nodes (each with optional parent reference) into a hierarchical tree structure. Used by `getSiteInfo` to build the category tree from Vendure collections.

```typescript { .api }
/**
 * Converts a flat array of nodes with parent references into a tree.
 * Nodes with no parent or whose parent is not in the array become top-level children.
 * Preserves 'expanded' state from currentState (for UI tree components).
 *
 * @param nodes - Flat array of items with id and optional parent.id
 * @param currentState - Optional existing tree to preserve expanded state from
 * @returns RootNode with children containing the tree structure
 */
function arrayToTree<T extends HasParent>(
  nodes: T[],
  currentState?: RootNode<T>
): RootNode<T>
```

**Usage:**

```typescript
import { arrayToTree, type HasParent } from '@vercel/commerce-vendure/utils/array-to-tree'

interface Category extends HasParent {
  id: string
  name: string
  parent?: { id: string } | null
}

const flatCategories: Category[] = [
  { id: '1', name: 'Electronics', parent: { id: '0' } },
  { id: '2', name: 'Clothing', parent: { id: '0' } },
  { id: '3', name: 'Laptops', parent: { id: '1' } },
]

const tree = arrayToTree(flatCategories)
// tree.children = [Electronics (with children: [Laptops]), Clothing]
```

**Notes:**

- Nodes whose `parent` is not found in the `mappedArr` are treated as top-level (root) nodes.
- The root `id` is set to the parent ID of the top-level nodes.
- The `expanded` property on each `TreeNode` defaults to `false` unless preserved from `currentState`.
