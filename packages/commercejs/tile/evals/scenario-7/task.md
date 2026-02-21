# Retrieve the Logged-In Customer Profile

Build a React component that displays the currently logged-in customer's profile information. The component should use the customer hook from the commerce provider, which reads the customer's JWT from a cookie, decodes it to get the customer ID, then fetches the full profile.

## Capabilities

### Customer profile display

- When a customer is logged in, the component renders their first name, last name, and email [@test](./tests/customer-profile-display.test.tsx)
- When no customer is logged in (hook returns null), the component renders a "Not logged in" message [@test](./tests/customer-not-logged-in.test.tsx)
- The hook reads the JWT token from the `commercejs_customer_token` cookie (the hook handles this internally) [@test](./tests/customer-token-cookie.test.tsx)
- While loading, the component renders a "Loading..." message [@test](./tests/customer-loading.test.tsx)

## Implementation

[@generates](./src/CustomerProfile.tsx)

## API

```typescript { #api }
export function CustomerProfile(): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a customer hook that reads the customer JWT from a cookie, decodes it for the customer ID, and fetches the profile (firstName, lastName, email, phone).

[@satisfied-by](@vercel/commerce-commercejs)
