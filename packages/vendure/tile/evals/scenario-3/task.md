# Update Cart Line Item Quantity

## Overview

Implement a React component that updates the quantity of an existing line item in the Vendure shopping cart using the package's update-item hook.

## Capabilities

### Update quantity for an existing line item

Create a `QuantitySelector` component that:

- Accepts a `lineItem` prop with `id`, `productId`, `variantId`, and `quantity` fields
- Uses the package's update-item hook, passing the `lineItem` as the item context
- Renders increment and decrement buttons that call the returned update function with the new quantity
- The update function must receive an object with at least `quantity` (and optionally `productId`/`variantId` from the context item)

[@test](./tests/quantity-selector.test.tsx)

### ValidationError on missing IDs

Show that calling the update function without a valid `itemId`, `productId`, or `variantId` throws a structured validation error (not a plain Error).

[@test](./tests/update-validation.test.ts)

### Cart mutation after update

Demonstrate that after a successful update, the cart cache is refreshed (mutate is called) so subsequent renders show the updated quantity.

[@test](./tests/cart-refresh.test.ts)

## Implementation

[@generates](./src/quantity-selector.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Provides a `useUpdateItem` hook that executes the `adjustOrderLine` GraphQL mutation. The hook accepts an optional item context to pre-populate itemId, productId, and variantId.

[@satisfied-by](@vercel/commerce-vendure)
