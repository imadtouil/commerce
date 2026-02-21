# Customer Login Form

## Overview

Implement a React login form component that authenticates a customer using the Vendure commerce package's login functionality.

## Capabilities

### Login with email and password

Create a `LoginForm` component that:

- Renders email and password input fields and a submit button
- Uses the package's login hook on form submission
- Passes `{ email, password }` to the login function
- On success, the component signals completion (e.g. calls an `onSuccess` callback prop)

[@test](./tests/login-form.test.tsx)

### Error on missing credentials

Show that calling the login function with an empty email or password throws a structured error (not a plain Error), before any network request is made.

[@test](./tests/missing-credentials.test.ts)

### ValidationError on failed login

Show that when the Vendure API returns an `InvalidCredentialsError` or `NotVerifiedError` result, the login function throws a `ValidationError` (not a generic network error).

[@test](./tests/invalid-credentials.test.ts)

## Implementation

[@generates](./src/login-form.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Provides a `useLogin` hook for authenticating customers via the Vendure Shop API's login GraphQL mutation.

[@satisfied-by](@vercel/commerce-vendure)
