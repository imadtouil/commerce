# Authentication

Client-side React hooks for customer authentication: login, logout, and account registration. All hooks require the app to be wrapped in `CommerceProvider`.

## Import

```typescript
import { useLogin, useLogout, useSignup } from '@vercel/commerce-vendure/auth'
// or individually:
import useLogin from '@vercel/commerce-vendure/auth/use-login'
import useLogout from '@vercel/commerce-vendure/auth/use-logout'
import useSignup from '@vercel/commerce-vendure/auth/use-signup'
```

## Types

```typescript { .api }
interface LoginInput {
  email: string
  password: string
}

interface SignupInput {
  email: string
  firstName: string
  lastName: string
  password: string
}
```

## Capabilities

### useLogin

Returns a function to authenticate a customer with email and password. On success, refreshes the customer data (via `useCustomer` mutate).

```typescript { .api }
/**
 * Returns a login function that authenticates a customer.
 * Refreshes customer data on successful login.
 * @returns Async login function that returns null on success
 * @throws CommerceError if email or password is not provided
 * @throws ValidationError if authentication fails (wrong credentials, unverified account, etc.)
 */
function useLogin(): (input: LoginInput) => Promise<null>

interface LoginInput {
  email: string
  password: string
}
```

**Usage:**

```typescript
import { useLogin } from '@vercel/commerce-vendure/auth'

function LoginForm() {
  const login = useLogin()
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault()
    try {
      await login({ email, password })
      // Customer is now logged in; useCustomer will reflect the new state
    } catch (error) {
      console.error('Login failed:', error.message)
    }
  }

  return (
    <form onSubmit={handleLogin}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">Login</button>
    </form>
  )
}
```

**Errors:**

- `CommerceError` with message `'A email and password are required to login'` — if `email` or `password` is missing
- `ValidationError` — for Vendure errors: `NativeAuthStrategyError`, `InvalidCredentialsError`, `NotVerifiedError`

### useLogout

Returns a function to log out the currently authenticated customer. Clears customer data on success.

```typescript { .api }
/**
 * Returns a logout function that logs out the current customer.
 * Clears customer data (sets to null) on success.
 * @returns Async logout function that returns null
 */
function useLogout(): () => Promise<null>
```

**Usage:**

```typescript
import { useLogout } from '@vercel/commerce-vendure/auth'

function LogoutButton() {
  const logout = useLogout()

  const handleLogout = async () => {
    await logout()
    // Customer data cleared; useCustomer will return null
  }

  return <button onClick={handleLogout}>Logout</button>
}
```

### useSignup

Returns a function to register a new customer account. Requires all fields: `firstName`, `lastName`, `email`, and `password`. On success, refreshes customer data.

```typescript { .api }
/**
 * Returns a signup function that registers a new customer account.
 * All fields are required. Refreshes customer data on success.
 * @returns Async signup function that returns null on success
 * @throws CommerceError if any required field is missing
 * @throws ValidationError if registration fails (e.g., email already in use)
 */
function useSignup(): (input: SignupInput) => Promise<null>

interface SignupInput {
  email: string
  firstName: string
  lastName: string
  password: string
}
```

**Usage:**

```typescript
import { useSignup } from '@vercel/commerce-vendure/auth'

function SignupForm() {
  const signup = useSignup()

  const handleSignup = async (data: SignupInput) => {
    try {
      await signup({
        firstName: 'Jane',
        lastName: 'Doe',
        email: 'jane@example.com',
        password: 'securepassword',
      })
      // Customer registered and logged in
    } catch (error) {
      console.error('Signup failed:', error.message)
    }
  }
}
```

**Errors:**

- `CommerceError` with message `'A first name, last name, email and password are required to signup'` — if any field is missing
- `ValidationError` — if Vendure registration returns an error (e.g., `EmailAddressConflictError`)

## Error Handling

Authentication errors come from `@vercel/commerce/utils/errors`:

- `CommerceError` — missing required input fields
- `ValidationError` — Vendure API-level authentication failures

The Vendure `login` mutation can return these error types:
- `NativeAuthStrategyError` — auth strategy not configured for native auth
- `InvalidCredentialsError` — wrong email or password
- `NotVerifiedError` — account not yet verified

The Vendure `registerCustomerAccount` mutation returns:
- `Success` — registration succeeded
- Error types (e.g., `EmailAddressConflictError`) — registration failed; wrapped in `ValidationError`
