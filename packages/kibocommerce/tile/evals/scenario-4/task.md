# Customer Login Form

Implement a React login form component that authenticates a customer against the Kibo Commerce backend.

## Requirements

1. Use the login mutation hook from the Kibo Commerce package to authenticate the user.
2. Render a form with two fields: `email` (text input of type "email") and `password` (input of type "password").
3. Render a submit button with the text "Sign In".
4. On form submission, call the login mutation function with `{ email, password }`.
5. While the login request is in progress, disable the submit button.
6. If login fails (the mutation throws), display the error message to the user.
7. Accept an optional `onSuccess` callback prop that is called after a successful login.

## Notes

- The login hook is a mutation hook that returns an async function.
- The login hook's mutation function expects an object with `email` and `password` fields.
- Both fields are required; the package validates this.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useLogin hook for customer authentication.

## Test Cases

- [@test](./tests/renders-form-fields.test.tsx) The component renders email, password inputs and a submit button.
- [@test](./tests/calls-login-on-submit.test.tsx) Submitting the form calls the login mutation with the correct email and password values.
- [@test](./tests/disables-button-during-login.test.tsx) The submit button is disabled while the login mutation is in progress.
- [@test](./tests/calls-on-success.test.tsx) The onSuccess callback is invoked after the login mutation resolves successfully.
