# Search and Filter Products

Build a React component that renders a product search interface. It should use the product search hook from the commerce provider to fetch products filtered by a search query, an optional category, and a sort order. Display the resulting product names in a list.

## Capabilities

### Product search with filtering and sorting

- Passing a search string to the hook returns products matching that string [@test](./tests/search-text.test.tsx)
- Passing a categoryId filters the results to that category [@test](./tests/search-category.test.tsx)
- Passing `"price-asc"` as the sort option returns products sorted by ascending price [@test](./tests/search-sort-price-asc.test.tsx)
- Passing `"price-desc"` as the sort option returns products sorted by descending price [@test](./tests/search-sort-price-desc.test.tsx)
- When the hook returns found: false, the component renders a "No products found" message [@test](./tests/search-not-found.test.tsx)

## Implementation

[@generates](./src/ProductSearch.tsx)

## API

```typescript { #api }
interface ProductSearchProps {
  search?: string;
  categoryId?: string;
  sort?: 'trending-desc' | 'latest-desc' | 'price-asc' | 'price-desc';
}
export function ProductSearch(props: ProductSearchProps): JSX.Element;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a product search hook accepting search, categoryId, brandId, and sort parameters. Returns products array and a found boolean.

[@satisfied-by](@vercel/commerce-commercejs)
