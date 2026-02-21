# Cart Management

Client-side React hooks for shopping cart operations. All hooks must be used within a `CommerceProvider`.

## Imports

```typescript
import useCart from '@vercel/commerce-kibocommerce/cart/use-cart'
import useAddItem from '@vercel/commerce-kibocommerce/cart/use-add-item'
import useUpdateItem from '@vercel/commerce-kibocommerce/cart/use-update-item'
import useRemoveItem from '@vercel/commerce-kibocommerce/cart/use-remove-item'
```

Or access via the provider package directly (re-exported in the `@vercel/commerce` hook system).

## Capabilities

### useCart

SWR-based hook that fetches and subscribes to the current cart state. Cart is identified via the `kibo_cart` cookie.

```typescript { .api }
import useCart from '@vercel/commerce-kibocommerce/cart/use-cart'

function useCart(input?: {
  swrOptions?: SWROptions
}): {
  data: Cart | null
  isEmpty: boolean
  isLoading: boolean
  error?: Error
  mutate: (data?: Cart | null, shouldRevalidate?: boolean) => Promise<void>
}
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `input.swrOptions` | `SWROptions` | Optional SWR configuration overrides (default: `revalidateOnFocus: false`) |

**Returns**:

| Field | Type | Description |
|-------|------|-------------|
| `data` | `Cart \| null` | Current cart data, null if not loaded |
| `isEmpty` | `boolean` | Computed: `true` when `data.lineItems.length === 0` |
| `isLoading` | `boolean` | Whether the request is in-flight |
| `error` | `Error \| undefined` | Error from the fetch, if any |
| `mutate` | `function` | SWR mutate function to manually update cart |

**Example**:

```typescript
import useCart from '@vercel/commerce-kibocommerce/cart/use-cart'

function CartSummary() {
  const { data: cart, isEmpty, isLoading } = useCart()

  if (isLoading) return <div>Loading cart...</div>
  if (isEmpty) return <div>Your cart is empty</div>

  return (
    <div>
      <p>Items: {cart?.lineItems.length}</p>
      <p>Total: {cart?.totalPrice}</p>
    </div>
  )
}
```

### useAddItem (Cart)

Mutation hook that returns an async function to add a product to the cart.

```typescript { .api }
import useAddItem from '@vercel/commerce-kibocommerce/cart/use-add-item'

function useAddItem(): (item: CartItemInput) => Promise<Cart>

interface CartItemInput {
  productId: string
  variantId: string
  quantity?: number    // Must be integer > 0, defaults to 1
}
```

**Errors**: Throws `CommerceError` if `quantity` is provided and is not an integer ≥ 1.

**Example**:

```typescript
import useAddItem from '@vercel/commerce-kibocommerce/cart/use-add-item'

function AddToCartButton({ product, variant }) {
  const addItem = useAddItem()

  const handleAddToCart = async () => {
    try {
      await addItem({
        productId: product.id,
        variantId: variant.id,
        quantity: 1,
      })
    } catch (error) {
      console.error('Failed to add item:', error)
    }
  }

  return <button onClick={handleAddToCart}>Add to Cart</button>
}
```

### useUpdateItem (Cart)

Mutation hook that returns a debounced async function to update a cart item's quantity. Automatically removes the item if quantity is set to 0 or less.

```typescript { .api }
import useUpdateItem from '@vercel/commerce-kibocommerce/cart/use-update-item'

// Type parameter T narrows input based on whether a line item is pre-bound via ctx
type UpdateItemActionInput<T = any> = T extends LineItem
  ? Partial<UpdateItemHook['actionInput']>
  : UpdateItemHook['actionInput']

function useUpdateItem<T extends LineItem | undefined = undefined>(ctx?: {
  item?: T      // Pre-bind a specific line item (input.id/productId/variantId become optional)
  wait?: number // Debounce delay in ms (default: 500)
}): (input: UpdateItemActionInput<T>) => Promise<Cart>
```

**Input fields** (when no ctx.item is bound):

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Cart line item ID |
| `productId` | `string` | Product ID |
| `variantId` | `string` | Variant ID |
| `quantity` | `number` | New quantity (integer); if < 1, item is removed |

**Errors**: Throws `ValidationError` if quantity is non-integer, or if `itemId`/`productId`/`variantId` cannot be resolved.

**Example**:

```typescript
import useUpdateItem from '@vercel/commerce-kibocommerce/cart/use-update-item'

function QuantitySelector({ lineItem }) {
  // Pre-bind the line item context
  const updateItem = useUpdateItem({ item: lineItem, wait: 300 })

  const handleChange = async (newQty: number) => {
    await updateItem({ quantity: newQty })
    // If newQty < 1, item is automatically removed
  }

  return (
    <input
      type="number"
      defaultValue={lineItem.quantity}
      onChange={(e) => handleChange(Number(e.target.value))}
    />
  )
}
```

### useRemoveItem (Cart)

Mutation hook that returns an async function to remove a line item from the cart.

```typescript { .api }
import useRemoveItem from '@vercel/commerce-kibocommerce/cart/use-remove-item'

type RemoveItemFn<T = any> = T extends LineItem
  ? (input?: RemoveItemActionInput<T>) => Promise<Cart | null | undefined>
  : (input: RemoveItemActionInput<T>) => Promise<Cart | null>

type RemoveItemActionInput<T = any> = T extends LineItem
  ? Partial<{ id: string }>
  : { id: string }

function useRemoveItem<T extends LineItem | undefined = undefined>(ctx?: {
  item?: T    // Pre-bind a specific line item (input.id becomes optional)
}): RemoveItemFn<T>
```

**Errors**: Throws `ValidationError` if `itemId` cannot be resolved (neither from input nor ctx.item).

**Example**:

```typescript
import useRemoveItem from '@vercel/commerce-kibocommerce/cart/use-remove-item'

function CartItem({ lineItem }) {
  const removeItem = useRemoveItem()

  const handleRemove = async () => {
    await removeItem({ id: lineItem.id })
  }

  // Or pre-bind the item
  const removeThisItem = useRemoveItem({ item: lineItem })
  const handleRemoveBound = async () => {
    await removeThisItem() // No argument needed
  }

  return <button onClick={handleRemove}>Remove</button>
}
```

## API Endpoint

The cart hooks communicate with the Next.js API at `/api/commerce/cart`:

| Method | Operation |
|--------|-----------|
| `GET` | Fetch current cart (`useCart`) |
| `POST` | Add item (`useAddItem`) with body `{ item: CartItemInput }` |
| `PUT` | Update item (`useUpdateItem`) with body `{ itemId, item: { productId, variantId, quantity } }` |
| `DELETE` | Remove item (`useRemoveItem`) with body `{ itemId }` |

The server-side cart handler:
1. Reads the customer cookie from the request
2. If no session exists, creates an anonymous shopper token and sets a new cookie
3. Fetches or mutates the Kibo cart via GraphQL using the shopper access token
4. Returns normalized `Cart` data

## Cart Data Shape

See [Types Reference](./types.md#cart) for the `Cart` and `LineItem` type definitions.
