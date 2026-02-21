# Customer

Client-side React hook for reading the current authenticated customer. Must be used within a `CommerceProvider`.

## Import

```typescript
import useCustomer from '@vercel/commerce-kibocommerce/customer/use-customer'
```

## Capabilities

### useCustomer

SWR-based hook that fetches the current logged-in customer's data. Returns `null` if no customer is authenticated.

```typescript { .api }
import useCustomer from '@vercel/commerce-kibocommerce/customer/use-customer'

function useCustomer(input?: {
  swrOptions?: SWROptions
}): {
  data: Customer | null
  isLoading: boolean
  error?: Error
  mutate: (data?: Customer | null, shouldRevalidate?: boolean) => Promise<void>
}

interface Customer {
  id: string
  firstName: string
  lastName: string
  email: string
  acceptsMarketing: boolean
}
```

**Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `swrOptions` | `SWROptions` | Optional SWR configuration overrides (default: `revalidateOnFocus: false`) |

**Returns**:

| Field | Type | Description |
|-------|------|-------------|
| `data` | `Customer \| null` | Current customer data, `null` if not authenticated |
| `isLoading` | `boolean` | Whether the request is in-flight |
| `error` | `Error \| undefined` | Error from the fetch, if any |
| `mutate` | `function` | SWR mutate to manually update customer state |

**API endpoint**: `GET /api/commerce/customer`

The server-side handler:
1. Reads the customer cookie from the request
2. Decodes the base64-encoded access token
3. Fetches customer account data from Kibo GraphQL using `x-vol-user-claims` header
4. Returns normalized `Customer` data, or `null` if not authenticated

**Example**:

```typescript
import useCustomer from '@vercel/commerce-kibocommerce/customer/use-customer'

function AccountHeader() {
  const { data: customer, isLoading } = useCustomer()

  if (isLoading) return <div>Loading...</div>

  if (!customer) {
    return <a href="/login">Sign In</a>
  }

  return (
    <div>
      Welcome, {customer.firstName} {customer.lastName}!
    </div>
  )
}
```

```typescript
// Check authentication status
function ProtectedPage() {
  const { data: customer, isLoading } = useCustomer()

  if (isLoading) return <div>Loading...</div>
  if (!customer) {
    // Redirect to login or show login prompt
    return <div>Please log in to access this page.</div>
  }

  return <div>Protected content for {customer.email}</div>
}
```

**Note**: The `useCustomer` hook is automatically used internally by:
- `useWishlist` — to determine the customer ID for wishlist fetching
- `useLogin` — mutated on login to update customer state
- `useLogout` — set to `null` on logout
- `useSignup` — mutated on signup to update customer state

### useAddItem (Customer Address — Stub)

A stub hook for adding customer addresses. The implementation is a placeholder and returns an empty object. Do not use for production address management.

```typescript { .api }
import useAddItem from '@vercel/commerce-kibocommerce/customer/address/use-add-item'

// Stub — returns empty object; not implemented
function useAddItem(): () => Promise<{}>
```

### useAddItem (Customer Card — Stub)

A stub hook for adding customer payment cards. The implementation is a placeholder and returns an empty object. Do not use for production payment management.

```typescript { .api }
import useAddItem from '@vercel/commerce-kibocommerce/customer/card/use-add-item'

// Stub — returns empty object; not implemented
function useAddItem(): () => Promise<{}>
