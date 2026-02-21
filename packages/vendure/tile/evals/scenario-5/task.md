# Customer Registration Form

## Overview

Implement a React registration form component that creates a new customer account using the Vendure commerce package's signup functionality.

## Capabilities

### Register a new customer account

Create a `SignupForm` component that:

- Renders `firstName`, `lastName`, `email`, and `password` input fields and a submit button
- Uses the package's signup hook on form submission
- Passes `{ firstName, lastName, email, password }` to the signup function
- On success, calls an `onSuccess` callback prop

[@test](./tests/signup-form.test.tsx)

### Error on missing required fields

Show that calling the signup function with any of the four required fields missing (firstName, lastName, email, or password) throws a structured error (not a plain Error) without making a network request.

[@test](./tests/missing-fields.test.ts)

### ValidationError on registration failure

Show that when the Vendure API returns a non-Success result (e.g. an error result), the signup function throws a `ValidationError`.

[@test](./tests/signup-failure.test.ts)

## Implementation

[@generates](./src/signup-form.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Provides a `useSignup` hook that executes the `registerCustomerAccount` GraphQL mutation. Requires firstName, lastName, email, and password.

[@satisfied-by](@vercel/commerce-vendure)
