# Fetch a Single Product by Slug (Server-Side)

Implement a Next.js `getStaticProps` function that fetches a single product's data by its permalink slug. The operation should retrieve the full product including its variants, normalize the data, and return it as props to the page.

## Capabilities

### Server-side product retrieval by permalink

- Given a product slug, the product is retrieved from the Commerce.js catalog using that slug as a permalink identifier [@test](./tests/get-product-slug.test.ts)
- The returned product data includes normalized variants with their options [@test](./tests/get-product-variants.test.ts)
- If no product is found for the given slug, the function returns `{ notFound: true }` [@test](./tests/get-product-not-found.test.ts)
- The returned product object conforms to the normalized product shape with id, name, description, slug, images, price, variants, and options fields [@test](./tests/get-product-shape.test.ts)

## Implementation

[@generates](./src/getProductStaticProps.ts)

## API

```typescript { #api }
export async function getProductStaticProps(
  context: { params: { slug: string } }
): Promise<{ props: { product: Product } } | { notFound: true }>;
```

## Dependencies { .dependencies }

### @vercel/commerce-commercejs 0.0.1 { .dependency }

Commerce.js provider for Next.js Commerce. Provides a server-side operation to fetch a product by its permalink slug, including all variant data, returning a normalized product object.

[@satisfied-by](@vercel/commerce-commercejs)
