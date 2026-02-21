# Add to Cart Button Component

Implement a React component that adds a product variant to the shopping cart when a button is clicked.

## Requirements

1. Use the add-item mutation hook from the Kibo Commerce package to add a product to the cart.
2. The component receives three props: `productId` (string), `variantId` (string), and `quantity` (number, default 1).
3. Render a button with the text "Add to Cart".
4. When the button is clicked, invoke the hook's returned function with the product details and quantity.
5. While the mutation is in flight, disable the button and change its text to "Adding...".
6. If an error occurs, display the error message below the button.

## Notes

- The mutation hook returns a function that accepts `{ productId, variantId, quantity }`.
- The hook's returned function is async; handle it with try/catch.
- Quantity must be a positive integer—the package validates this server-side but your component should pass a valid value.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useAddItem hook for cart mutations.

## Test Cases

- [@test](./tests/renders-button.test.tsx) The component renders a button with text "Add to Cart" in its initial state.
- [@test](./tests/calls-add-item.test.tsx) Clicking the button calls the mutation function returned by the add-item hook with the correct productId, variantId, and quantity.
- [@test](./tests/shows-loading-state.test.tsx) While the mutation is in progress, the button is disabled and shows "Adding..." text.
- [@test](./tests/shows-error.test.tsx) When the mutation throws an error, the error message is displayed below the button.
