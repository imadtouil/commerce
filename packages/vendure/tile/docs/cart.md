# Cart Management

Client-side React hooks for managing the Vendure shopping cart (active order). All hooks require the app to be wrapped in `CommerceProvider`.

## Import

```typescript
import { useCart, useAddItem, useUpdateItem, useRemoveItem } from '@vercel/commerce-vendure/cart'
// or individually:
import useCart from '@vercel/commerce-vendure/cart/use-cart'
import useAddItem from '@vercel/commerce-vendure/cart/use-add-item'
import useUpdateItem from '@vercel/commerce-vendure/cart/use-update-item'
import useRemoveItem from '@vercel/commerce-vendure/cart/use-remove-item'
```

## Types

```typescript { .api }
interface Cart {
  id: string
  createdAt: string
  taxesIncluded: boolean       // always true for Vendure
  lineItemsSubtotalPrice: number   // subTotalWithTax / 100
  currency: { code: string }
  subtotalPrice: number        // subTotalWithTax / 100
  totalPrice: number           // totalWithTax / 100
  customerId?: string
  lineItems: LineItem[]
  isEmpty: boolean             // computed: lineItems.length === 0
}

interface LineItem {
  id: string
  name: string                 // productVariant.name
  quantity: number
  url: string                  // productVariant.product.slug
  variantId: string
  productId: string
  images: Array<{ url: string }>
  discounts: Array<{ value: number }>  // amount / 100
  path: string                 // /${productVariant.product.slug}
  variant: {
    id: string
    name: string
    sku: string
    price: number              // discountedUnitPriceWithTax / 100
    listPrice: number          // unitPriceWithTax / 100
    image: { url: string }
    requiresShipping: boolean  // always true
  }
}
```

## Capabilities

### useCart

Fetches the current active cart (Vendure active order). Uses SWR for caching and revalidation. Returns `null` if no active order exists.

```typescript { .api }
/**
 * Fetches the currently active Vendure order (cart).
 * @param input.swrOptions - SWR configuration options
 * @returns Cart object with isEmpty computed property, or null if no active order
 */
function useCart(input?: {
  swrOptions?: {
    revalidateOnFocus?: boolean
    [key: string]: any
  }
}): (Cart & { isEmpty: boolean }) | null
```

**Usage:**

```typescript
import { useCart } from '@vercel/commerce-vendure/cart'

function CartIcon() {
  const cart = useCart()

  if (!cart || cart.isEmpty) {
    return <span>Empty cart</span>
  }

  return (
    <span>{cart.lineItems.length} items — ${cart.totalPrice}</span>
  )
}
```

### useAddItem

Returns a function to add a product variant to the cart. Automatically refreshes cart data after adding.

```typescript { .api }
/**
 * Returns a function to add a product variant to the cart.
 * Throws CommerceError if quantity is not a positive integer.
 * @returns Async function that adds item and returns updated Cart
 */
function useAddItem(): (input: AddItemInput) => Promise<Cart | null>

interface AddItemInput {
  variantId: string    // Vendure product variant ID
  quantity?: number    // Default: 1; must be a positive integer >= 1
}
```

**Usage:**

```typescript
import { useAddItem } from '@vercel/commerce-vendure/cart'

function AddToCartButton({ variantId }: { variantId: string }) {
  const addItem = useAddItem()

  const handleAddToCart = async () => {
    try {
      await addItem({ variantId, quantity: 1 })
    } catch (error) {
      console.error('Failed to add item:', error)
    }
  }

  return <button onClick={handleAddToCart}>Add to Cart</button>
}
```

**Errors:**

- `CommerceError` — thrown if `quantity` is provided but not a positive integer (< 1 or non-integer)

### useUpdateItem

Returns a function to update the quantity of an existing cart line item. Must be called with `item` context or provide all required IDs in the update input.

```typescript { .api }
/**
 * Returns a function to update a cart line item's quantity.
 * @param ctx.item - Existing LineItem to update (provides itemId, productId, variantId context)
 * @param ctx.wait - Debounce wait time in milliseconds
 * @returns Async function that updates item quantity and returns updated Cart
 */
function useUpdateItem(ctx?: {
  item?: LineItem
  wait?: number
}): (input: UpdateItemActionInput) => Promise<Cart | null>

interface UpdateItemActionInput {
  quantity: number
  productId?: string   // required if not provided via ctx.item
  variantId?: string   // required if not provided via ctx.item
}
```

**Usage:**

```typescript
import { useUpdateItem } from '@vercel/commerce-vendure/cart'

function QuantitySelector({ item }: { item: LineItem }) {
  const updateItem = useUpdateItem({ item })

  const handleQuantityChange = async (newQuantity: number) => {
    await updateItem({ quantity: newQuantity })
  }

  return (
    <input
      type="number"
      value={item.quantity}
      onChange={(e) => handleQuantityChange(Number(e.target.value))}
    />
  )
}
```

**Errors:**

- `ValidationError` — thrown if `itemId`, `productId`, or `variantId` cannot be resolved from ctx or input

### useRemoveItem

Returns a function to remove a line item from the cart by its ID.

```typescript { .api }
/**
 * Returns a function to remove a line item from the cart.
 * @returns Async function that removes item by id and returns updated Cart
 */
function useRemoveItem(): (input: RemoveItemInput) => Promise<Cart | null>

interface RemoveItemInput {
  id: string    // LineItem id to remove
}
```

**Usage:**

```typescript
import { useRemoveItem } from '@vercel/commerce-vendure/cart'

function RemoveButton({ itemId }: { itemId: string }) {
  const removeItem = useRemoveItem()

  const handleRemove = async () => {
    await removeItem({ id: itemId })
  }

  return <button onClick={handleRemove}>Remove</button>
}
```

## CartResult Type

Internal type from `use-cart.tsx`, representing possible cart mutation responses from Vendure:

```typescript { .api }
interface CartResult {
  activeOrder?: CartFragment
  addItemToOrder?: CartFragment
  adjustOrderLine?: CartFragment
  removeOrderLine?: CartFragment
}
```

## Error Handling

All cart mutation hooks throw errors from `@vercel/commerce`:

- `CommerceError` — general commerce errors (e.g., invalid quantity, Vendure API errors)
- `ValidationError` — validation failures (e.g., missing required fields)

Vendure GraphQL errors are wrapped automatically. Check `error.message` for details.
