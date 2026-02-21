# Add a Product to the Shopping Cart

Build a React component that renders an "Add to Cart" button. When clicked, it should add a specified product to the shopping cart using the mutation hook provided by the commerce provider. The component should accept a productId and an optional variantId as props.

## Capabilities

### Add product to cart

- Clicking the button calls the add-item mutation with the correct productId [@test](./tests/add-item-call.test.tsx)
- When a variantId is provided as a prop, it is included in the mutation call [@test](./tests/add-item-variant.test.tsx)
- When no quantity is specified, the mutation is called with the default quantity of 1 [@test](./tests/add-item-default-quantity.test.tsx)
- After a successful add, the button becomes enabled again (not in a loading state) [@test](./tests/add-item-loading.test.tsx)

## Implementation

[@generates](./src/AddToCartButton.tsx)

## API

```typescript { #api }
interface AddToCartButtonProps {
  productId: string;
  variantId?: string;
}
export function AddToCartButton(props: AddToCartButtonProps): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides an add-item mutation hook that accepts a productId, optional variantId, and optional quantity (defaults to 1).

[@satisfied-by](@vercel/commerce-commercejs)
