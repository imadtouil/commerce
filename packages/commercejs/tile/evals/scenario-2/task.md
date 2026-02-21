# Update Cart Item Quantity

Build a React component that renders a quantity input for a cart line item. When the quantity input changes, the update-item mutation should be called with the new quantity. The component should support debouncing to avoid excessive API calls during rapid input changes.

## Capabilities

### Update line item quantity with debouncing

- Changing the input value triggers the update-item mutation with the new quantity, the item id, and the productId [@test](./tests/update-item-call.test.tsx)
- The mutation is not called immediately on every keystroke; it is debounced [@test](./tests/update-debounce.test.tsx)
- The hook is initialized with a custom wait time of 750ms passed via the hook's context parameter [@test](./tests/update-wait-time.test.tsx)
- The component renders a numeric input pre-populated with the current item quantity [@test](./tests/update-renders-quantity.test.tsx)

## Implementation

[@generates](./src/QuantityInput.tsx)

## API

```typescript { #api }
interface QuantityInputProps {
  itemId: string;
  productId: string;
  variantId: string;
  currentQuantity: number;
}
export function QuantityInput(props: QuantityInputProps): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides an update-item mutation hook that debounces quantity update calls. Accepts a context object with a wait property to configure debounce delay.

[@satisfied-by](@vercel/commerce-commercejs)
