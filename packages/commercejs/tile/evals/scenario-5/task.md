# Fetch Site Information and Categories

Implement a Next.js `getStaticProps` function that retrieves site metadata from the Commerce.js catalog, specifically the list of product categories. The data should be returned as props with a normalized categories array.

## Capabilities

### Server-side site info retrieval

- The function retrieves all product categories from the Commerce.js catalog [@test](./tests/get-categories.test.ts)
- Each returned category has normalized id, name, slug, and path fields [@test](./tests/category-shape.test.ts)
- The brands field in the returned data is an empty array (Commerce.js does not expose a brands API) [@test](./tests/brands-empty.test.ts)
- The function returns both categories and brands as props [@test](./tests/get-site-info-props.test.ts)

## Implementation

[@generates](./src/getSiteInfoStaticProps.ts)

## API

```typescript { #api }
interface Category {
  id: string;
  name: string;
  slug: string;
  path: string;
}
export async function getSiteInfoStaticProps(): Promise<{
  props: {
    categories: Category[];
    brands: never[];
  };
}>;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a server-side operation to fetch site information including normalized product categories and an empty brands array.

[@satisfied-by](@vercel/commerce-commercejs)
