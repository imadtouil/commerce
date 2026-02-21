# Customer Profile

Customer data management including profile retrieval and stub implementations for card and address management.

## Import

```typescript
import { useCustomer } from '@vercel/commerce-commercejs/customer'
import { useCards, useAddItem as useAddCard } from '@vercel/commerce-commercejs/customer/card'
import { useAddresses, useAddItem as useAddAddress } from '@vercel/commerce-commercejs/customer/address'
```

## Types

```typescript { .api }
interface Customer {
  id: string
  firstName: string
  lastName: string
  email: string
  phone: string
}
```

## Capabilities

### useCustomer

Returns authenticated customer profile data. Reads the `commercejs_customer_token` JWT cookie, decodes it to get the customer ID, and fetches the customer from the Commerce.js API.

```typescript { .api }
/**
 * @param input - Optional SWR hook options
 * @returns SWR response with customer data or null if not authenticated
 */
function useCustomer(input?: {
  swrOptions?: {
    revalidateOnFocus?: boolean
    revalidateOnMount?: boolean
    revalidateOnReconnect?: boolean
    refreshInterval?: number
    [key: string]: any
  }
}): {
  data: Customer | null | undefined
  error: any
  isLoading: boolean
  mutate: (data?: Customer | null, shouldRevalidate?: boolean) => Promise<any>
}
```

**Authentication check:**
- Returns `null` if no `commercejs_customer_token` cookie is present (user not logged in)
- Decodes the JWT to extract the customer ID (`cid` claim)
- Fetches customer data from `https://api.chec.io/v1/customers/:id` using the JWT as bearer token

**Default SWR options:** `revalidateOnFocus: false`

**Usage:**

```typescript
import { useCustomer } from '@vercel/commerce-commercejs/customer'

function CustomerProfile() {
  const { data: customer, isLoading } = useCustomer()

  if (isLoading) return <div>Loading...</div>
  if (!customer) return <div>Not logged in</div>

  return (
    <div>
      <p>Name: {customer.firstName} {customer.lastName}</p>
      <p>Email: {customer.email}</p>
      <p>Phone: {customer.phone}</p>
    </div>
  )
}
```

### useCards (Stub)

**Note: This hook is a stub.** Returns an object with `isEmpty: true`. No actual card data is fetched.

```typescript { .api }
/**
 * Stub implementation - always returns isEmpty: true
 * @returns Object with isEmpty flag
 */
function useCards(): { isEmpty: true }
```

**Usage:**

```typescript
import { useCards } from '@vercel/commerce-commercejs/customer/card'

function SavedCards() {
  const { isEmpty } = useCards()
  if (isEmpty) return <p>No saved cards (feature not implemented)</p>
  return null
}
```

### useAddItem (Card) (Stub)

**Note: This hook is a stub.** Returns a no-op async function.

```typescript { .api }
/**
 * Stub implementation - does nothing
 * @returns Async no-op function
 */
function useAddItem(): () => Promise<{}>
```

From `@vercel/commerce-commercejs/customer/card`.

### useAddresses (Stub)

**Note: This hook is a stub.** Returns an object with `isEmpty: true`. No actual address data is fetched.

```typescript { .api }
/**
 * Stub implementation - always returns isEmpty: true
 * @returns Object with isEmpty flag
 */
function useAddresses(): { isEmpty: true }
```

**Usage:**

```typescript
import { useAddresses } from '@vercel/commerce-commercejs/customer/address'

function SavedAddresses() {
  const { isEmpty } = useAddresses()
  if (isEmpty) return <p>No saved addresses (feature not implemented)</p>
  return null
}
```

### useAddItem (Address) (Stub)

**Note: This hook is a stub.** Returns a no-op async function.

```typescript { .api }
/**
 * Stub implementation - does nothing
 * @returns Async no-op function
 */
function useAddItem(): () => Promise<{}>
```

From `@vercel/commerce-commercejs/customer/address`.
