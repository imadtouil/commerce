# Shopping Cart Display Component

Build a React component that displays the current state of a customer's shopping cart.

## Requirements

1. Use the cart query hook provided by the Kibo Commerce package to fetch the current cart.
2. Display a loading indicator while the cart data is being fetched.
3. If the cart is empty (use the `isEmpty` property returned by the hook), render a message saying "Your cart is empty".
4. If the cart contains items, render an unordered list where each item shows the product name and quantity.
5. Export the component as the default export.

## Notes

- The hook is accessed via the commerce context and provides `data`, `isLoading`, and `error` fields.
- Cart line items are available at `data.lineItems`.
- Each line item has a `name` and `quantity` property.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useCart hook for fetching the current shopping cart state.

## Test Cases

- [@test](./tests/shows-loading.test.tsx) When the cart hook returns `isLoading: true`, the component renders a loading indicator.
- [@test](./tests/shows-empty-cart.test.tsx) When `data.isEmpty` is true, the component renders the "Your cart is empty" message.
- [@test](./tests/shows-line-items.test.tsx) When the cart has line items, each item's name and quantity are rendered.
- [@test](./tests/uses-cart-hook.test.tsx) The component calls the useCart hook from the package (not a custom fetch).
