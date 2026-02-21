# Cart Item Quantity Updater

Implement a React component that lets users change the quantity of a line item already in their cart.

## Requirements

1. Use the update-item mutation hook from the Kibo Commerce package to modify a cart line item's quantity.
2. The component receives a `lineItem` prop with the shape `{ id: string, productId: string, variantId: string, quantity: number }`.
3. Render a numeric input initialized to `lineItem.quantity`.
4. When the input value changes, call the update hook's mutation function with the updated quantity and the line item context.
5. The mutation hook should be initialized with the line item passed as the `item` argument (the hook accepts an item context).
6. Setting the quantity to 0 or lower should result in the item being removed from the cart (the package handles this automatically).
7. Export the component as the default export.

## Notes

- The update-item hook accepts an optional `item` context object when initialized.
- The mutation function accepts `{ quantity }` and optionally `{ id, productId, variantId }`.
- The package internally debounces updates to avoid excessive API calls.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useUpdateItem hook for modifying cart line item quantities.

## Test Cases

- [@test](./tests/renders-quantity-input.test.tsx) The component renders an input element initialized to the lineItem's quantity.
- [@test](./tests/calls-update-on-change.test.tsx) Changing the input value calls the update mutation function with the new quantity.
- [@test](./tests/initializes-with-item.test.tsx) The update hook is initialized with the lineItem as the item context argument.
- [@test](./tests/zero-quantity-removes-item.test.tsx) Setting quantity to 0 is passed through to the hook (which delegates removal internally).
