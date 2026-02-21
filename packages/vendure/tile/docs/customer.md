# Customer Profile

Client-side React hook for retrieving the currently authenticated customer's profile. Requires the app to be wrapped in `CommerceProvider`.

## Import

```typescript
import { useCustomer } from '@vercel/commerce-vendure/customer'
// or:
import useCustomer from '@vercel/commerce-vendure/customer/use-customer'
```

## Types

```typescript { .api }
interface Customer {
  firstName: string
  lastName: string
  email: string       // maps from emailAddress in Vendure API
}
```

## Capabilities

### useCustomer

Fetches the currently authenticated customer's profile from Vendure using the `activeCustomer` GraphQL query. Returns `null` if no customer is authenticated. Uses SWR for caching; does not revalidate on window focus.

```typescript { .api }
/**
 * Fetches the active customer profile.
 * @param input.swrOptions - SWR configuration options
 * @returns Customer profile or null if not authenticated
 */
function useCustomer(input?: {
  swrOptions?: {
    revalidateOnFocus?: boolean
    [key: string]: any
  }
}): Customer | null
```

**Usage:**

```typescript
import { useCustomer } from '@vercel/commerce-vendure/customer'

function UserProfile() {
  const customer = useCustomer()

  if (!customer) {
    return <p>Not logged in</p>
  }

  return (
    <div>
      <p>Name: {customer.firstName} {customer.lastName}</p>
      <p>Email: {customer.email}</p>
    </div>
  )
}
```

**Checking auth state:**

```typescript
import { useCustomer } from '@vercel/commerce-vendure/customer'

function NavBar() {
  const customer = useCustomer()
  const isLoggedIn = customer !== null

  return (
    <nav>
      {isLoggedIn ? (
        <span>Welcome, {customer.firstName}!</span>
      ) : (
        <a href="/login">Sign In</a>
      )}
    </nav>
  )
}
```

## Vendure GraphQL Query

The hook uses the `activeCustomer` query:

```graphql
query activeCustomer {
  activeCustomer {
    id
    firstName
    lastName
    emailAddress
  }
}
```

The `emailAddress` field from Vendure is mapped to `email` in the returned `Customer` object.

## Notes

- Customer state is automatically refreshed after `useLogin`, `useLogout`, and `useSignup` calls.
- The hook uses SWR with `revalidateOnFocus: false` by default.
- Customer address management (`customer/address/use-add-item`) and card management (`customer/card/use-add-item`) are stub implementations that are not functional.
