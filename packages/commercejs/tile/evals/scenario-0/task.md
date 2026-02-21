# Retrieve and Display the Shopping Cart

Build a React component that retrieves the current shopping cart and displays its contents. The component should consume the cart hook provided by the commerce provider and render the cart's line items along with a message indicating whether the cart is empty.

## Capabilities

### Display cart line items

- When the cart contains items, render a list showing each item's name and quantity [@test](./tests/cart-items.test.tsx)
- When the cart is empty (isEmpty is true), render an "Empty cart" message [@test](./tests/empty-cart.test.tsx)
- The cart hook should be called without arguments and return both the cart data and an isEmpty flag [@test](./tests/cart-hook.test.tsx)
- While cart data is loading (data is undefined), render a "Loading..." message [@test](./tests/cart-loading.test.tsx)

## Implementation

[@generates](./src/CartDisplay.tsx)

## API

```typescript { #api }
export function CartDisplay(): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a cart hook that retrieves the current shopping cart with an isEmpty computed property.

[@satisfied-by](@vercel/commerce-commercejs)
