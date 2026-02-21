# Customer Login via Passwordless Authentication

Build a React login form component that initiates the Commerce.js passwordless login flow. The component should collect the user's email address and, upon submission, call the login mutation hook provided by the commerce provider.

## Capabilities

### Passwordless login flow initiation

- Submitting the form calls the login mutation with the entered email address [@test](./tests/login-call.test.tsx)
- The mutation uses a callback URL derived from the deployment base URL, pointing to `/api/login` [@test](./tests/login-callback-url.test.tsx)
- When no custom deployment URL or Vercel URL is configured, the callback URL uses `http://localhost:3000` as the base [@test](./tests/login-default-base-url.test.tsx)
- After the mutation is called, the component does not navigate or redirect internally (the redirect is handled server-side) [@test](./tests/login-no-redirect.test.tsx)

## Implementation

[@generates](./src/LoginForm.tsx)

## API

```typescript { #api }
export function LoginForm(): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a login mutation hook that initiates a passwordless login by email, using a deployment URL helper to construct the callback URL.

[@satisfied-by](@vercel/commerce-commercejs)
