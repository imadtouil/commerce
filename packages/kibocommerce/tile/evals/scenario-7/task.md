# Customer Wishlist Display

Implement a React component that displays the authenticated customer's wishlist.

## Requirements

1. Use the wishlist query hook from the Kibo Commerce package to fetch the wishlist.
2. Pass `{ includeProducts: true }` to the hook so that product details are included in the response.
3. If the user is not authenticated (no customer data available), render "Please sign in to view your wishlist".
4. Display a loading state while the wishlist is being fetched.
5. If the wishlist is empty (use the `isEmpty` property), render "Your wishlist is empty".
6. If the wishlist has items, render a list of item names.

## Notes

- The wishlist hook automatically fetches the customerId via useCustomer internally.
- If no authenticated customer is found, the hook will not issue a wishlist request.
- The hook returns `{ data, isLoading, error }` where `data` contains the wishlist with `lineItems` and the `isEmpty` property.
- Each line item in the wishlist has a `name` property.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useWishlist hook for fetching the authenticated customer's wishlist.

## Test Cases

- [@test](./tests/shows-sign-in-prompt.test.tsx) When no customer is authenticated, the component renders the sign-in prompt.
- [@test](./tests/shows-loading.test.tsx) While isLoading is true, a loading indicator is shown.
- [@test](./tests/shows-empty-wishlist.test.tsx) When isEmpty is true, the "Your wishlist is empty" message is rendered.
- [@test](./tests/shows-wishlist-items.test.tsx) When wishlist has items, each item's name is rendered in a list.
