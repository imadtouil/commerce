# Product Search Component

Build a React component that allows users to search the product catalog and displays the results.

## Requirements

1. Use the product search hook from the Kibo Commerce package to query the catalog.
2. Render a text input for the search query, bound to a local state variable.
3. Pass the current search input as the `search` parameter to the search hook.
4. Optionally accept a `categoryId` prop and pass it to the hook when provided.
5. While results are loading, render a "Searching..." message.
6. When results are available, render the count of results found (e.g., "5 products found") followed by a list of product names.
7. If no results are found (`found === 0`), render "No products found".

## Notes

- The search hook is a SWR query hook that accepts `{ search, categoryId, brandId, sort }` as its input.
- It returns `{ products, found, isLoading, error }`.
- Each product in the `products` array has a `name` property.

## Dependencies { .dependencies }

### @vercel/commerce-kibocommerce 0.0.1 { .dependency }

Kibo Commerce provider for Next.js Commerce. Provides the useSearch hook for querying the product catalog.

## Test Cases

- [@test](./tests/uses-search-hook.test.tsx) The component calls the product search hook with the current search input value.
- [@test](./tests/shows-loading.test.tsx) While isLoading is true, "Searching..." is rendered.
- [@test](./tests/shows-products.test.tsx) When products are returned, the count and each product name are rendered.
- [@test](./tests/shows-no-results.test.tsx) When found is 0 and isLoading is false, "No products found" is rendered.
