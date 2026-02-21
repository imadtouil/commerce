# Wishlist Management

Client-side React hooks for customer wishlist operations. All wishlist hooks require an authenticated customer — they will throw `CommerceError` if used without a logged-in customer.

## Imports

```typescript
import useWishlist from '@vercel/commerce-kibocommerce/wishlist/use-wishlist'
import useAddItem from '@vercel/commerce-kibocommerce/wishlist/use-add-item'
import useRemoveItem from '@vercel/commerce-kibocommerce/wishlist/use-remove-item'
```

## Capabilities

### useWishlist

SWR-based hook that fetches the current customer's wishlist. Automatically uses the customer ID from `useCustomer`. Returns `null` if no customer is logged in.

```typescript { .api }
import useWishlist from '@vercel/commerce-kibocommerce/wishlist/use-wishlist'

function useWishlist(input?: {
  includeProducts?: boolean
  swrOptions?: SWROptions
}): {
  data: Wishlist | null
  isEmpty: boolean
  isLoading: boolean
  error?: Error
  mutate: (data?: Wishlist | null, shouldRevalidate?: boolean) => Promise<void>
}
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `includeProducts` | `boolean` | Whether to include full product details in wishlist items (default: `false`) |
| `swrOptions` | `SWROptions` | SWR configuration overrides (default: `revalidateOnFocus: false`) |

**Returns**:

| Field | Type | Description |
|-------|------|-------------|
| `data` | `Wishlist \| null` | Wishlist data, or `null` if not logged in |
| `isEmpty` | `boolean` | Computed: `true` when `data.items.length === 0` |
| `isLoading` | `boolean` | Whether request is in-flight |
| `error` | `Error \| undefined` | Fetch error if any |
| `mutate` | `function` | SWR mutate to manually update wishlist |

**Behavior**: Internally uses `useCustomer()` to get `customer.id`. If no customer is logged in, returns `null` without making any API request.

**API endpoint**: `GET /api/commerce/wishlist?customerId=<id>&products=1` (products param only when `includeProducts: true`)

**Example**:

```typescript
import useWishlist from '@vercel/commerce-kibocommerce/wishlist/use-wishlist'

function WishlistDisplay() {
  const { data: wishlist, isEmpty, isLoading } = useWishlist({
    includeProducts: true,
  })

  if (isLoading) return <div>Loading wishlist...</div>
  if (!wishlist) return <div>Please log in to view wishlist</div>
  if (isEmpty) return <div>Your wishlist is empty</div>

  return (
    <ul>
      {wishlist.items?.map((item) => (
        <li key={item.id}>{item.product?.name}</li>
      ))}
    </ul>
  )
}
```

### useAddItem (Wishlist)

Mutation hook that returns an async function to add a product to the wishlist. Requires an authenticated customer.

```typescript { .api }
import useAddItem from '@vercel/commerce-kibocommerce/wishlist/use-add-item'

function useAddItem(): (item: WishlistItemInput) => Promise<Wishlist>

interface WishlistItemInput {
  productId: string
  variantId: string
}
```

**Parameters**:

| Field | Type | Description |
|-------|------|-------------|
| `productId` | `string` | Product ID to add to wishlist |
| `variantId` | `string` | Variant ID of the product |

**Errors**: Throws `CommerceError` with message `'Signed customer not found'` if no customer is logged in.

**Side effects**: Revalidates `useWishlist` cache after adding.

**API endpoint**: `POST /api/commerce/wishlist` with body `{ item: WishlistItemInput }`

**Example**:

```typescript
import useAddItem from '@vercel/commerce-kibocommerce/wishlist/use-add-item'

function AddToWishlistButton({ product, variant }) {
  const addItem = useAddItem()

  const handleAddToWishlist = async () => {
    try {
      await addItem({
        productId: product.id,
        variantId: variant.id,
      })
    } catch (error: any) {
      if (error.message === 'Signed customer not found') {
        // Redirect to login
      }
    }
  }

  return <button onClick={handleAddToWishlist}>Add to Wishlist</button>
}
```

### useRemoveItem (Wishlist)

Mutation hook that returns an async function to remove an item from the wishlist by item ID. Requires an authenticated customer.

```typescript { .api }
import useRemoveItem from '@vercel/commerce-kibocommerce/wishlist/use-remove-item'

function useRemoveItem(ctx?: {
  wishlist?: Wishlist   // Optional: provide wishlist for revalidation context
}): (input: { id: string | number }) => Promise<Wishlist>
```

**Parameters**:

| Field | Type | Description |
|-------|------|-------------|
| `input.id` | `string \| number` | The wishlist item's `id` field from `WishlistItem`. When `useWishlist` is called with `includeProducts: false` (default), `item.id` = productCode. The server matches removal by productCode. |

**Context**:

| Field | Type | Description |
|-------|------|-------------|
| `ctx.wishlist` | `Wishlist \| undefined` | Optional wishlist context (passed to `useWishlist` for revalidation) |

**Errors**: Throws `CommerceError` with message `'Signed customer not found'` if no customer is logged in.

**Side effects**: Revalidates `useWishlist` cache after removing.

**API endpoint**: `DELETE /api/commerce/wishlist` with body `{ itemId: String(id) }`

**Example**:

```typescript
import useRemoveItem from '@vercel/commerce-kibocommerce/wishlist/use-remove-item'

function WishlistItem({ item }) {
  const removeItem = useRemoveItem()

  const handleRemove = async () => {
    await removeItem({ id: item.id })
  }

  return (
    <div>
      <span>{item.product?.name}</span>
      <button onClick={handleRemove}>Remove</button>
    </div>
  )
}
```

## Wishlist Data Shape

See [Types Reference](./types.md#wishlist) for the `Wishlist` and `WishlistItem` type definitions.
