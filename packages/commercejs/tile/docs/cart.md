# Cart Management

Full cart lifecycle management via Commerce.js SDK. All cart hooks integrate with SWR for automatic caching and revalidation.

## Import

```typescript
import { useCart, useAddItem, useUpdateItem, useRemoveItem } from '@vercel/commerce-commercejs/cart'
```

## Types

```typescript { .api }
// Exported from @vercel/commerce-commercejs/cart/use-update-item
type UpdateItemActionInput<T = any> = T extends LineItem
  ? Partial<{ id?: string; quantity?: number; productId?: string; variantId?: string }>
  : { id?: string; quantity?: number; productId?: string; variantId?: string }

interface Cart {
  id: string
  createdAt: string          // ISO 8601 timestamp (derived from Unix timestamp)
  currency: { code: string } // e.g. 'USD'
  taxesIncluded: boolean     // always false for Commerce.js
  lineItems: LineItem[]
  lineItemsSubtotalPrice: number  // raw price value
  subtotalPrice: number           // raw price value
  totalPrice: number              // raw price value
}

interface LineItem {
  id: string
  variantId: string    // variant ID, or item ID if no variant
  productId: string
  name: string         // product name
  quantity: number
  discounts: any[]     // always empty array
  path: string         // product permalink
  options: Array<{ name: string; value: string }>  // selected variant options
  variant: {
    id: string
    sku: string
    name: string
    requiresShipping: boolean  // always false
    price: number              // raw price
    listPrice: number          // raw price (same as price)
    image: { url: string }
  }
}
```

## Capabilities

### useCart

Fetches and returns the current cart. The cart is retrieved via `commerce.cart.retrieve()`. Returns an SWR response augmented with an `isEmpty` computed property.

```typescript { .api }
/**
 * @param input - Optional SWR hook options
 * @returns SWR response with cart data and isEmpty flag
 */
function useCart(input?: {
  swrOptions?: {
    revalidateOnFocus?: boolean
    revalidateOnMount?: boolean
    revalidateOnReconnect?: boolean
    refreshInterval?: number
    [key: string]: any
  }
}): {
  data: Cart | null | undefined
  error: any
  isLoading: boolean
  isEmpty: boolean   // true when lineItems.length === 0 or cart is null
  mutate: (data?: Cart | null, shouldRevalidate?: boolean) => Promise<any>
}
```

**Default SWR options:** `revalidateOnFocus: false`

**Usage:**

```typescript
import { useCart } from '@vercel/commerce-commercejs/cart'

function CartSummary() {
  const { data: cart, isEmpty, isLoading } = useCart()

  if (isLoading) return <div>Loading...</div>
  if (isEmpty) return <div>Your cart is empty</div>

  return (
    <div>
      <p>Items: {cart?.lineItems.length}</p>
      <p>Total: ${cart?.totalPrice}</p>
      {cart?.lineItems.map(item => (
        <div key={item.id}>
          {item.name} × {item.quantity}
        </div>
      ))}
    </div>
  )
}
```

### useAddItem

Returns a function that adds a product to the cart. Calls `commerce.cart.add(productId, quantity, variantId?)`. The cart SWR cache is updated immediately after a successful add.

```typescript { .api }
/**
 * @returns Async function to add an item to the cart
 */
function useAddItem(): (input: AddItemInput) => Promise<Cart>

interface AddItemInput {
  productId: string    // Commerce.js product ID
  variantId?: string   // Commerce.js variant ID (optional)
  quantity?: number    // Quantity to add, default: 1
}
```

**Notes:**
- If `variantId` is `undefined` or the string `'undefined'`, the variant parameter is omitted from the SDK call
- Updates cart SWR cache after successful add (no refetch)

**Usage:**

```typescript
import { useAddItem } from '@vercel/commerce-commercejs/cart'

function AddToCartButton({ product }) {
  const addItem = useAddItem()

  const handleAdd = async () => {
    try {
      const cart = await addItem({
        productId: product.id,
        variantId: product.variants[0]?.id,
        quantity: 1,
      })
      console.log('Cart updated:', cart)
    } catch (error) {
      console.error('Failed to add item:', error)
    }
  }

  return <button onClick={handleAdd}>Add to Cart</button>
}
```

### useUpdateItem

Returns a debounced function that updates a cart item's quantity. Calls `commerce.cart.update(itemId, { quantity })`. Debounced by default to avoid excessive API calls during rapid quantity changes.

```typescript { .api }
/**
 * @param ctx - Optional context with a pre-bound line item and debounce wait time
 * @returns Debounced async function to update an item's quantity
 */
function useUpdateItem(ctx?: {
  item?: LineItem   // Pre-bound line item (provides default id, productId, variantId, quantity)
  wait?: number     // Debounce delay in ms, default: 500
}): (input: UpdateItemInput) => Promise<Cart>

interface UpdateItemInput {
  id?: string          // Cart item ID (overrides ctx.item.id)
  quantity?: number    // New quantity (overrides ctx.item.quantity)
  productId?: string   // Product ID (overrides ctx.item.productId)
  variantId?: string   // Variant ID (overrides ctx.item.variantId)
}
```

**Errors:**
- Throws `ValidationError` if `itemId`, `productId`, or `variantId` cannot be resolved from input or ctx.item

**Usage:**

```typescript
import { useUpdateItem } from '@vercel/commerce-commercejs/cart'
import type { LineItem } from '@vercel/commerce/types/cart'

// Unbound (provide all fields in input)
function QuantityInput({ item }: { item: LineItem }) {
  const updateItem = useUpdateItem({ item, wait: 500 })

  return (
    <input
      type="number"
      defaultValue={item.quantity}
      onChange={(e) => updateItem({ quantity: parseInt(e.target.value) })}
    />
  )
}

// Bound to a specific item
function CartItemRow({ item }: { item: LineItem }) {
  const updateItem = useUpdateItem({ item })

  const increment = () => updateItem({ quantity: item.quantity + 1 })
  const decrement = () => updateItem({ quantity: item.quantity - 1 })

  return (
    <div>
      <button onClick={decrement}>-</button>
      <span>{item.quantity}</span>
      <button onClick={increment}>+</button>
    </div>
  )
}
```

### useRemoveItem

Returns a function that removes an item from the cart. Calls `commerce.cart.remove(itemId)`. The cart SWR cache is updated immediately after removal.

```typescript { .api }
/**
 * @returns Async function to remove an item from the cart
 */
function useRemoveItem(): (input: { id: string }) => Promise<Cart>
```

**Parameters:**
- `input.id` - The cart line item ID to remove

**Usage:**

```typescript
import { useRemoveItem } from '@vercel/commerce-commercejs/cart'

function RemoveButton({ itemId }: { itemId: string }) {
  const removeItem = useRemoveItem()

  const handleRemove = async () => {
    try {
      await removeItem({ id: itemId })
    } catch (error) {
      console.error('Failed to remove item:', error)
    }
  }

  return <button onClick={handleRemove}>Remove</button>
}
```
