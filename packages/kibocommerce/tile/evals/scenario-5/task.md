# Customer Registration Form

Implement a React component that allows a new user to create an account on the Kibo Commerce storefront.

## Requirements

1. Use the signup mutation hook from the Kibo Commerce package to register a new customer.
2. Render a form with four fields: `firstName`, `lastName`, `email`, and `password`.
3. Render a submit button labeled "Create Account".
4. On form submission, call the signup mutation with `{ firstName, lastName, email, password }`.
5. While the mutation is in progress, disable the submit button.
6. Display any errors returned from the mutation below the form.
7. Accept an optional `onSuccess` prop callback that is called after a successful registration.

## Notes

- The signup hook performs a two-step operation: account creation and immediate login. Your component does not need to implement this logic—it is handled by the hook internally.
- All four fields are required by the package.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useSignup hook for new customer registration.

## Test Cases

- [@test](./tests/renders-all-fields.test.tsx) The form renders firstName, lastName, email, and password inputs plus a submit button.
- [@test](./tests/calls-signup-on-submit.test.tsx) Submitting the form calls the signup mutation with all four field values.
- [@test](./tests/disables-button-during-signup.test.tsx) The submit button is disabled while the signup mutation is in progress.
- [@test](./tests/shows-error-on-failure.test.tsx) If the mutation rejects, the error message is rendered in the UI.
