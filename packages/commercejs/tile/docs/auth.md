# Authentication

Customer authentication using Commerce.js magic-link login. The user provides their email; Commerce.js sends a login link. When the user clicks the link, the token is exchanged for a JWT stored in a cookie.

## Import

```typescript
import { useLogin, useLogout, useSignup } from '@vercel/commerce-commercejs/auth'
```

## Authentication Flow

1. Call `login({ email })` → Commerce.js sends a magic-link email
2. User clicks the link (e.g., `/api/login/:token`)
3. The login API endpoint exchanges the token for a JWT and sets the `commercejs_customer_token` cookie
4. The `useCustomer()` hook reads the cookie and fetches the customer profile
5. Call `logout()` to clear the cookie and reset customer state

The login callback URL is automatically derived from:
- `NEXT_PUBLIC_COMMERCEJS_DEPLOYMENT_URL` (if set)
- `https://${NEXT_PUBLIC_VERCEL_URL}` (if on Vercel)
- `http://localhost:3000` (fallback for local development)

The Next.js rewrite `/api/login/:token` → `/api/login?token=:token` must be configured (provided by `@vercel/commerce-commercejs/next.config`).

## Capabilities

### useLogin

Initiates magic-link login by sending an email via Commerce.js. Calls `commerce.customer.login(email, callbackUrl)`.

```typescript { .api }
/**
 * @returns Async function to initiate email-based magic-link login
 */
function useLogin(): (input: LoginInput) => Promise<null>

interface LoginInput {
  email: string   // Customer email address to send the magic link to
}
```

**Returns:** Always `null` (login is asynchronous via email)

**Usage:**

```typescript
import { useLogin } from '@vercel/commerce-commercejs/auth'

function LoginForm() {
  const login = useLogin()
  const [email, setEmail] = useState('')
  const [sent, setSent] = useState(false)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    try {
      await login({ email })
      setSent(true)
    } catch (error) {
      console.error('Login failed:', error)
    }
  }

  if (sent) return <p>Check your email for a login link.</p>

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
        placeholder="Enter your email"
      />
      <button type="submit">Send Login Link</button>
    </form>
  )
}
```

### useLogout

Clears the customer session by removing the `commercejs_customer_token` cookie and resetting the customer SWR cache to `null`.

```typescript { .api }
/**
 * @returns Async function to log out the current customer
 */
function useLogout(): () => Promise<null>
```

**Side effects:**
- Removes the `commercejs_customer_token` cookie
- Mutates the customer SWR cache to `null` (without triggering revalidation)

**Returns:** Always `null`

**Usage:**

```typescript
import { useLogout } from '@vercel/commerce-commercejs/auth'

function LogoutButton() {
  const logout = useLogout()

  return <button onClick={logout}>Log Out</button>
}
```

### useSignup

**Note: This hook is a stub and not implemented.** Returns a no-op function.

```typescript { .api }
/**
 * @returns No-op function (signup not implemented)
 */
function useSignup(): () => void
```

**Usage:** Not recommended; Commerce.js signup is not supported in this provider.

## API Endpoint: Login Callback

The login callback endpoint must be registered as a Next.js API route at `pages/api/login.ts` (or included via `commercejsAPI(commerce)`). The Next.js rewrite from `next.config` converts `/api/login/:token` to `/api/login?token=:token`.

```typescript
// pages/api/commerce/[...commerce].ts
import commercejsAPI from '@vercel/commerce-commercejs/api/endpoints'
import { getCommerceApi } from '@vercel/commerce-commercejs/api'

const commerce = getCommerceApi()
export default commercejsAPI(commerce)
```

**Login handler behavior:**
- Reads `token` query parameter from the URL
- If no token: redirects to the deployment URL
- If token present: calls `commerce.customer.getToken(token, false)` to get a JWT
- Sets `commercejs_customer_token` cookie (max-age: 86400 seconds / 24 hours, secure in production)
- Returns redirect headers
