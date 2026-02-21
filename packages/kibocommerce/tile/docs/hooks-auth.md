# Authentication

Client-side React hooks for customer authentication: login, logout, and account signup. All hooks must be used within a `CommerceProvider`.

## Imports

```typescript
import useLogin from '@vercel/commerce-kibocommerce/auth/use-login'
import useLogout from '@vercel/commerce-kibocommerce/auth/use-logout'
import useSignup from '@vercel/commerce-kibocommerce/auth/use-signup'
```

## Capabilities

### useLogin

Mutation hook that returns an async function to authenticate a customer. On success, invalidates and revalidates both the customer and cart SWR caches.

```typescript { .api }
import useLogin from '@vercel/commerce-kibocommerce/auth/use-login'

function useLogin(): (input: LoginInput) => Promise<void>

interface LoginInput {
  email: string
  password: string
}
```

**Parameters**:

| Field | Type | Description |
|-------|------|-------------|
| `email` | `string` | Customer email address |
| `password` | `string` | Customer password |

**Errors**: Throws `CommerceError` if `email` or `password` is missing.

**Side effects**: On successful login:
- Revalidates `useCustomer` data (fetches current customer)
- Revalidates `useCart` data (merges anonymous cart with customer cart)

**Example**:

```typescript
import useLogin from '@vercel/commerce-kibocommerce/auth/use-login'
import { useState } from 'react'

function LoginForm() {
  const login = useLogin()
  const [error, setError] = useState<string | null>(null)

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const form = e.currentTarget
    setError(null)
    try {
      await login({
        email: form.email.value,
        password: form.password.value,
      })
      // Redirect or update UI after successful login
    } catch (err: any) {
      setError(err.message)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      {error && <p>{error}</p>}
      <button type="submit">Login</button>
    </form>
  )
}
```

**API endpoint**: `POST /api/commerce/login` with body `{ email, password }`

### useLogout

Mutation hook that returns an async function to log out the current customer. Clears both customer and cart state (sets them to `null`) without revalidating.

```typescript { .api }
import useLogout from '@vercel/commerce-kibocommerce/auth/use-logout'

function useLogout(): () => Promise<void>
```

**Side effects**: On successful logout:
- Sets `useCustomer` data to `null`
- Sets `useCart` data to `null`

**Example**:

```typescript
import useLogout from '@vercel/commerce-kibocommerce/auth/use-logout'

function LogoutButton() {
  const logout = useLogout()

  return (
    <button onClick={() => logout()}>
      Logout
    </button>
  )
}
```

**API endpoint**: `GET /api/commerce/logout`

### useSignup

Mutation hook that returns an async function to create a new customer account. On success, revalidates the customer SWR cache.

```typescript { .api }
import useSignup from '@vercel/commerce-kibocommerce/auth/use-signup'

function useSignup(): (input: SignupInput) => Promise<void>

interface SignupInput {
  firstName: string
  lastName: string
  email: string
  password: string
}
```

**Parameters**:

| Field | Type | Description |
|-------|------|-------------|
| `firstName` | `string` | Customer first name |
| `lastName` | `string` | Customer last name |
| `email` | `string` | Customer email address |
| `password` | `string` | Customer password |

**Errors**: Throws `CommerceError` if any of the four required fields is missing.

**Side effects**: On successful signup:
- Revalidates `useCustomer` data (fetches newly created customer)

**Example**:

```typescript
import useSignup from '@vercel/commerce-kibocommerce/auth/use-signup'

function SignupForm() {
  const signup = useSignup()

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const form = e.currentTarget
    await signup({
      firstName: form.firstName.value,
      lastName: form.lastName.value,
      email: form.email.value,
      password: form.password.value,
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="firstName" required />
      <input name="lastName" required />
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit">Create Account</button>
    </form>
  )
}
```

**API endpoint**: `POST /api/commerce/signup` with body `{ firstName, lastName, email, password }`
