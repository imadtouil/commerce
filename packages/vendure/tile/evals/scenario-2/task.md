# Add Product Variant to Cart

## Overview

Implement a React component that allows a user to add a product variant to their shopping cart using the Vendure commerce package's add-to-cart functionality.

## Capabilities

### Add item with variant ID and quantity

Create an `AddToCartButton` component that:

- Accepts `variantId: string` and `quantity: number` as props
- Uses the package's add-item hook to trigger the mutation when clicked
- Renders a button that calls the add-item function with the provided `variantId` and `quantity`
- Must use the hook from the Vendure commerce package directly (not a generic commerce hook)

[@test](./tests/add-to-cart.test.tsx)

### Quantity validation error

Demonstrate that the add-item mutation throws a structured error (not a plain Error) when given:

- A non-integer quantity (e.g. 1.5)
- A quantity less than 1 (e.g. 0 or -1)

[@test](./tests/quantity-validation.test.ts)

### Default quantity fallback

Show that calling the add-item mutation without an explicit quantity (or with `quantity: undefined`) defaults to adding 1 item.

[@test](./tests/default-quantity.test.ts)

## Implementation

[@generates](./src/add-to-cart.tsx)

## Dependencies { .dependencies }

### @vercel/commerce-vendure 0.0.1 { .dependency }

Vendure provider for the Next.js Commerce framework. Provides a `useAddItem` hook that executes the `addItemToOrder` GraphQL mutation against the Vendure Shop API.

[@satisfied-by](@vercel/commerce-vendure)
